# Star Schema Benchmark Results

https://[TO DO].oxla.com/



## Overview

This repository contains collections of performance results on Star Schema Benchmark. Detailed description of this data set can be found in paper published by their author: https://www.cs.umb.edu/~poneil/StarSchemaB.PDF
In this repository we publish a results for dataset generated using a code from this repository: https://github.com/vadimtk/ssb-dbgen
Dataset was generated with scale 1000. At that scale facts table contains about 6 bln rows. Raw CSV data size is ~723GB. Generated data used for this benchmark can be downloaded from those paths:
s3://oxla-public/ssb_s1000/customer.csv.gz
s3://oxla-public/ssb_s1000/lineorder.csv.gz
s3://oxla-public/ssb_s1000/part.csv.gz
s3://oxla-public/ssb_s1000/supplier.csv.gz

## Acknowledgment

This benchmark result comparison framework was prepared using amazing work of Clickhouse team available here: https://github.com/ClickHouse/ClickBench/

## Goals

The main goals of this benchmark are:

### Reproducibility

Published results can be easily reproduced using hardware available on popular cloud or using managed service, if that's how given database is distributed.

### Data warehouse use case

We wanted to gather and present results on a benchmark related to data warehouse use case. The most popular benchmarks related to this use case are:
* TPC-DS
* TCP-H
* Star Schema Benchmark (which is based on TPC-H)
* Clickbench (to some degree)

While the most prominent benchmarks are TPC-DS and TPC-H they are propertiary benchmarks which can't be used without restrictions. Also there are no published results than can be easily browsed to compare popular databases. Clickbench is very popular benchmark. It is using real data with realistic data distribution it. It also covers amazingly well use case of real time OLAP database. Unfortunately it does not cover well wider use case of data warehouses and it is relatively small: it is so small that compressed data is small enough to fit into memory of a laptop which leads to testing in memory performance.

While Star Schema Benchmark (SSB) is not perfect it covers data warehouse use case relatively well. It can also be used to generate relatively large dataset (over 700GB of raw data, over 200GB of compressed data). Unfortunately its data has only uniform distribution which is not normal for real data. Also all of group by operations in SSB result in small number of results (no case with more than 10 000 results).

## Limitations

The following limitations should be acknowledged:

1. The dataset has star schema. It means there are no join between two tables going through intermediate data.

2. The table consist 6bln rows: that's a significant amount but still too small to show how well solutions that are horizontally scalable distribute the work. 

3. No testing of concurrency or mixed workload (writes and reads over the same table).

5. Many setups and systems are different enough to make direct comparison tricky. It is not possible to test the efficiency of storage used for in-memory databases, or the time of data loading for stateless query engines. The goal of the benchmark is to give the numbers for comparison and let you derive the conclusions on your own.

No benchmark result will accurately show you performance of a database in particular use case. Its results might be even misleading. So please use it with caution.

## Rules and Contribution

### How To Add a New Result

To introduce a new system, simply copy-paste one of the directories and edit the files accordingly:
- `benchmark.sh`: this is the main script to run the benchmark on a fresh VM; Ubuntu 22.04 or newer should be used by default, or any other system if specified in the comments. The script may not necessarily run in a fully automated manner - it is recommended always to copy-paste the commands one by one and observe the results. For managed databases, if the setup requires clicking in the UI, write a `README.md` instead.
- `README.md`: contains comments and observations if needed. For managed databases, it can describe the setup procedure to be used instead of a shell script.
- `create.sql`: a CREATE TABLE statement. If it's a NoSQL system, another file like `wtf.json` can be presented.
- `queries.sql`: contains 13 queries to run;
- `run.sh`: a loop for running the queries; every query is run three times; if it's a database with local on-disk storage, the first query should be run after dropping the page cache;
- `results`: put the .json files with the results for every hardware configuration there.

To introduce a new result for an existing system on different hardware configurations, add a new file to `results`.

To introduce a new result for an existing system with a different usage scenario, either copy the whole directory and name it differently (e.g. `timescaledb`, `timescaledb-compression`) or add a new file to the `results` directory.

### Installation And Fine-Tuning

The systems can be installed or used in any reasonable way: from a binary distribution, from a Docker container, from the package manager, or compiled - whatever is more natural and simple or gives better results.

It's better to use the default settings and avoid fine-tuning. Configuration changes can be applied if it is considered strictly necessary and documented.

Fine-tuning and optimization for the benchmark are not recommended but allowed. In this case, add the results on vanilla configuration and fine-tuned configuration separately. What is not allowed is manually pre computing result (e. g. denormalizing data).

### Data Loading

The dataset is available in `CSV` format by the following links:

- https://datasets.clickhouse.com/hits_compatible/hits.csv.gz
- https://datasets.clickhouse.com/hits_compatible/hits.tsv.gz
- https://datasets.clickhouse.com/hits_compatible/hits.json.gz
- https://datasets.clickhouse.com/hits_compatible/hits.parquet

