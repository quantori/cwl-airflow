[![Build Status](https://travis-ci.org/Barski-lab/cwl-airflow.svg?branch=master)](https://travis-ci.org/Barski-lab/cwl-airflow) -  **Travis CI**  
[![Build Status](https://ci.commonwl.org/buildStatus/icon?job=airflow-conformance)](https://ci.commonwl.org/job/airflow-conformance) - **CWL conformance tests**  

# cwl-airflow

Python package to extend **[Apache-Airflow 1.9.0](https://github.com/apache/incubator-airflow)**
functionality with **[CWL v1.0](http://www.commonwl.org/v1.0/)** support.
---
## Demo mode
*(assuming that you already have installed and properly configured python, pip, setuptools and docker)*
1. Install *cwl-airflow*
    ```sh
    $ pip install cwl-airflow --user --find-links https://michael-kotliar.github.io/cwl-airflow-wheels/
    ```
2. Init configuration
    ```sh
    $ cwl-airflow init
    ```
3. Run *demo*
    ```sh
    $ cwl-airflow demo --auto
    ```
4. When all demo wokrflows are submitted program will provide you with the link for Airflow Webserver.
It may take some time (usually less then half a minute) for Airflow Webserver to load and display all the data

---


## Table of Contents

* **[How It Works](#How-It-Works)**
  * [Key concepts](#Key-concepts)
* **[Installation](#Installation)**
  * [Requirements](#Requirements)
    * [Ubuntu 16.04.4 (Xenial Xerus)](#Ubuntu-16.04.4-(Xenial-Xerus))
    * [macOS 10.13.5 (High Sierra)](#macOS-10.13.5-(High-Sierra))
    * [Both Ubuntu and macOS](#Both-Ubuntu-and-macOS)
  * [Install cwl-airflow](#Install-cwl-airflow)
* **[Using cwl-airflow](#Using-cwl-airflow)**
    * [Configuration](#Configuration)
    * [Submitting new job](#Submitting-new-job)
    * [Demo mode](#Demo-mode)
---

## How It Works
### Key concepts

1. **CWL descriptor file** - *YAML* or *JSON* file to describe the workflow inputs, outputs and steps.
   File should be compatible with CWL v1.0 specification
2. **Job file** - *YAML* or *JSON* file to set the values for the wokrflow inputs.
   For *cwl-airflow* to function properly the Job file should include 3 mandatory and 
   one optional fields:
   - *workflow* - mandatory field to specify the absolute path to the CWL descriptor file
   - *output_folder* - mandatory field to specify the absolute path to the folder
   where all the output files should be moved after successful workflow execution
   - *tmp_folder* - optional field to specify the absolute path to the folder
   for storing intermediate results. After workflow execution this folder will be deleted.
   - *uid* - mandatory field that is used for generating DAG's unique identifier. 
3. **DAG** - directed acyclic graph that describes the workflow structure.
4. **Jobs folder** - folder to keep all Job files scheduled for execution or the ones that
have already been processed. The folder's location is set as *jobs* parameter of *cwl* section
in Airflow configuration file.  

*cwl-airflow* package provides the original Apache-Airflow with the script that when placed into the DAGs
folder will parse CWL descriptor and Job files and create the new DAG for each of the Job file.
Such a DAG will be executed when Airflow Scheduler is running.   


## Installation
### Requirements 
#### Ubuntu 16.04.4 (Xenial Xerus)
- python 2.7 or 3.5 (tested on the default Python 2.7.12 and 3.5.2)
- docker
  ```
  sudo apt-get update
  sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  sudo apt-get update
  sudo apt-get install docker-ce
  sudo groupadd docker
  sudo usermod -aG docker $USER
  ```
  Log out and log back in so that your group membership is re-evaluated.
- python-dev (or python3-dev if using Python 3.5)
  ```bash
  sudo apt-get install python-dev # python3-dev
  ``` 
  *python-dev* is required in case your system needs to compile some python
  packages during the installation. We have built python *wheels* for most of such packages
  and provided them through *--find-links* argument while installing *cwl-airflow*.
  Nevertheless in case of installation problems you might still be required to install
  this dependency.
  
  
#### macOS 10.13.5 (High Sierra)
- python 2.7 or 3.6 (tested on the default Python 2.7.10 and the latest Python 3.6.5 availble from Homebrew)
- docker (follow the [link](https://docs.docker.com/docker-for-mac/install/)
  to install Docker on Mac)
- Apple Command Line Tools
  ```bash
  xcode-select --install
  ```
  Click *Install* on the pop up when it appears, follow the instructions
  
#### Both Ubuntu and macOS
- pip
  ```bash
  wget https://bootstrap.pypa.io/get-pip.py
  python get-pip.py --user
  ```
  When using the system Python on MacOS, you might need to update your *PATH* variable following
  the instruction printed on console
- setuptools (should be updated to the latest)
  ```
  pip install -U setuptools --user
  ```
### Install cwl-airflow
```sh
$ pip install cwl-airflow --user --find-links https://michael-kotliar.github.io/cwl-airflow-wheels/
```
To avoid installing *Xcode* for macOS users and *python-dev* for Ubuntu users, some
of the Python packages have been already compiled and put into the separate
[repository](https://michael-kotliar.github.io/cwl-airflow-wheels/)
that is set with *--find-links* argument.



## Using cwl-airflow

### Configuration

Before using *cwl-airflow* it should be initialized with the default configuration by running the command
```sh
$ cwl-airflow init
```
Additionally you can specify the following optional parameters:

- `-l LIMIT`, `--limit LIMIT` sets the number of processed jobs kept in history.
 Default 10 for each of the category: *Running*, *Success*, *Failed* 
- `-j JOBS`, `--jobs JOBS` sets the path to the folder where all the new jobs will be added.
Default *~/airflow/jobs*
- `-t DAG_TIMEOUT`, `--timeout DAG_TIMEOUT` sets timeout (in seconds) for importing all the DAGs
from the DAG folder. Default 30 seconds 
- `-r WEB_INTERVAL`, `--refresh WEB_INTERVAL` sets the webserver workers refresh interval (in seconds). Default 30 seconds
- `-w WEB_WORKERS`, `--workers WEB_WORKERS` sets the number of webserver workers to be refreshed at the same time. Default 1
- `-p THREADS`, `--threads THREADS` sets the number of threads for Airflow Scheduler. Default 2

If you update Airflow configuration file manually (default location is *~/airflow/airflow.cfg*),
make sure to run *cwl-airflow init* command to apply all the changes,
especially if *core/dags_folder* or *cwl/jobs* parameters from the configuration file are changed.

    
### Submitting new job

To submit new CWL descriptor and Job files to *cwl-airflow* run the following command
```bash
cwl-airflow submit WORKFLOW JOB
```
Additionally you can specify the following optional parameters
- `-o OUTPUT_FOLDER`, `--outdir OUTPUT_FOLDER` sets the path to the folder
   where all the output files should be moved after successful workflow execution.
   Default: current directory
- `-t TMP_FOLDER`, `--tmp TMP_FOLDER` sets the path to the folder for storing intermediate results.
After workflow execution this folder will be deleted.
Default: */tmp*
- `-u UID`, `--uid UID` sets the ID for DAG's unique identifier generation.
Default: *random uuid*
- `-r`, `--run` runs submitted workflow with Airflow Scheduler

Arguments `-o`, `-t` and `-u` doesn't overwrite the values set in the Job file in the fields
*output_folder*, *tmp_folder* and *uid* correspondingly.

The *submit* command will resolve all relative paths from Job file adding mandatory fields
*workflow*, *output_folder* and *uid* (if not provided) and will copy Job file to the
Jobs folder.

The *submit* command will **not** execute submitted workflow unless *-r* argument is provided.
Otherwise, make sure that *Airflow Scheduler* (and optionally *Airflow Webserver*) is running.

To start Airflow Scheduler
```bash
airflow scheduler
```
To start Airflow Webserver
```bash
airflow webserver
```

The CWL descriptor file and all input files referenced in Job file should not be moved or deleted while
workflow is running.  

## Demo mode
To get the list of the available demo workflows to run
```bash
$ cwl-airflow demo --list
```
To submit demo workflow from the list
(to execute submitted wokrflow Airflow Scheduler should be started separately)
```bash
$ cwl-airflow demo super-enhancer.cwl
```
To submit all available demo workflows
(to execute submitted wokrflows Airflow Scheduler should be started separately)
```bash
$ cwl-airflow demo --manual
```
To execute all available demo workflows (automatically starts Airflow Scheduler and Airflow Webserver)
```bash
$ cwl-airflow demo --auto
```
Additionally you can specify the following optional parameters
- `-o OUTPUT_FOLDER`, `--outdir OUTPUT_FOLDER` sets the path to the folder
   where all the output files should be moved after successful workflow execution.
   Default: current directory
- `-t TMP_FOLDER`, `--tmp TMP_FOLDER` sets the path to the folder for storing intermediate results.
After workflow execution this folder will be deleted.
Default: */tmp*
- `-u UID`, `--uid UID` sets the ID for DAG's unique identifier generation. Ignored with *--manual* or *--auto*
options. Default: *random uuid*