---
title: "Data Lake Files Format"
date: "2024-08-01"
summary: "learning about data lake files format"
description: "parquet, arrow, etc"
toc: false
readTime: true
---

## Data Lake Files Format

Quick notes about parquet and arrow and other parquet-like file formats. We are interested mostly as  query and discovery data repo, so parquet and arrow is  natural choice.

A little notes on compressed json, as this is the default datalake file format

### Note on Compressing JSON

Json is so de-facto, that when designing a datalake we likely need to think about storing and reading json (csv for structured)

Compressing is always a read write overhead, but transfer, download and storage will benefit.

Gzip is the natural conventional format, and is by default supported almost everywhere.

ZSTD is another format to think about. Generally we expect a real-life use case to have a 10 times compression ratio.

Reading raw json is effectively free, but ZTSD and libdeflate for gzip or cloudflare gzip might have massive read performance increases as well.

For our pattern, most of our read is in parquet, and most of our scrape is network costed, so the compression read/write is probably negligible.

### Arrow vs Parquet

There is a little confusion (mostly on my end) with these two Apache projects, so a quick note on both formats and where they lie:

- Arrow is an IN MEMORY framework and data format.
- In way, because it's focused on in mem, it is probably the de-facto standard for interoperating on columnar data.
- Parquet is largely a file format instead. You are meant to save parquets and read parquets. Parquets is very loved by Hadoop and Spark, and therefore is a de-facto query save format.
- Parquet wins in simplicity and ease of use, adds nice things like compression, schema evolution and nested data
- A standard pattern is save in Parquet, manipulate in Arrow, transfer in Arrow

### How Arrow works

OLAP need speed and volume, especially for large scale data processing and storage (Important for ML workloads). Hadoop opened up the world of big data processing by giving a computing framework to distribute workloads and tools that are built on top of this paradigm, like Hive gives hadoop SQL-like syntax, or HDFS gives hadoop a FS.

Generally for file formats we want to consider row based or columnar, especially for structured data. Columnar given a schema is generally more efficient for retrieval, given proper indexing, that's why Parquet and ORC are columnar, since they aim to support a OLAP workload. Key features of columnar:

- Data adjacency for sequential access (allows for scanning)
- O(1) random access
- SIMD and vectorization friendly
- Zero copy access in shared memory

Memory is cheap, which supports low latency (semi-real time) analytics. So it is more ok now to load sufficient amount of data into memory before analytics workload. But to let us work with different tools and API, you need a in memory intermediate format to allow for interop. So we have to think of a backend data format (ideally columnar) to support cross API, cross language workloads. (This is fairly similar to ONNX and its motivation) (Data transport problems) (removing SERDE cost across tools)

Schema is important to columnar, and honestly Arrow is effectively a data type definition and columnar vectors in a trench coat (the trench coat being table-like containers). It does provide a million read/write methods, and it's the standard backend for pandas. So in general you want to read_table parquet and csv files, then load to a more analytically friendly API to use.  

Metadata (datatypes) leverages Flatbuffers lib for messaging across languages, since it is essentially a fast SERDE lib across languages and formats. **Reference-counted off-heap buffer memory management, facilitating zero-copy memory sharing and adept handling of memory-mapped files.** also means the in memory data can be accessed by multiple threads simultaneously with no copy cost, by off-heap storage and reference counting. Files are also effectively mapped off heap to memory for access. This is a performance gain (less copy and move in memory) and memory save. See Go bytes. Buffer for a better idea of a more basic implementation of reference counted off heap buffer memory.

### How Parquet works

Working from a Hadoop perspective, columnar data is a efficient way to store tabular data, Parquet in particular, is built with nested data and compression in mind. These two focus support the file aspect of parquet, Blocks in HDFS and files make it easy to access per columns. But in order to support better IO and write (since writing is a ROW WISE operation). Parquet therefore takes a hybrid approach using row groups.

Data wise, there are 3 abstractions for the tabular data in parquet:

