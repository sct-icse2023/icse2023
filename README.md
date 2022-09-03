# Artifact description
This is the artifact of submission #985 (**Selective Concolic Testing**) in ICSE 2023. 

The artifact is provided as a docker image, which contains the prototype of the proposed method and benchmarks used for evaluation. The aim is to assist the reviewers in reproducing the experimental results in our evaluation.

# Prerequisites
+ Hardware: ~2.50 GHz CPU (all experiments were performed on a server with Intel(R) Xeon(R) Platinum 8269CY CPU @ 2.50GHz), 50+GB free disk space.
+ Unix / Linux OS: We have validated the artifact on Ubuntu system
+ [Docker](https://www.docker.com/pricing/)

# Running the artifact
We assume that the following commands are run in sudo mode. 

Firstly, pull the already prebuilt docker image from [docker hub](https://hub.docker.com/r/sctengine/sct-engine). Please make sure its name is `sctengine/sct-engine`.
```sh
$ docker pull sctengine/sct-engine:v2.01
```

If everything is ok, a `sctengine/sct-engine` image should be found in the images listed by `docker images` command. Then, you can create a container of such image and start a Bash session using the following command. An interactive `bash` shell on the container is also executed at the moment.
```sh
$ docker run -i -t sctengine/sct-engine:v2.01 bash
```

If all goes well, the container should be running successfully. Otherwise, you can seek help from [Docker Doc](https://docs.docker.com/) if needed. 

Now, navigate into the directory containing our experimental environment and list the contents. 
```sh
$ cd /home/sct/exp && ls
core98/  gsl70/  reproduce_core.py  reproduce_gsl.py
```
The necessary items to reproduce our evaluation are listed, including benchmark programs and scripts. The rest of this README assumes you are working in `/home/sct/exp`.

# Reproducing experimental results

There are two benchmarks in our evaluation: GSL (`exp/gsl70`) that contains complex non-linear floating-point conditions, and Coreutils (`exp/core98`) where most programs only contain linear conditions. Hence, we provide two scripts to reproduce experimental results on these two benchmarks, respectively. 

In view of the long running time of our evaluation, the scripts try to perform all experiments in parallel. Specifically, the scripts take a user-defined value *n* and exploit at most *n* cores of working machine at run time. The value should be carefully considered to avoid resource exhaustion. In our evaluation, *n* is 70, since all experiments were performed on a 104-core server with 192GB memory.

**1. Reproduce experimental results on GSL benchmark:**

```bash
$ nohup python3 reproduce_gsl.py 70 & # wait...
```

The analysis will take approximately **175 core-hours**. If you run one CPU core for one hour, that is one core-hour. 

When the analysis is finished, the script will create a new directory `gsl_result` to output the running results, which include a series of *csv* files that correspond to the experimental results of proposed method (`s_res.csv` and `s_covtrend.csv`) and different baselines (the others).

```bash
$ ls ./gsl_result
o_covtrend.txt  pr_covtrend.txt  ps_covtrend.txt  rcn_covtrend.txt  s_covtrend.txt
o_res.csv       pr_res.csv       ps_res.csv       rcn_res.csv       s_res.csv
```

Note that the files with CSV extension give the supported data of **Fig. 4** and **Table 1**, the files with TXT extension give the supported data of **Fig. 6**.

**2. Reproduce experimental results on Coreutils benchmark:**

```bash
$ nohup python3 reproduce_coreutils.py 70 & # wait...
```

The analysis will take approximately **245 core-hours**. When the analysis is finished, you can use KLEE's default statistical tool `klee-stats` to collect the running results. `klee-stats` will collect many useful data directly from the output of KLEE-based tools and represent them in a visually appealing ASCII table.

```bash
$ python3 /home/sct/klee-2.3/build/bin/klee-stats /home/sct/exp/core98_ps/bcfiles
```
Here, we mainly focus on the ICov column, which indicates the coverage of LLVM instructions. These results give the supported data of **Fig. 5**, **Fig. 8** and **Table 2**.

# Analyzing single program
If you are interested in a certain benchmark program, e.g., `gsl_sf_erf_e`, please navigate to the related directory and invoke `run_selective.sh` script as follows.

```bash
# ./run_selective.sh <progDir> <progName> <config> <seconds>
# <config> - smt | jfs | optimize | predict
$ cd /home/sct/exp/gsl70
$ ./run_selective.sh erf gsl_sf_erf_e smt 300
```
Note that the four configurations here correspond to **PS**, **PR**, **S** and **O** in the paper, respectively. The `run_selective.sh` script will invoke our tool configured with pure Z3 backend to analyze program `gsl_sf_erf_e` under `erf` folder for 300 seconds. The analyzing results of this program can be found in the working directory, e.g., `gsl_sf_erf_e_output` and `gsl_sf_erf_e.runlog`.

If you'd like to invoke the vanilla KLEE (**RCN** in the paper), you can invoke `run_klee.sh` script as follows.
```bash
# ./run_klee.sh <progDir> <progName> <seconds>
$ cd /home/sct/exp/gsl70
$ ./run_klee.sh erf gsl_sf_erf_e 300
```

The `run_klee.sh` script will invoke vanilla KLEE to analyze program `gsl_sf_erf_e` under `erf` folder for 300 seconds.

We also provide a `replay.sh` to collect data, i.e., #L-Cov and #F-Cov, of all existing output directories on GSL benchmark. The collected data are redirected to `cov_trend.txt` and `res_all.csv` files.

```bash
$ ./replay.sh
```
