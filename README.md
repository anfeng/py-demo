## INTRODUCTION

This prototype illustrates how to use Spark and TensorFlow in Python

##  BUILD TF/SPARK CONNECTOR

```
git clone https://github.com/tensorflow/ecosystem.git tf-ecosystem
pushd tf-ecosystem/hadoop
mvn clean install
popd
pushd tf-ecosystem/spark/spark-tensorflow-connector
mvn clean install
popd
```

##  SET UP ENVIRONMENT

```
export HADOOP_HOME=/Users/andyfeng/dev/hadoop-2.7.5
export SPARK_HOME=/Users/andyfeng/dev/spark-on-k8s 
export TF_SPARK_CONNECTOR=/Users/andyfeng/dev/tf-ecosystem/spark/spark-tensorflow-connector
export PATH=$HADOOP_HOME/bin:$SPARK_HOME/bin:$PATH 
export PYTHONPATH=$SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.10.6-src.zip:$PYTHONPATH
```

##  Apply Spark to analyze AWS BIN IMAGE DATASETS 

AWS BIN Image Dataset is located at S3: https://aws.amazon.com/public-datasets/amazon-bin-images/

The following command applies some ETL and SQL to extract and transform datasets into a subdataset saved in TensorFlow Record format on my S3 bucket.   

```
spark-submit --jars $HADOOP_HOME/share/hadoop/tools/lib/hadoop-aws-2.7.5.jar, \
    $HADOOP_HOME/share/hadoop/tools/lib/aws-java-sdk-1.7.4.jar, \
    $TF_SPARK_CONNECTOR/target/spark-tensorflow-connector_2.11-1.6.0.jar \ 
    ds_select.py tfsp-andyf
```