- _Row group_: A logical horizontal partitioning of the data into rows. There is no physical structure that is guaranteed for a row group. A row group consists of a column chunk for each column in the dataset.
- _Column chunk_: A chunk of the data for a particular column. They live in a particular row group and are guaranteed to be contiguous in the file.
- _Page_: Column chunks are divided up into pages. A page is conceptually an indivisible unit (in terms of compression and encoding). There can be multiple page types which are interleaved in a column chunk. For example, data pages, dictionary pages and index pages.

A file can have one or more row groups, a row group has exactly one column chunk per column,  column chunks can contain one or more pages.

Think of it this way: To parallelize operations: Map-reduce is done on file/row group (Slice of data), IO is parallelize in column chunks (Slice of columns, per row groups), and encoding compression in Pages (Data item level).

Parquet format metadata wise, we have FileMetaData, SchemaElement, ColumnMetaData, PageHeader.

- File Metadata provides file info, such as number of rows, data schema, and row group metadata.
- Row group metadata is not on the official metadata model, but it contains the Column meta data. Effectively it's the collection of the column metadata of the column chunks inside the row group.
- Column meta data has the encoding, compression, size, page ofsets, num of values, min/max of the column chunks. This is the important data to scan to minimize column reads. It can also use to limit row reads by filters and index.
- Page header: it's essentially the encoding values of the granular data. It also handles nested data information (nested
data needs the repetition levels encoding to handle the nested data, since the page header is the way to read and decode data)

For a deepdive to the nested and repeated field storage see [this](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/36632.pdf)
and research google dremel (big query)

Parquet writing:

1. a write request opens up with the actual data, compression scheme, encoding, file scheme and other metadata etc.
2. The Writer collects this info, especially data schema
3. First a magic hash is written to encode the file as parquet
4. Then it gets the number of row groups based on the row groups max size and data size. It then iterates through each row group
5. for each row group, iterate through the columns to write a column chunk for each row group.
6. Chunks will begin by getting the num of rows per page, then the columns stats are calculated and saved
7. column chunks are written page by page (as expected since a page is the smallest unit of data for parquet) the page also writes the page header. If this is a dictionary (nested data), there is also a dictionary page to contain the recursive page headers
8. once a column chunk is complete, the column chunk metadata is written.
9. this is iterated through row groups, then through files
Pretty standard understanding of the iterative writing process, given the file hierachy

Parquet reading

