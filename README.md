# Flow2Vec: Value-Flow-Based Precise Code Embedding

This package contains the implementation of Flow2Vec, a new code embedding approach that precisely preserves interprocedural program dependence (a.k.a value-flows), from our paper:

> Yulei Sui, Xiao Cheng, Guanqin Zhang, Haoyu Wang, "Flow2Vec: Value-Flow-Based Precise Code Embedding".

This artifact is packaged as a Docker image for users to reproduce the results in the “Evaluation” section (Section 5) of the associated paper.

## Table of Contents

- [Flow2Vec: Value-Flow-Based Precise Code Embedding](#flow2vec-value-flow-based-precise-code-embedding)
  - [Table of Contents](#table-of-contents)
  - [Requirements](#requirements)
  - [Size of the artifact](#size-of-the-artifact)
  - [1. Getting Started](#1-getting-started)
    - [1.1 Get benchmark statistics (Table 1)](#11-get-benchmark-statistics-table-1)
    - [1.2 Precision, recall and RMSE (Figure 7)](#12-precision-recall-and-rmse-figure-7)
    - [1.3 Efficiency (Figure 8)](#13-efficiency-figure-8)
  - [2. Step by step](#2-step-by-step)
    - [2.1 Code classification and summarization (Table 2).](#21-code-classification-and-summarization-table-2)
    - [2.2 F1 score under different lengths of code (Figure 9)](#22-f1-score-under-different-lengths-of-code-figure-9)
    - [2.3 F1 score under different embedding dimensions (Figure 10)](#23-f1-score-under-different-embedding-dimensions-figure-10)
    - [2.4 Ablation analysis (Figure 11)](#24-ablation-analysis-figure-11)
    - [2.5 Case study (Figure 12)](#25-case-study-figure-12)
  - [License](#license)
  - [MD5](#md5)

## Requirements

The experiments were conducted in Unbuntu 18.04 Linux on a Intel Xeon Gold 6132 @ 2.60GHz CPUs with 128 GB RAM machine. Note that
* A machine with less memory and CPU may take much longer time (than that in the paper) or may be unscalable to evaluate all the 32 benchmarks. We have also provided a subset of datasets with only 8 small benchmarks to do a fast experiment to reproduce their  data in the paper (as suggested by the artifact submission website). To run this fast experiment, please try to allocate docker as much RAM memory (>8G memory) as possible on your local machine, otherwise the running might be killed by the underlying OS.
* Some of the results are performance data (e.g., Figure 8) the exact numbers depend on the particular machine. It would be good to observe their trends.
* The actual embedding particularly when performing random path sampling which might cause slightly different results  when producing the trained model (e.g., Figure 7).

## Size of the artifact

Docker image size is around 2.85GB (8.12 GB after decompression)


## 1. Getting Started

```sh
docker pull flow2vec/oopsla
docker run -i -t flow2vec/oopsla /bin/bash
cd /home
export LLVM_DIR=/home/clang10
```

The artifact package contains:

- `README.md`: detailed instructions for running and testing the artifact.
- `/home/script`:  scripts to reproduce the experiments
- `/home/clang10`  LLVM pre-binary package
- `SVF/`: source code for sparse value flow analysis and graph representations.
- `flow2vec`: binary implementation of code embedding.
- `classification` and `summarization`: binary implementations of code classification and summarization.
- `resources/classification/`: value-flow path contexts datasets for code classification.
- `resources/summarization/`: value-flow path contexts datasets for code summarization.
- `resources/model/`: well-trained model for case study.
- `resources/data/`: the value flow graphs and callgraphs of  benchmarks.
- `resources/case/`: the value flow graphs and callgraphs of code fragments for case study.
- `resources/open-source/`: LLVM IR of benchmarks.
- `result/`: reproduced results in json format.


### 1.1 Get benchmark statistics (Table 1)

To show the statistics of the benchmarks shown in Table 1:

```sh
# #LOI, #Method, #Pointers, #Objects, #Call
bash /home/script/statics.sh
# nodes |V| and edges |E| on the value-flow graph
bash /home/script/V_E_count.sh
```
The results are also available in `result/statistics.txt` and `result/computed_nodes_edges.txt`

Note that the statistics are produced by the lastest SVF and recent LLVM/Clang-10.0.0 for later integration/release purposes. The numbers might differ a bit from those in Table 1 (for #Pointers, #Objects, |V| and |E|) since previously we use an early version of SVF and LLVM-9.0 (1st sentence in Section 5.1 in our paper) as the front-end.


### 1.2 Precision, recall and RMSE (Figure 7)

1.2.1 Run embedding only on the eight small benchmarks, including bc, convert, dc, gzip, echogs, less, mkromfs_0, ctypes_test. (running time includes pointer analysis, IVFG generation, context-sensitive reachability, matrix transformation)
```sh
# Rhis command triggers multiple runs for each benchmark under different embedding dimensions 10, 60, 110, 160 and 210. The estimated run time is around 20 mins
bash ./script/embedding.sh small
# Store the results under folder `/home/result/small` and print the results to terminal.
bash /home/script/evaluation.sh /home/result/small
```

1.2.2 Run embedding on a particular benchmark (e.g., gzip). The result will be stored in /home/gzip
``` sh
bash /home/script/embedding_single.sh /home/resources/open-source/gzip.orig /home/gzip
# To see the results of a particular benchmark (e.g., gzip).
bash /home/script/evaluation.sh /home/gzip
```

1.2.3 Run embedding on all benchmarks (it took around 35 hours including IO read/write on a 128RAM machine). The pre-run results on our machine are saved under "/home/result/embed-time" for your reference.
``` sh
# To see the results of the pre-run results directly (stored in `/home/result/evaluation.txt`)
bash /home/script/evaluation.sh /home/result/embed-time
# Re-run all the benchmarks
bash /home/script/embedding.sh all
```


### 1.3 Efficiency (Figure 8)

Obtain the running time including value-flow construction time and high-order embedding time.

``` sh
bash /home/script/efficiency.sh
cat /home/result/efficiency.txt
```

## 2. Step by step

### 2.1 Code classification and summarization (Table 2).

Run the following command to obtain the result for code classification and summarization (Table 2). It may take 30-40 minutes on a local machine with 8GB RAM.


```sh
cd /home
./classification -op 1 & ./summarization -op 1
```

The result will be stored in `result/classification.json` and `result/summarization.json` for code classification and summarization respectively.

**(Optional)** To regenerate datasets (composite) for code classification and code summarization:

```sh
cd /home
./flow2vec -op 1
```

### 2.2 F1 score under different lengths of code (Figure 9)

Run the following command to directly reproduce the result under different lengths of code (Figure 9):

```sh
cd /home
./classification -op 2 & ./summarization -op 2
```

The result will be stored in `result/classification-lengths.json` and `result/summarization-lengths.json` for code classification and summarization respectively.

**(Optional)** To regenerate datasets under different lengths of code, run:

```sh
cd /home
./flow2vec -op 2
```

### 2.3 F1 score under different embedding dimensions (Figure 10)

To get the result in Figure 10:

```sh
cd /home
./classification -op 3 & ./summarization -op 3
```

The result will be stored in `result/classification-dimensions.json` and `result/summarization-dimensions.json` for code classification and summarization respectively.

**(Optional)** To regenerate datasets under different dimensions for code classification and summarization:

```sh
cd /home
./flow2vec -op 3
```

### 2.4 Ablation analysis (Figure 11)

To get the ablation analysis result:

```sh
cd /home
./classification -op 4 & ./summarization -op 4
```

The result will be stored in `result/classification-ablation.json` and `result/summarization-ablation.json` for code classification and summarization respectively.

**(Optional)** To regenerate datasets for ablation analysis for code classification and summarization.

```sh
cd /home
./flow2vec -op 4
```



### 2.5 Case study (Figure 12)

The four code fragments (Source code A-D) in Figure 12 are put in `resources/case/case.cpp` and their value-flow graph is stored in  `resources/case/svfg_final.dot`. We have also provided another example under `resources/case-ex/case-ex.cpp`. (Note that due to the natural of distributed representation of code embedding, the predicted results by a model can be imprecise and different from the ground truth, due to the model trained using limited available training samples.)

To reproduce the result for Figure 12:

```sh
cd /home
./summarization -op 5
```

The result will be stored in `result/case.json`.

**(Optional)** To regenerate datasets for case study.

```sh
cd /home
./flow2vec -op 5
```


## License

The artifact is available under the GNU Lesser General Public License v3.0 or later.

## MD5
```md5
digest: sha256:cfd49586716d688ac19a38c1442ebc53cfbe93138ad35252c6e250a5029faa9f
```
