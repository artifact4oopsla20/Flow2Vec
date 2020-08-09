# Flow2Vec: Value-Flow-Based Precise Code Embedding

This reposory presents the official implementation of Flow2Vec, a new code embedding approach that precisely preserves interprocedural program dependence (a.k.a value-flows), from our paper:

> Yulei Sui, Xiao Cheng, Guanqin Zhang, Haoyu Wang, "Flow2Vec: Value-Flow-Based Precise Code Embedding"

This artifact is packaged as a Docker image, enabling users to reproduce the claims made in the “Evaluation” section of the associated paper
(Section 5) with serveral command lines. It also includes two clients: Code classification (Section 4.4.1) and Code summarization (Section 4.4.2).

## Table of Contents

- [Flow2Vec: Value-Flow-Based Precise Code Embedding](#flow2vec-value-flow-based-precise-code-embedding)
  - [Requirements](#requirements)
  - [Getting Started](#getting-started)
  - [Content](#content)
  - [Step by step](#step-by-step)
    - [1. Statistics (Table 1)](#1-statistics-table-1)
    - [2. Precision, recall and RMSE (Figure 7)](#2-precision-recall-and-rmse-figure-7)
    - [3. Efficiency (Figure 8)](#3-efficiency-figure-8)
    - [4. Code classification and summarization (Table 2)](#4-code-classification-and-summarization-table-2)
    - [5. F1 score under different lengths of code (Figure 9)](#5-f1-score-under-different-lengths-of-code-figure-9)
    - [6. F1 score under different embedding dimensions (Figure 10)](#6-f1-score-under-different-embedding-dimensions-figure-10)
    - [7. Ablation analysis (Figure 11)](#7-ablation-analysis-figure-11)
    - [8. Case study (Figure 12)](#8-case-study-figure-12)
  - [License](#license)
  - [Size of the artifact](#size-of-the-artifact)

## Requirements

The artifact package (around 3GB and 10GB after decompression) has been successfully run on Docker (Unbuntu 18.04 Linux) on a  Intel Xeon Gold 6132 @ 2.60GHz CPUs with 128 GB RAM machine. Note that

(1)  A machine with less memory and CPU may take much longer time (than that in the paper) or may not be scalable to evaluate all the 32 benchmarks. We also provide a small set of datasets with only 8 small benchmarks if you want to do a fast experiment to reproduce their numbers. To run this fast experiment, please try to allocate docker as much RAM memory (>8G memory) as possible on your local machine, otherwise the out-of-memory run will be killed by the OS.
(2) Some of the results are performance data for example (Figure 8) the exact numbers depend on the particular machine. It would be good to observe their trends.
(3) The actual embedding particularly when performing random path sampling which might cause slightly different results depending each run when producing the actual trained model.


## Getting Started

```sh
docker pull flow2vec/artifact:v1.0
docker run -i -t flow2vec/artifact:v1.0 /bin/bash
cd /home
export LLVM_DIR=/home/clang10
```

## Content

The artifact package contains:

- `README.md`: detailed instructions for running and testing the artifact.
- `/home/script`:  scripts to reproduce the experiments
- `/home/clang10`  LLVM pre-binary package
- `SVF/`: source and binary implementation for sparse value flow analysis and graph representations.
- `reconstruction`: implementation of graph reconstruction evaluation.
- `flow2vec`: binary implementation of code embedding.
- `classification`: binary implementation of code classification.
- `summarization`: binary implementation of code summarization.
- `resources/vec/`: value-flow path contexts datasets for code classification.
- `resources/seq/`: value-flow path contexts datasets for code summarization.
- `resources/model/`: well-trained model for case study.
- `resources/data/`: the value flow graphs and callgraphs of  benchmarks.
- `resources/case/`: the value flow graphs and callgraphs of code fragments for case study.
- `resources/open-source/`: benchmarks source.
- `result/`: reproduced results in json format.

## Step by step

### 1. Statistics (Table 1)

To show the statistics of the benchmarks shown in Table 1, run:

```sh
# statistics for source code and IR
bash /home/script/statics.sh
# statistics for nodes and edges on the value-flow graph
bash /home/script/V_E_count.sh
```

The result are also available in `result/statistics.txt ; result/computed_nodes_edges.txt`

### 2. Precision, recall and RMSE (Figure 7)

2.1 Run embedding only on eight small benchmarks, including bc, convert, dc, gzip, echogs, less, mkromfs_0, msg_structure. (running time includes pointer analysis, IVFG generation, context-sensitive reachability, matrix transformation) 
```sh
# Rhis command triggers multiple runs for each benchmark under different embedding dimensions 10, 60, 110, 160 and 210. The estimated run time is around 30 mins
bash ./script/embedding.sh small
# Store the results under folder `/home/result/small` and print the results to terminal.
bash /home/script/evaluation.sh /home/result/small
```

2.2 Run embedding on a particular benchmark (e.g., gzip). The result will be stored in /home/gzip
``` sh
bash /home/script/embedding_single.sh /home/resources/open-source/gzip.orig /home/gzip
# To see the results of a particular benchmark (e.g., gzip).
bash /home/script/evaluation.sh /home/gzip
```


2.3 Run embedding on all benchmarks (it took around 35 hours including IO read/write on a 128RAM machine). The pre-run results on our machine are saved under "/home/result/embed-time" for your reference.
``` sh
# To see the results of the pre-run results directly (stored in `/home/result/evaluation.txt`)
bash /home/script/evaluation.sh /home/result/embed-time
# Re-run all the benchmarks
bash /home/script/embedding.sh all
```


### 3. Efficiency (Figure 8)

Obtain the running time including value-flow construction time and high-order embedding time.

``` sh
bash /home/script/efficiency.sh
cat /home/result/efficiency.txt
```

### 4. Code classification and summarization (Table 2)

Run the following command to obtain the result for code classification and summarization (Table 2):

```sh
cd /home
./classification -op 1 & ./summarization -op 1
```

The result will be stored in `result/f2v-vec.json` and `result/f2v-seq.json` for code classification and summarization respectively.

**(Optional)** To regenerate datasets (composite) for code classification and code summarization, run:

```sh
cd /home
./flow2vec -op 1     
```

### 5. F1 score under different lengths of code (Figure 9)

Run the following command to directly reproduce the result under different lengths of code (Figure 9):

```sh
cd /home
./classification -op 2 & ./summarization -op 2
```

The result will be stored in `result/dimensions-vec.json` and `result/dimensions-seq.json` for code classification and summarization respectively.

**(Optional)** To regenerate datasets under different lengths of code, run:

```sh
cd /home
./flow2vec -op 2    # you can skip this step, we have finished.
```

### 6. F1 score under different embedding dimensions (Figure 10)

To get the result in Figure 10:

```sh
cd /home
./classification -op 3 & ./summarization -op 3
```

The result will be stored in `result/dimensions-vec.json` and `result/dimensions-seq.json` for code classification and summarization respectively.

**(Optional)** To regenerate datasets under different dimensions for code classification and summarization:

```sh
# you can skip this step, we have finished.
cd /home
./flow2vec -op 3
```

### 7. Ablation analysis (Figure 11)

To get the ablation analysis result:

```sh
cd /home
./classification -op 4 & ./summarization -op 4
```

The result will be stored in `result/ablation-vec.json` and `result/ablation-seq.json` for code classification and summarization respectively.

**(Optional)** To regenerate datasets for ablation analysis for code classification and summarization.

```sh
cd /home
# you can skip this step, we have finished.
./flow2vec -op 4
```



### 8. Case study (Figure 12)

The four code fragments (Source code A-D) in Figure 12 are put in `resources/case/case.cpp` and their value-flow graphs are under `resources/case`. We have also provided another example for user to try Flow2Vec under `resources/case-ex/case-ex.cpp`.

To reproduce the result for case study, run:

```sh
cd /home
./summarization -op 5
```

The result will be available in `result/case.json`. Note that the changes made on `example.c` may not produce desired output due to model training using limited samples.

**(Optional)** To regenerate datasets for case study.

```sh
cd /home
# you can skip this step, we have finished.
./flow2vec -op 5
```


## License

The artifact is available under the GNU Lesser General Public License v3.0 or later.

## Size of the artifact

Docker image size is around 8.12 GB
