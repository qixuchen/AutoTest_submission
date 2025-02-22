# AutoTest: 
This repo contains data, code and full technical report for **AutoTest**. The data and large files (e.g., doduo models) for this project can be found in our [google drive](https://drive.google.com/drive/folders/15pNNrk9IqXOMofR5b2cpftk4A-AfjQiN?usp=sharing).


## Reproduce: jupyter notebook to reproduce main results

We prepared a Jupyter notebook to reproduce the main results in the paper.

In `code/AutoTest_reproduce_main_results.ipynb` we  (1) reproduce our main results in this notebook, including the PR curves of methods compared in the paper (Figure 7 and 8 of the paper), and (2) demonstrate examples errors detected by our proposed SDC on the benchmarks (in output cells).



## Data: train and benchmark datasets

We make our training corpora (`RT-train`, `ST-train`) and the two 1200-column benchmark data  (`RT-bench`, `ST-bench`) available in `data` folder of our [google drive](https://drive.google.com/drive/folders/15pNNrk9IqXOMofR5b2cpftk4A-AfjQiN?usp=sharing) (described in Section 5.1 of the paper). We will release the benchmark data publicly once the double-blind review process is complete.

### Training corpora

Each line in the training corpora `RT-train` and `ST-train` corresponds to a extracted column. Each line has 4 fields (tab-separated) and the meaning of each field from left to right is as follows.


`fname`: the file name from where the column is extracted

`col_header`: header of the column

`dist_val_str` : string of distinct column values

`dist_val_count` : number of distinct column values

Values in `dist_val_str` are concatenated using `___`. For example, a column with `dist_val_str`

    a___b___c___d

corresponds to a column with 4 values a, b, c and d.

### Two 1200-column benchmark data

Each row in `RT-bench` and `ST-bench` corresponds to a table column. 

Each row is described by the following fields:

`header`: header of the column

`ground_truth`: Set of **obvious** errors in this column (if any) 

`ground_truth_debateable`: Set of **contingent** errors in this column (if any)

`dist_val` : a list of distinct column values

`dist_val_count` : number of distinct column values


### Synthetic corpus for SDC selection

The synthetic dataset used for SDC selection is stored in `synthetic.txt`. Each line corresponds to a synthesized column and has 4 fields (tab-separated).

`header`: header of the column

`ground_truth`: the synthesized error.

`dist_val` : a list of distinct column values

`dist_val_count` : number of distinct column values

### Data cleaning benchmarks

The `experiments_using_data_cleaning_benchmarks` folder contains (1) the 9 input benchmark datasets used in our experiment in `data_cleaning_benchmark.txt` (Table 7 of our paper), and (2) the output from running our SDC in `data_clean_sdc.csv`, which shows the new SDC that can be applied on any columns in `data_cleaning_benchmark.txt`.




## Code: main repo

The code for the paper can be found under `code/AutoTest`. Before running the code, please follow the Installation section carefully for setup instructions.

### (1) Offline SDC generation and quality assessment

The code for SDC generation and quality assessment can be found in `STEP1_SDC_generation.ipynb`.

This part takes an unlabeled large corpus as an input and mines SDC in a variety of domains (see the paper for details).
The mined SDC are stored in `AutoTest/output/rule` by default.

The code for this part is expected to be slow on a personal device. 
It took us more than 50 hours to train on a corpus with ~200K columns, on a machine with 2.4GHz 64-core CPU and 512G memory. 

### (2) Offline SDC selection

The code for coarse-grained and fine-grained SDC selection are in `STEP2_SDC_selection.ipynb`.

The input is the set of SDC mined from the previous step. A selected subset of SDC that satisfies the specified constraints is returned.

As a demonstration, you may check `sdc_readables.json` which contains SDC selected by the fine-grained selection process, converted to human-readable format.

### (3) Online inference using SDC

`STEP3_SDC_application.ipynb` contains the code for applying mined rule on test columns to detect possible errors.


## Installation

The project is developed on a Ubuntu system with 2.4GHz 64-core CPU, 512G memory, and a Python version 3.7.16.

After downloading the repo, create a virtual environment `VENV`. Then install all dependencies in the activated `VENV`:

    conda create -n VENV
    conda activate VENV
    cd ./AutoTest
    pip install -r requirements.txt

### Configuration setup

Before running the code, you need to specify the directories where you want store the results, datasets and SDCs in `config.py`. 
<!-- See the comments in `config.py` for the meaning of each directory.  -->

In general, there are two major directories that needs to be correctly set. The first one is the path specified by `config.dir.project_base_dir` which is the directory of the code base (i.e., where `AutoTest` is located).

The second one is the path specified by `config.dir.storage_root_dir` which is the directory where the corpora, benchmarks and intermediate results are stored.
Note that you need to reserve sufficiently large storage space for this directory. We recommend to reserve at least 200 - 300 GB, but more may be required if you want to try the code on larger training corpora. 

You may run `AutoTest_path_setup.py` and follow the instructions to check if everything is set up correctly.

Remember to put (1) the training corpura, (2) the test benchmarks and (3) the synthetic dataset (for SDC selection) to the location as specified in `config.py`. Specifically,

1, Put RT_Train and ST_Train under `{config.dir.storage_root_dir}/{config.dir.storage_root.train_corpora}`.

2, Put RT_bench and ST_bench under `{config.dir.storage_root_dir}/{config.dir.storage_root.benchmark}`.

3, Put the synthetic dataset (for SDC selection) under `{config.dir.storage_root_dir}/{config.dir.storage_root.synth_dataset}`.

### Download GLoVe pretrained vectors

The GLoVe-related SDC requires GLoVe pretrained vectors. You may download it under under its [official website](https://nlp.stanford.edu/projects/glove/). The [6B version](https://nlp.stanford.edu/data/glove.6B.zip) is used in this project.

After downloading the zip file, unzip it and put the content under `{config.dir.storage_root_dir}/{config.dir.storage_root.glove}`.


### Load doduo models


The Doduo-related SDC requires [doduo](https://github.com/megagonlabs/doduo). We have separated the file for Doduo's large model and data files in a separate place. You may find them in `model` and `data` under `code/doduo-materials` in the [google drive](https://drive.google.com/drive/folders/15pNNrk9IqXOMofR5b2cpftk4A-AfjQiN?usp=sharing).

After downloading them, you need to put them into the corresponding location under `AutoTest/doduo-project` as follows.

```console
$ tree doduo-project
doduo-project
├── data
├── model
...
```



### Minor note
It is noticed that the version of `multiprocessing` module used by this project may report the following bug when the training corpus is too large (~200K columns as what we used in the paper). 
You may see the following error message:

    struct.error: 'i' format requires -2147483648 <= number <= 2147483647

You can fix this by manually change several lines of code in the `multiprocessing` module, following [this post](https://github.com/open-mmlab/mmdetection/issues/2044). 

More specifically, the change is [this](https://github.com/python/cpython/commit/bccacd19fa7b56dcf2fbfab15992b6b94ab6666b).
