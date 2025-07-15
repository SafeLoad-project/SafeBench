<p align="center">
    <h3 align="center">SafeBench</h3>
    <p align="center">An industrial-grade benchmark of identifying memory-overloading queries.</p>
    <p align="center">
        <a href="https://safeload-project.github.io/Homepage/">🏠 Homepage</a> •
        <a href="https://www.kaggle.com/datasets/onefanwu/safebench">📊 Datasets</a> •
        <a href="https://safeload-project.github.io/Homepage/#/query-features">🧩 Query Features</a> •
        <a href="https://safeload-project.github.io/Homepage/#/rule-library">📚 Rule Library</a>
    </p>
</p>


## 🚀 Introduction
**SafeBench** is an industrial-grade benchmark open-sourced to advance academic research on preemptively identifying memory-overloading (MO) queries. SafeBench was curated by the Alibaba Cloud [AnalyticDB](https://www.alibabacloud.com/en/product/analyticdb-for-mysql) team following rigorous data quality assessment and thorough removal of anomalous data. The [homepage](https://safeload-project.github.io/Homepage/) of SafeBench is now live. **The datasets can be downloaded [here](https://www.kaggle.com/datasets/onefanwu/safebench).**

| Subset | \#(Queries) | \#(Clusters) | \#(Pos.) |    G1    |    G2    |
|:------:|:-----------:|:------------:|:--------:|:---------:|:--------:|
|   A1   |  52,080,130 |      854     |   4,014  | training |          |
|   A2   |  50,856,068 |      862     |   2,508  |  testing | training |
|   A3   |  48,821,677 |      856     |   2,331  |          |  testing |

This table outlines the specifications of SafeBench, which is constructed from real-world production data collected from Alibaba Cloud's data warehouse, AnalyticDB. The dataset spans three continuous days, forming three distinct subsets: SafeBench A1 (Day 1), A2 (Day 2), and A3 (Day 3). These subsets include over 150 million analytical queries executed across more than 800 production database clusters. The average CPU time per query is 7.9 seconds (A1), 8.6 seconds (A2), and 8.6 seconds (A3), respectively. 

To assess the effectiveness of MO query detection methods, we divide the datasets into two evaluation groups.
Group G1 uses A1 for training and A2 for testing, while Group G2 uses A2 for training and A3 for testing. 
By default, SafeBench uses the data from the previous day for training and the data from the current day for testing, reflecting the practical deployment setting in AnalyticDB where both models and heuristic rules are updated on a daily basis. Of course, researchers are free to combine data from A1, A2, and A3 as training or testing sets according to their own experimental needs.

## 🧬 Feature Group
SafeBench provides detailed profiling for each query from both the query-level and the cluster-level perspectives. Each query is uniquely identified by a query ID with a submission timestamp and is annotated with metadata such as cluster name, total CPU time, and a binary label indicating whether memory overload occurred. Each query is further represented by a 163-dimensional feature vector, comprising 147 query-level features and 16 cluster-level features. 

The query-level features characterize structural and statistical aspects of query plans, including operator types, cardinalities, memory-intensive operators, and user-specified execution modes. The cluster-level features describe system-level context, such as resource utilization metrics, static hardware configurations, and historical memory overload signals. Notably, all eight cluster resource metrics are collected at a one-minute granularity, a design choice intended to minimize the monitoring overhead on the data warehouse system. For example, if a query is issued at 08:03:30, its corresponding cluster resource metrics are taken from the snapshot recorded at 08:03:00. These features offer a holistic view of query execution behaviors and system conditions, enabling robust detection and analysis of MO patterns. 

A categorized summary of the features is provided in the following table.

| <b/>Category</b>      | <b/>Feature Group</b>                    | <b/>Description</b>                                                                                                                                                                                                                                                                                     | <b/>Count</b> |
|---------------|----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-----:|
| 🟦 Query-level   | Operator Count                   | Number of distinct operators (23 types in total), such as the count of join nodes in the query.                                                                                                                                                                                        |   23  |
| 🟦 Query-level   | Operator  Cardinality    | Cardinality statistics for 13 operators, covering 8 metrics per operator, including total and maximum values of the output size and row count. These operators are selected based on error reports from MO queries.                                                                             |  104  |
| 🟦 Query-level   | Memory-intensive Operators       | Fine-grained features of operators typically associated with high memory usage (i.e., join, aggregation, window, sort); includes child node cardinality for join and total number of varchar-typed grouping keys in aggregation. |   19  |
| 🟦 Query-level   | Execution  Configuration | Configuration information during query execution scheduling, including the total degree of query parallelism and an indicator of whether the SQL query explicitly specifies an execution mode (e.g., batch mode), which may impact memory behavior.                                    |   2   |
| 🟩 Cluster-level | Resource Metrics                 | Cluster resource usage metrics collected at 1-minute granularity, including QPS, average memory pool utilization, and CPU utilization.                                                                                                                                                          |   8   |
| 🟩 Cluster-level | Cluster Configuration            | Static configuration of the provisioned cluster, including the number of CPU cores and the resource group ID.                                                                                                                                                                                   |   6   |
| 🟩 Cluster-level | OOM Indicator                    | Number of OOM events observed in the corresponding cluster on the previous day.                                                                                                                                                                                                                 |   1   |

## 📁 File Description
The SafeBench datasets can be found [here](https://www.kaggle.com/datasets/onefanwu/safebench) at Kaggle. We take the three files from the A1 dataset as examples. The files **A1\_positive\_samples.parquet** and **A1\_negative\_samples.parquet** store the positive and negative samples, respectively, for the A1 dataset. Both Parquet files share an identical schema, comprising three columns: `processid_createtime`, `instance_name`, and `feat_vec`.

The first two columns, `processid_createtime` and `instance_name`, are of **string type**. `processid_createtime` serves as a query identifier, while `instance_name` denotes the database cluster name. The third column, `feat_vec`, is a **float array** that holds a 163-dimensional feature vector. It is noteworthy that the latter part of the `processid_createtime` string, following the hyphen (`-`), represents the timestamp when the query occurred.

The file **A1\_cputime.parquet** contains the actual total CPU time consumed by each query. This file has two columns: `processid_createtime` and `wall_time`. The `processid_createtime` column serves as the query identifier, while `wall_time` represents the CPU time consumed by the respective query.


## 🏷️ Feature Identifiers and Description
The following table lists the features associated with each query in SafeBench, along with their identifiers and descriptions.

| <b>Feature Identifier</b> | <b>Description</b>                                               |
|--------------------|-----------------------------------------------------------|
| feature_0          | The number of aggregation nodes.                                     |
| feature_1          | The total estimated number of input rows across all aggregation nodes.                      |
| feature_2          | The maximum estimated number of input rows across all aggregation nodes.                    |
| feature_3 | The total estimated number of output rows across all aggregation nodes. |
| feature_4 | The maximum estimated number of output rows across all aggregation nodes. |
| feature_5 | The total estimated input size across all aggregation nodes. |
| feature_6 | The maximum estimated input size across all aggregation nodes. |
| feature_7 | The total estimated output size across all aggregation nodes. |
| feature_8 | The maximum estimated output size across all aggregation nodes. |
| feature_9          | The number of exchange nodes.                                         |
| feature_10         | The total estimated number of input rows across all exchange nodes.                          |
| feature_11 | The maximum estimated number of input rows across all exchange nodes. |
| feature_12 | The total estimated number of output rows across all exchange nodes. |
| feature_13 | The maximum estimated number of output rows across all exchange nodes. |
| feature_14 | The total estimated input size across all exchange nodes. |
| feature_15 | The maximum estimated input size across all exchange nodes. |
| feature_16 | The total estimated output size across all exchange nodes. |
| feature_17 | The maximum estimated output size across all exchange nodes. |
| feature_18         | The number of join nodes.                                             |
| feature_19         | The total estimated number of input rows across all join nodes.                             |
| feature_20 | The maximum estimated number of input rows across all join nodes. |
| feature_21 | The total estimated number of output rows across all join nodes. |
| feature_22 | The maximum estimated number of output rows across all join nodes. |
| feature_23 | The total estimated input size across all join nodes. |
| feature_24 | The maximum estimated input size across all join nodes. |
| feature_25 | The total estimated output size across all join nodes. |
| feature_26 | The maximum estimated output size across all join nodes. |
| feature_27         | The number of remoteSource nodes.                                     |
| feature_28         | The total estimated number of input rows across all remoteSource nodes.                      |
| feature_29 | The maximum estimated number of input rows across all remoteSource nodes. |
| feature_30 | The total estimated number of output rows across all remoteSource nodes. |
| feature_31 | The maximum estimated number of output rows across all remoteSource nodes. |
| feature_32 | The total estimated input size across all remoteSource nodes. |
| feature_33 | The maximum estimated input size across all remoteSource nodes. |
| feature_34 | The total estimated output size across all remoteSource nodes. |
| feature_35 | The maximum estimated output size across all remoteSource nodes. |
| feature_36         | The number of tablecommit nodes.                                      |
| feature_37         | The total estimated number of input rows across all  tablecommit nodes.                     |
| feature_38 | The maximum estimated number of input rows across all tablecommit nodes. |
| feature_39 | The total estimated number of output rows across all tablecommit nodes. |
| feature_40 | The maximum estimated number of output rows across all tablecommit nodes. |
| feature_41 | The total estimated input size across all tablecommit nodes. |
| feature_42 | The maximum estimated input size across all tablecommit nodes. |
| feature_43 | The total estimated output size across all tablecommit nodes. |
| feature_44 | The maximum estimated output size across all tablecommit nodes. |
| feature_45         | The number of tablescan nodes.                                        |
| feature_46         | The total estimated number of input rows across all tablescan nodes.                        |
| feature_47 | The maximum estimated number of input rows across all tablescan nodes. |
| feature_48 | The total estimated number of output rows across all tablescan nodes. |
| feature_49 | The maximum estimated number of output rows across all tablescan nodes. |
| feature_50 | The total estimated input size across all tablescan nodes. |
| feature_51 | The maximum estimated input size across all tablescan nodes. |
| feature_52 | The total estimated output size across all tablescan nodes. |
| feature_53 | The maximum estimated output size across all tablescan nodes. |
| feature_54 | The number of tablewriter nodes. |
| feature_55 | The total estimated number of input rows across all tablewriter nodes. |
| feature_56 | The maximum estimated number of input rows across all tablewriter nodes. |
| feature_57 | The total estimated number of output rows across all tablewriter nodes. |
| feature_58 | The maximum estimated number of output rows across all tablewriter nodes. |
| feature_59 | The total estimated input size across all tablewriter nodes. |
| feature_60 | The maximum estimated input size across all tablewriter nodes. |
| feature_61 | The total estimated output size across all tablewriter nodes. |
| feature_62 | The maximum estimated output size across all tablewriter nodes. |
| feature_63 | The number of window nodes. |
| feature_64 | The total estimated number of input rows across all window nodes. |
| feature_65 | The maximum estimated number of input rows across all window nodes. |
| feature_66 | The total estimated number of output rows across all window nodes. |
| feature_67 | The maximum estimated number of output rows across all window nodes. |
| feature_68 | The total estimated input size across all window nodes. |
| feature_69 | The maximum estimated input size across all window nodes. |
| feature_70 | The total estimated output size across all window nodes. |
| feature_71 | The maximum estimated output size across all window nodes. |
| feature_72 | The number of markDistinct nodes. |
| feature_73 | The total estimated number of input rows across all markDistinct nodes. |
| feature_74 | The maximum estimated number of input rows across all markDistinct nodes. |
| feature_75 | The total estimated number of output rows across all markDistinct nodes. |
| feature_76 | The maximum estimated number of output rows across all markDistinct nodes. |
| feature_77 | The total estimated input size across all markDistinct nodes. |
| feature_78 | The maximum estimated input size across all markDistinct nodes. |
| feature_79 | The total estimated output size across all markDistinct nodes. |
| feature_80 | The maximum estimated output size across all markDistinct nodes. |
| feature_81 | The number of topn nodes. |
| feature_82 | The total estimated number of input rows across all topn nodes. |
| feature_83 | The maximum estimated number of input rows across all topn nodes. |
| feature_84 | The total estimated number of output rows across all topn nodes. |
| feature_85 | The maximum estimated number of output rows across all topn nodes. |
| feature_86 | The total estimated input size across all topn nodes. |
| feature_87 | The maximum estimated input size across all topn nodes. |
| feature_88 | The total estimated output size across all topn nodes. |
| feature_89 | The maximum estimated output size across all topn nodes. |
| feature_90 | The number of limit nodes. |
| feature_91 | The total estimated number of input rows across all limit nodes. |
| feature_92 | The maximum estimated number of input rows across all limit nodes. |
| feature_93 | The total estimated number of output rows across all limit nodes. |
| feature_94 | The maximum estimated number of output rows across all limit nodes. |
| feature_95 | The total estimated input size across all limit nodes. |
| feature_96 | The maximum estimated input size across all limit nodes. |
| feature_97 | The total estimated output size across all limit nodes. |
| feature_98 | The maximum estimated output size across all limit nodes. |
| feature_99 | The number of topnRowNumber nodes. |
| feature_100 | The total estimated number of input rows across all topnRowNumber nodes. |
| feature_101 | The maximum estimated number of input rows across all topnRowNumber nodes. |
| feature_102 | The total estimated number of output rows across all topnRowNumber nodes. |
| feature_103 | The maximum estimated number of output rows across all topnRowNumber nodes. |
| feature_104 | The total estimated input size across all topnRowNumber nodes. |
| feature_105 | The maximum estimated input size across all topnRowNumber nodes. |
| feature_106 | The total estimated output size across all topnRowNumber nodes. |
| feature_107 | The maximum estimated output size across all topnRowNumber nodes. |
| feature_108 | The number of sort nodes. |
| feature_109 | The total estimated number of input rows across all sort nodes. |
| feature_110 | The maximum estimated number of input rows across all sort nodes. |
| feature_111 | The total estimated number of output rows across all sort nodes. |
| feature_112 | The maximum estimated number of output rows across all sort nodes. |
| feature_113 | The total estimated input size across all sort nodes. |
| feature_114 | The maximum estimated input size across all sort nodes. |
| feature_115 | The total estimated output size across all sort nodes. |
| feature_116 | The maximum estimated output size across all sort nodes. |
| feature_117        | The number of filter nodes.                                           |
| feature_118        | The number of output nodes.                                           |
| feature_119        | The number of project nodes.                                          |
| feature_120        | The number of sipConsumer nodes.                                      |
| feature_121        | The number of sipProducer nodes.                                      |
| feature_122        | The number of scalar nodes.                                           |
| feature_123        | The number of runtimeCollect nodes.                                   |
| feature_124        | The number of runtimeFilter nodes.                                    |
| feature_125        | The number of values nodes.                                           |
| feature_126        | The number of runtimeScan nodes.                                      |
| feature_127        | The total number of INNER-type joins among all join nodes.                      |
| feature_128        | The total number of FULL-type joins among all join nodes.                       |
| feature_129        | The total number of LEFT-type joins among all join nodes.                       |
| feature_130        | The total number of RIGHT-type joins among all join nodes.                      |
| feature_131        | The total number of join nodes with a REPLICATED distribution type.                 |
| feature_132        | The total number of join criteria across all join nodes.                |
| feature_133        | The total number of output symbols of type VARCHAR across all join nodes. |
| feature_134        | The total number of join keys with data type VARCHAR in the join conditions across all join nodes.      |
| feature_135        | The total number of output symbols across all join nodes.                      |
| feature_136        | The total estimated output size of the left child nodes across all join nodes.                    |
| feature_137        | The total estimated output size of the right child nodes across all join nodes.                    |
| feature_138        | The total number of grouping keys across all aggregation nodes.                |
| feature_139        | The total sum of grouping-set count across all aggregation nodes.                |
| feature_140        | The total number of grouping keys of type VARCHAR across all aggregation nodes. |
| feature_141        | The total number of aggregation nodes with step type PARTIAL.             |
| feature_142        | The total number of partitionBy keys across all window nodes.                    |
| feature_143        | The total number of orderBy keys across all window nodes.                        |
| feature_144        | The total number of ordering directions across all window nodes.                 |
| feature_145        | The total number of orderBy keys across all sort nodes.                         |
| feature_146        | The number of queries processed per second by the DB cluster.                                              |
| feature_147        | The average utilization ratio of the general resource pool in the DB cluster.                            |
| feature_148        | The average utilization ratio of the system resource pool in the DB cluster.                              |
| feature_149        | The maximum observed utilization ratio of the general resource pool in the DB cluster.                            |
| feature_150        | The maximum observed utilization ratio of the system resource pool in the DB cluster.                             |
| feature_151        | The percentage of CPU resources currently utilized by the DB cluster.                                 |
| feature_152        | The percentage of memory resources currently utilized by the DB cluster.                                  |
| feature_153        | The average query response time, measured in milliseconds, for queries executed on the DB cluster.                                  |
| feature_154        | The total number of CPU cores allocated to the DB cluster.                                |
| feature_155        | The number of CPU cores dedicated to query execution within the DB cluster.                             |
| feature_156        | The number of CPU cores allocated to worker threads.                               |
| feature_157        | The number of front-end nodes in the DB cluster.                             |
| feature_158        | The user-configured garbage collection level in the DB cluster.                         |
| feature_159        | The degree of query parallelism.                                               |
| feature_160        | Resource group ID of the DB cluster.                                         |
| feature_161        | The number of Out-of-Memory (OOM) events that occurred in the database cluster associated with the query on the previous day.                                 |
| feature_162        | Whether the query specifies batch execution mode.                             |

## 📚 Rule Library
Please refer to [rule library](https://github.com/SafeLoad-project/SafeBench/blob/main/rule_library.txt) for more details.

## 📝 License
SafeBench is released under the Creative Commons Attribution-NonCommercial 4.0 International License (CC BY-NC 4.0). This license permits non-commercial use, distribution, and reproduction in any medium, provided the original author(s) and source are credited. For detailed terms, please refer to the [CC BY-NC 4.0 license](https://creativecommons.org/licenses/by-nc/4.0/) deed.
