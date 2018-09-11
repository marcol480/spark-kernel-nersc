# Apache Spark kernel for Cori@NERSC

## The kernels

Log on Cori@NERSC, and run the script makekernel.py to create a Jupyter kernel for using pyspark in notebooks at NERSC:

```
$ python makekernel.py --help

usage: makekernel.py [-h] -kernelname KERNELNAME
                     [-spark_version SPARK_VERSION]
                     [-pyspark_args PYSPARK_ARGS] [--local]

Create Jupyter kernels for using Apache Spark at NERSC

optional arguments:
  -h, --help            show this help message and exit
  -kernelname KERNELNAME
                        Name of the Jupyter kernel to be displayed
  -spark_version SPARK_VERSION
                        Version of Apache Spark. Available: 2.0.0, 2.1.0,
                        2.3.0. Note that 2.0.0, and 2.1.0 are standard
                        kernels, while 2.3.0 makes use of shifter to run.
                        Default is 2.3.0.
  -pyspark_args PYSPARK_ARGS
                        Submission arguments for pyspark. See
                        https://spark.apache.org/docs/latest/submitting-
                        applications.html for more information. Default is "--
                        master local[4]".
  --local               If specified, kernel and startup scripts will be dump
                        on the current working directory instead of the NERSC
                        kernel folder. Default is False.
```

The kernel will be stored at `$HOME/.ipython/kernels/`.

### Apache Spark version 2.3.0+ (recommended)

For Spark version 2.3.0+, Spark ran inside of Shifter.
Note that he directory `/global/cscratch1/sd/<user>/tmpfiles` will be created to store temporary files used by Spark.

### Apache Spark version <= 2.1.0 (old)

For Spark version <= 2.1.0, a startup script will be created in addition to the kernel,
in order to load the Spark module and launch a Spark cluster before launching the notebook:

```bash
#!/bin/bash
module load spark/<version>
start-all.sh
/usr/common/software/python/3.5-anaconda/bin/python -m ipykernel $@
```

The startup scripts will be stored with the kernel at `$HOME/.ipython/kernels/`.
We support only Python 3.5 for the moment.

### Pyspark arguments

Pyspark most common arguments include:

- `--master local[ncpu]`: the number of CPU to use.
- `--conf spark.eventLog.enabled=true` `--conf spark.eventLog.dir=<file:/dir>` `--conf spark.history.fs.logDirectory=<file:/dir>`: store the logs. By default Spark will put event logs in `file://$SCRATCH/spark/spark_event_logs`, and you will need to create this directory the very first time you start up Spark.
- `--packages ...`: Any package you want to use. For example, you can try out the great [spark-fits](https://github.com/astrolabsoftware/spark-fits) connector using `--packages com.github.astrolabsoftware:spark-fits_2.11:0.6.0`!

### Access the logs from the Spark UI

Once your job is terminated, you can have access to the log via the Spark history UI. Log on Cori, and load the spark/history module:

```
module load spark/history
```

Then go to the folder where the logs are stored, and launch the history server and follow the URL:

```
# This is the default location
cd $SCRATCH/spark/spark_event_logs
./run_history_server.sh
```

Once you are done, just stop the server by executing `./run_history_server.sh --stop`. 

### Note concerning resources
    
    The large-memory login node used by https://jupyter-dev.nersc.gov/
    is a shared resource, so please be careful not to use too many CPUs
    or too much memory.

    That means avoid using `--master local[*]` in your kernel, but limit
    the resources to a few core. Typically `--master local[4]` is enough for
    prototyping a program.

## Use pyspark in JupyterLab

Connect to https://jupyter-dev.nersc.gov/hub/login and create a notebook with
the kernel you just created:

<p align="center"><img width="600" src="https://github.com/astrolabsoftware/spark-kernel-nersc/raw/master/pic/load_kernel.png"/> </p>

<p align="center"><img width="600" src="https://github.com/astrolabsoftware/spark-kernel-nersc/raw/master/pic/spark_notebook.png"/> </p>

## Known issue

When switching kernels, and re-running a notebook, we often get the following error:

```
Py4JJavaError                             Traceback (most recent call last)
/usr/local/bin/spark-2.3.0/python/pyspark/sql/utils.py in deco(*a, **kw)
     62         try:
---> 63             return f(*a, **kw)
     64         except py4j.protocol.Py4JJavaError as e:

/usr/local/bin/spark-2.3.0/python/lib/py4j-0.10.6-src.zip/py4j/protocol.py in 
get_return_value(answer, gateway_client, target_id, name)
    319                     "An error occurred while calling {0}{1}{2}.\n".
--> 320                     format(target_id, ".", name), value)
    321             else:

Py4JJavaError: An error occurred while calling o83.load.
: org.apache.spark.sql.AnalysisException: java.lang.RuntimeException: 
java.lang.RuntimeException: Unable to instantiate 
org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient;
	at 
org.apache.spark.sql.hive.HiveExternalCatalog.withClient(
HiveExternalCatalog.scala:106)
	at 
	
	... (long... very long)
	
AnalysisException: 'java.lang.RuntimeException: java.lang.RuntimeException:
Unable to instantiate 
org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient;'
```

Just go to the folder where the notebook is running, and delete the temporary folder:

```
rm -r metastore_db
```

Then restart your kernel, and all should be fine.

## Thanks to

- The NERSC consulting and support team for their great help!