1. A read request needs file lists, filters for row groups, desired columns to read
2. File metadata gets the file schema and row group metadata
3. filters limit row group scans (limited use since there isn't actually a lot of metadata inside row groups) (one exception is indexing via bloom filter, a bloom filter can exist per column, and aggregated to row group level for index checks)
4. Columns lists are specified then this is filtered at the column chunk level
5. This is read iteratively through the hierarchy
6. for a column chunk we can read pages linearly, row count tracks the length of the column chunk, page headers handles compression and encoding.

A few other things on read:

- because of row headers, there is no reason why we cannot read multiple files
- because of row groups, there is inherent parallel readds
- encoding is efficient based on column chunks, since for a column the schema is defined.
- RLE encoding and dictionary encoding helps with data size, beats pure text like json csv
- OLAP loves the idea to skip reads.

Performance notes:

- parquet readers can be used to get zipped bombed. The. Dictionary encoding can be used to reference a value over and over again and that can cause the in memory size to explode.
- Columnar storage and columnar compressions are a plus for parquet performance (or columnar formats in general)
- Columnar partitioning helps a lot in file formats with a lot of columns. In that respect this is very Hive, where the unique values in one or more columns can help with early filtering and read speeeds.
- Row group partitioning is ok, but most tools do not support this as much.
- Defining key partition columns is a feature in most mass parquet writers
  
## An Empirical Evaluation of Columnar Storage Formats

Quick notes on the paper in the title:

- Common columnar storage like parquet and ORC are developed in the early 10s, so now is a good time to benchmark and stress tests these formats, and compare it to current hardwares and practices
- Common good features dictionary encoding by default, decoding speed over compression ratio, block compression is optional, flexible data structures.
- But there are inefficiencies in GPU decoding and ML workloads (load all data, or chunking)
- ORC and parquet are pretty similar in performance
- Dictionary encoding is great for small distinct value columns, common in real world
- Storage is cheaper than compute, so maxing decoding speed is cheaper than compression
- Auxiliary data structures like zone maps bloom filters further help reads. This is only basically supported in parquet/ORC
- ML workloads are high column count workloads. This is not as efficient in this file based storage (one point to make, high column count workloads also makes partitioning difficult.)
- existing columnar formats are probably not great for vector Embedding loading and scanning, there are also floating point compression issues
- Large binaries, do not use in columnar data
- Lessons:
- Dictionary encoding is effective across data. Types, even floating point, this is especially true for real life datasets with low unique value counts. This should be kept for other formats
- Simple encoding schemes equals good decoding performance.
- Query processing bottlenecks are computation. Limit compression and heavy encoding unless necessary
- Metadata layout should be centralized and. Random access support is good for wide data. I/O blocks neeed to think of cloud bandwidth latency
- Storage is cheap, so more indexing and complex data types can help
- Nested data models should align with modern in-memory formats
- ML workflows are wide table and low selectivity
- GPU decoding

Some definite downsides for parquet

- Point lookups are bad because of scanning a page
- Large values make it are to tune row group size
- Large schemas are hard when the table is wide
- Metadata is limited

### Other File Formats - Same Gen as Parquet

#### Avro

Avro is row based storage, that’s native to Kafka.That makes sense, Kafka is storing (sending?) Data on a per event basis, and an event being per row makes tons of sense. It’s self describing (header contains info to read the file) and schema evolution is simple. Data is in binary and metadata is in json

Avro is very Hadoop centric because it runs great in parallel. It’ good for write heavy ops, and schema evolution heavy write ops

#### ORC

ORC is Hive driven columnar storage, so very Hadoop focused again.

With Hive it supports ACID operations. It is again somewhat hybrid, columns are group into a collection of rows, then read iteratively. Type support is good to support HiveQL, concurrency benefits from the stripe based structure (Stripes = Row groups).

Good in Hive, good with reads

### Other File Formats - Parquet Killers

#### Lance

Lance goes further. No more row groups, everything is a data page, column metadata and a footer. No types and no built in encodings, so everything is dependent of extensions

Silently ditching type and schema indicates that LV2 is going to leverage arrow (and arrow only)

No row groups do mean better random access, but non sequential pages mean more reads, which is bad in cloud.

#### Nimble

Nimble is more iterative. It uses flatbuffers to handle wide table (Note this is an Arrow approach as well) encoding support is extensive and extendable

metadata is part of payload encoding instead of a schema in the file

Nimble is treated more as a product than a spec, so instead of extending the format using the core library is encouraged

#### Vortex

Vortex again is very driven by arrow (trend). Noting this here as a space to watch but it hasn’t really been launched yet.

#### Raw Data

links:

```html
https://www.ssp.sh/brain/data-lake-file-formats/
https://medium.com/@diehardankush/comparing-data-storage-parquet-vs-arrow-aa2231e51c8a
https://arrow.apache.org/docs/format/CanonicalExtensions.html
https://ssp.sh/brain/apache-arrow/

https://blog.datasyndrome.com/python-and-parquet-performance-e71da65269ce
https://parquet.apache.org/docs/overview/motivation/
https://vutr.substack.com/p/the-overview-of-parquet-file-format
https://www.ssp.sh/brain/apache-parquet/
https://airbyte.com/blog/data-lake-lakehouse-guide-powered-by-table-formats-delta-lake-iceberg-hudi
https://materializedview.io/p/nimble-and-lance-parquet-killers
https://www.youtube.com/watch?v=bISBNVtXZ6M

https://www.vldb.org/pvldb/vol17/p148-zeng.pdf
https://github.com/XinyuZeng/EvaluationOfColumnarFormats

https://github.com/spiraldb/vortex
```