To correctly compare the insertion time, the dataset should be downloaded and decompressed before loading (if it's using external compression; the parquet file includes internal compression and can be loaded as is). The dataset should be loaded as a single file in the most straightforward way. Splitting the dataset for parallel loading is not recommended, as it will make comparisons more difficult. Splitting the dataset is possible if the system cannot eat it as a whole due to its limitations.

You should not wait for cool down after data loading or running OPTIMIZE / VACUUM before the main benchmark queries unless it is strictly required for the system.

The used storage size can be measured without accounting for temporary data if there is temporary data that will be removed in the background. The built-in introspection capabilities can be used to measure the storage size, or it can be measured by checking the used space in the filesystem.

### Indexing

The table can have one index / ordering / partitioning / sharding or clustering.

Manual creation of other indices is not recommended, although if the system creates indexes automatically, it is considered ok.

### Preaggregation

The creation of pre-aggregated tables or indices, projections, or materialized views is not recommended for the purpose of this benchmark. Although you can add fine-tuned setup and results for reference, they will be out of competition.

If a system is of a "multidimensional OLAP" kind, and so is always or implicitly doing aggregations, it can be added for comparison.

### Caching

If the system contains a cache for query results, it should be disabled.

It is okay if the system performs caching for source data (buffer pools and similar). If the cache or buffer pools can be flushed, they should be flushed before the first run of every query.

If the system contains a cache for intermediate data, that cache should be disabled if it is located near the end of the query execution pipeline, thus similar to a query result cache.

### Incomplete Results

Many systems cannot run the full benchmark suite successfully due to OOMs, crashes, or unsupported queries. The partial results should be included nevertheless. Put `null` for the missing numbers.

### If The Results Cannot Be Published

Some vendors don't allow publishing benchmark results due to the infamous [DeWitt Clause](https://cube.dev/blog/dewitt-clause-or-can-you-benchmark-a-database). Most of them still allow the use of the system for benchmarks. In this case, please submit the full information about installation and reproduction, but without the `results` directory. A `.gitignore` file can be added to prevent accidental publishing.

We allow both open-source and proprietary systems in our benchmark, as well as managed services, even if registration, credit card, or salesperson call is required - you still can submit the testing description if you don't violate the TOS.

Please let us know if some results were published by mistake by opening an issue on GitHub.

### If a Mistake Or Misrepresentation Is Found

It is easy to accidentally misrepresent some systems. While acting in good faith, the authors admit their lack of deep knowledge of most systems. Please send a pull request to correct the mistakes.

### Results Usage And Scoreboards

The results can be used for comparison of various systems, but always take them with a grain of salt due to the vast amount of caveats and hidden details. Always reference the original benchmark and this text.

We allow but do not recommend creating scoreboards from this benchmark or saying that one system is better (faster, cheaper, etc.) than another.

There is a web page to navigate across benchmark results and present a summary report. It allows filtering out some systems, setups, or queries. For example, if you found some subset of the 13 queries are irrelevant, you can simply exclude them from the calculation and share the report without these queries.

You can select the summary metric from one of the following: "Cold Run", "Hot Run", "Load Time", and "Data Size". If you select the "Load Time" or "Data Size", the entries will be simply ordered from best to worst, and additionally, the ratio to the best non-zero result will be shown (the number of times one system is worse than the best system in this metric). Load time can be zero for stateless query engines.

If you select "Cold Run" or "Hot Run", the aggregation across the queries is performed in the following way:

1. The first run for every query is selected for Cold Run. For Hot Run, the minimum from 2nd and 3rd run time is selected, if both runs are successful, or null if some were unsuccessful.

By default, the "Hot Run" metric is selected, because it's not always possible to obtain a cold runtime for managed services, while for on-premise a quite slow EBS volume is used by default which makes the comparison slightly less interesting.

2. For every query, find a system that demonstrated the best (fastest) query time and take it as a baseline.

This gives us a point of comparison. Alternatively, we can take a benchmark entry like "ClickHouse on c6a.metal" as a baseline and divide all query times by the baseline time. This would be quite arbitrary and asymmetric. Instead, we take the best result for every query separately.

3. For every query, if the result is present, calculate the ratio to the baseline, but add constant 10ms to the nominator and denominator, so the formula will be: `(10ms + query_time) / (10ms + baseline_query_time)`. This formula gives a value >= 1, which is equal to 1 for the best benchmark entry on this query.

We are interested in relative query run times, not absolute. The benchmark has a broad set of queries, and there can be queries that typically run in 100ms (e.g., for interactive dashboards) and some queries that typically run in a minute (e.g., complex ad-hoc queries). And we want to treat these queries as equally important in the benchmark, that's why we need relative values.

The constant shift is needed to make the formula well-defined when query time approaches zero. For example, some systems can get query results in 0 ms using table metadata lookup, and another in 10 ms by range scan. But this should not be treated as the infinite advantage of one system over the other. With the constant shift, we will treat it as only two times an advantage.

4. For every query, if the result is not present, substitute it with a "penalty" calculated as follows: take the maximum query runtime for this benchmark entry across other queries that have a result, but if it is less than 300 seconds, put it 300 seconds. Then multiply the value by 2. Then calculate the ratio as explained above.

For example, one system crashed while trying to run a query which can highlight the maturity, or lack of maturity, of a system. Or does not run a query due to limitations. If this system shows run times like 1..1000 sec. on other queries, we will substitute 2000 sec. instead of this missing result.

5. Take the geometric mean of the ratios across the queries. It will be the summary rating.

Why geometric mean? The ratios can only be naturally averaged in this way. Imagine there are two queries and two systems. The first system ran the first query in 1s and the second query in 20s. The second system ran the first query in 2s and the second query in 10s. So, the first system is two times faster on the first query and two times slower on the second query and vice-versa. The final score should be identical for these systems.

## Hardware

By default, all tests are run on c6a.4xlarge VM in AWS with 4000 GB gp2.
