---
layout: default
title: Kùzu 0.0.3 Release
permalink: /blog/kuzu-0.0.3-release.html
parent: Blog
nav_order: 1
---

<p align="center">
  <a href="https://github.com/kuzudb/kuzu"><img src="/img/kuzu-logo.png" width="300"></a>
</p>

<p align="center">
  <a href="https://github.com/kuzudb/kuzu" class="btn fs-5 mb-4 mb-md-0"><i class="fa-brands fa-github"></i></a>
  <a href="https://join.slack.com/t/kuzudb/shared_invite/zt-1qgxnn8ed-9LL7rfKozijOtvw5HyWDlQ" class="btn fs-5 mb-4 mb-md-0"><i class="fa-brands fa-slack"></i></a>
  <a href="https://twitter.com/kuzudb" class="btn fs-5 mb-4 mb-md-0"><i class="fa-brands fa-twitter"></i></a>
</p>

# Kùzu 0.0.3 Release
We are happy to release Kùzu 0.0.3 today. This release comes with the following new main features and improvements:
- [Kùzu as a Pytorch Geometric (PyG) Remote Backend](#kùzu-as-a-pyg-remote-backend): You can now train PyG GNNs and other models directly using graphs (and node features) stored on Kùzu.  See this [Colab notebook](https://colab.research.google.com/drive/12fOSqPm1HQTz_m9caRW7E_92vaeD9xq6)
for a demonstrative example. 
- [Data ingestion from multiple files and numpy files](#data-ingestion-improvements): See below for details
- [Query optimizer improvements](#query-optimizer-improvements): See below for details
- [New buffer manager](#new-buffer-manager): A new state-of-art buffer manager based on [VMCache](https://www.cs.cit.tum.de/fileadmin/w00cfj/dis/_my_direct_uploads/vmcache.pdf).
- [INT32, INT16, FLOAT, and FIXED LIST data types](#new-data-types) (the latter is particularly suitable to store node features in graph ML applications)
- [Query timeout mechanism and interrupting queries from CLI](#other-system-functionalities).

For installing the new version, 
please visit the [download section of our website](https://kuzudb.com/#download) 
and [getting started guide](https://kuzudb.com/docs/getting-started.html) and the full
[release notes are here](https://github.com/kuzudb/kuzu/releases). Please visit
the [Colab Notebooks](https://kuzudb.com/docs/getting-started/colab-notebooks) section of our
documentation website to play with our [Colab notebooks](https://kuzudb.com/docs/getting-started/colab-notebooks).


Enjoy! Please give us a try, [a Github ⭐](https://github.com/kuzudb/kuzu) and your feedback and feature requests! Also follow
us on [Twitter](https://twitter.com/kuzudb)!

## Kùzu as a PyG Remote Backend
Kùzu now implements PyG's Remote Backend interface. So you can directly 
train GNNs using Kùzu as your backend storage. Quoting [PyG documentation's](https://pytorch-geometric.readthedocs.io/en/latest/advanced/remote.html) description
of the Remote Backend feature:

> ...[this feature enables] users to train GNNs on graphs far larger than the size of their
machine’s available memory. It does so by introducing simple, easy-to-use, and extensible abstractions of a `torch_geometric.data.FeatureStore` and a   `torch_geometric.data.GraphStore` that plug directly into existing familiar PyG interfaces.

With our current release, once you store your graph and features in Kùzu,
PyG's samplers work seamlessly using Kùzu's implementation of `FeatureStore` and `GraphStore` interfaces. For example, 
this enables your existing GNN models to work seamlessly by fetching both subgraph samples and node features
from Kùzu instead of PyG's in-memory storage. 
Therefore you can train graphs that do not
fit into your memory since Kùzu, as a DBMS, stores its data on disk. Try this demonstrative [Colab notebook](https://colab.research.google.com/drive/12fOSqPm1HQTz_m9caRW7E_92vaeD9xq6) to 
see an example of how to do this. The current release comes with a limitation that we only truly implement the `FeatureStore` interface.
Inside `GraphStore` we still store the graph topology in memory. 
So in reality only the features are stored and scanned from disk. We plan to address this limitation later on.

Here is also a demonstrative experiment (but certainly not comprehensive study) for the type of training performance 
vs memory usage tradeoff you can expect. 
We trained a simple 3-layers Graph Convolutional Network (GCN) model on [ogbn-papers100M](https://ogb.stanford.edu/docs/nodeprop/#ogbn-papers100M) dataset, which contains about 111 million nodes
with 128 dimensional node features and about 1.6 billion edges. 
Storing the graph topology takes around 48GB[^1] and the features takes 53 GBs. Given our current limitation,
we can reduce 53 GB to something much smaller (we will limit it to as low as 10GB).
We used a machine with one RTX 4090 GPU with 24 GB of memory, two Xeon Platinum 8175M CPUs, and 384 GB RAM, which 
is enough for PyG's in-memory store to store the entire graph and all features in memory.

During training, we use the `NeighborLoader` of PyG with batch size of 48000 and sets the `num_neighbors` to `[30] * 2`, which means at each batch roughly 60 neighbor nodes of 48000 nodes will be sampled from the `GraphStore` and the features of those nodes will be scanned
from Kùzu's storage. We picked this sample size because this gives us a peak GPU memory usage of approximately 22 GB, i.e.,
we can saturate the GPU memory. We used 16 cores[^2] during the sampling process. We run each experiment in a Docker instance
and limit the memory systematically from 110GB, which is enough for PyG to run completely in memory, down to 90, 70, and 60GB.
At each memory level we run the same experiment by using Kùzu as a Remote Backend, where we 
have to use about 48GB to store the topology and give the remaining memory to Kùzu's buffer manager.
For example when the memory is 60GB, we can only give ~10GB to Kùzu.

| Configuration                 | End to End Time (s) | Per Batch Time (s)  | Time Spent on Training (s) | Time Spent on Copying to GPU (s) | Docker Memory | 
|-------------------------------|-----------------|-----------------|------------------------|------------------------------|-------------|
|         PyG In-memory         |      140.17     |      1.4       |          6.62          |             31.25            | 110 GB      |
| Kùzu Remote Backend (bm=60GB) |     392.6     |      3.93      |          6.29          |             34.18            | 110 GB       |          
| Kùzu Remote Backend (bm=40GB) |     589.0     |      5.89      |          6.8          |             32.6            | 90 GB       |          
| Kùzu Remote Backend (bm=20GB) |     1156.1     |      11.5      |          6.0          |             36            | 70 GB       |          
| Kùzu Remote Backend (bm=10GB) |     1121.92     |      11.21      |          6.88          |             35.03            | 60 GB   |         

So, when have enough memory, there is about 2.8x slow down (from 1.4s to 3.93s per batch). This
is the case when Kùzu has enough buffer memory (60GB) to store the 53GB of features but we still incur the cost of 
scanning them through Kùzu's buffer manager. So no or very little disk I/O happens (except the first time
the features are scanned to the buffer manager). Then as we lower the memory, Kùzu can hold only part 
of the of node features in its buffer manager, so
we force Kùzu to do more and more I/O. The per batch time increase to 5.89s at 40GB of buffer manager size, 
then seems to stabilize around 11s (so around 8.2x slowdown). 

The slow down is better if you use smaller batch sizes but for the end to end training time, you
should probably still prefer to use larger batch sizes. This is a place where we would need to
do more research to see how much performance is on the table with further optimizations.

But in summary, if you have 
large datasets that don't fit on your current systems' memories and would like to easily train your PyG models 
off of disk (plus get all the usability features of a GDBMS as you prepare your datasets for training), 
this feature can be very useful for you!

## Data Ingestion Improvements

**Ingest from multiple files**: You can now load data from multiple files of the same type into a node/rel table in two ways:
  - **file list**: `["vPerson0.csv", "vPerson1.csv", "vPerson2.csv"]`
  - **glob pattern matching**: Similar to Linux [Glob](https://man7.org/linux/man-pages/man7/glob.7.html), this will load files that matches the glob pattern.

**Ingest from npy files**: We start exploring how to enable data ingesting in column by column fashion. Consider a `Paper` table defined in the following DDL.
```
CREATE NODE TABLE Paper(id INT64, feat FLOAT[768], year INT64, label DOUBLE, PRIMARY KEY(id));
```
Suppose your raw data is stored in npy formats where each column is represented as a numpy array on disk:
"node_id.npy", "node_feat_f32.npy", "node_year.npy", "node_label.npy".
You can now directly copy from npy files where each file is loaded to a column in `Paper` table as follows:
```
COPY Paper FROM ("node_id.npy", "node_feat_f32.npy", "node_year.npy", "node_label.npy") BY COLUMN;
```

**Reduce memory consumption when ingesting data into node tables:**
This release further optimizes the memory consumption during data ingestion of node tables.
We no longer keep the whole node table in memory before flushing it to disk as a whole. Instead, we process a chunk of a file
and flush its corresponding pages, so incur only the memory cost of ingesting a chunk (or as many chunks as there are threads running).
This greatly reduces memory usage when the node table is very large.

## Query Optimizer Improvements

**Projection push down for sink operator**:
We now push down projections down to the first sink operator 
above the last point in a query plan they are needed.
Consider the following query
```
MATCH (a:person) WHERE a.age > 35 RETURN a.salary AS s ORDER BY s;
```
This query's (simplified) plan is:  `Scan->Filter->OrderBY->ResultCollector`, where both 
`ORDER BY` and the final `ResultCollector` are sink operators. 
`ResultCollector` is where we accumulate the expressions in the `RETURN` clause. 
This is simplified because `ORDER BY` actually consists of several physical operators. 
Both column `age` and `salary` are scanned initially but only `salary` is needed in `ResultCollector`. 
`age`, which is needed by `Filter` is projected out in the `ResultCollector`. We now push the projection of `age`
to `ORDER BY`, so `ORDER BY` does not have to materialize it.

**Other optimizations:** We implemented several other optimizations, such as we reorder the filter expressions so equality conditions
are evaluated first, several improvements to cardinality estimator, and improved sideway information passing for joins. For the latter, 
in our core join operator, which we called  ASP-Joins in our [CIDR paper](https://www.cidrdb.org/cidr2023/papers/p48-jin.pdf), we would blindly
perform sideways information passing (sip) from build to probe (or vice versa; 
see [our paper](https://www.cidrdb.org/cidr2023/papers/p48-jin.pdf) for details). Sometimes if there is no 
filters on the probe and build sides, this is just an overhead as it won't decrease the amount of scans on either side. 
In cases where we think sip won't help reduce scans, we do vanilla Hash Joins now.

## New Buffer Manager

Before this release, we had two internal buffer pools with 2 different frame sizes of 4KB and 256KB,
so operators could only grab buffers of these two sizes. Plus when you loaded your DB and wanted to allocate
say 10GB buffer pool, we manually gave a fixed percentage to 4KB pool and the rest to 256KB pool. 
This didn't give any flexibility for storing large objects and complicated code to manage 
buffers when operators needed them.  Terrible design; 
just don't do this!

We bit the bullet and decided to read the literature and pick a state-of-art buffer manager design that is
also practical. We switched to the mmap-based approach described in VMCache design from [this recent paper](https://www.cs.cit.tum.de/fileadmin/w00cfj/dis/_my_direct_uploads/vmcache.pdf) by Leis et al.. 
This is a very nice design 
and makes it very easy to support multiple buffer sizes very easily and only uses hardware locks (we used 
software locks in our previous buffer manager). It also supports using optimistic reading,
which we verified improves our query performance a lot.

## New Data Types

We now support several additional data types that were missing.

**[FIXED-LIST](https://kuzudb.com/docs/cypher/data-types.html#fixed-list-data-type-tensor-experimental) data type:** This is important if you're doing graph ML and storing node features
in Kùzu. It is the efficient way to store fixed-length vectors. Here's the summary of how
to declare a node or rel property in your schemas to use the fixed-list data type.

| Data Type | Description | DDL definition |
| --- | --- | --- | 
| FIXED-LIST | a list of fixed number of values of the same numerical type | INT64[8] |

When possible use FIXED LIST instead of regular [VAR-LIST](https://kuzudb.com/docs/cypher/data-types.html#var-list-data-type) data type
for cases when you know the size of your lists/vectors. It's much more efficient.

Note that FIXED-LIST is an experimental feature. Currently only bulk loading (e.g. `COPY` statement) and reading is supported.

**INT32, INT16, FLOAT data types:** The release also comes with support for the following data types:

| Data Type | Size | Description |
| --- | --- | --- |
| INT32| 4 bytes | signed four-byte integer |
| INT16| 2 bytes | signed two-byte integer |
| FLOAT | 4 bytes | single precision floating-point number |

For our next release, our focus on data types will be on complex ones, STRUCT and MAP. So stay tuned for those!

## Other System Functionalities

**Query timeout**: We will now automatically stop any query that exceeds a specified timeout value (if one exists). 
The default query timeout value is set to -1, which signifies that the query timeout feature is initially disabled. 
You can activate the query timeout by configuring a positive timeout value through:
  - 1. C++ API: `Connection::setQueryTimeOut(uint64_t timeoutInMS)`
  - 2. CLI: `:timeout [timeoutValue]`

**Interrupt:** You can also interrupt your queries and can stop your long running queries manually. There
are two ways to do this:
  - C++ API: `Connection::interrupt()`: interrupt all running queries within the current connection.
  - CLI: interrupt through `CTRL + C`

Note: The Interruption and Query Timeout features are not applicable to `COPY` commands in this release.

[^1]: Internally, PyG coverts the edge list to CSC format for sampling, which duplicates the graph structures in memory. When you download the graph topology it actually takes about 24GB.
[^2]: We set `num_workers` to 16 when running the PyG in-memory setup. Since Kùzu does not currently work with multiple workers in Python, we limit `num_workers` to 1 when sampling from Kùzu but internally Kùzu scans in parallel with 16 threads.
