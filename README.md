This prototype illustrates how to use PySpark and TensorFlow together in K8S. 

##  Apply PySpark to analyze datasets locally

AMAZON BIN Image Dataset is located at S3 bucket aft-vbi-pds (https://aws.amazon.com/public-datasets/amazon-bin-images/).
We apply [ds_select.py](https://github.com/anfeng/py-demo/blob/master/py2tf/ds_select.py) to use PySpark API to  
extract, query and transform datasets, and finally save the result subdataset in TensorFlow Record format on my S3 bucket.   

You could use my pip package pysp2tfdemo with PySpark as below. 

```
pip install pysp2tfdemo

export PYSP2TF=/usr/local/lib/python2.7/site-packages/py2tf

${SPARK_HOME}/bin/spark-submit --jars $PYSP2TF/jars/hadoop-aws-2.7.5.jar,$PYSP2TF/jars/aws-java-sdk-1.7.4.jar,$PYSP2TF/jars/spark-tensorflow-connector_2.11-1.6.0.jar $PYSP2TF/ds_select.py tfsp-andyf
```

The log will include the following section about the newly created dataset.

```
+----------+--------------------+----+------------------+----+---------------+------+-------------------+--------+
|      asin|                name|unit|             value|unit|          value|  unit|              value|quantity|
+----------+--------------------+----+------------------+----+---------------+------+-------------------+--------+
|B01EM75DEO|Little Artist- Pa...|  IN|     0.99999999898|  IN|11.749999988015|pounds|0.14999999987421142|       2|
|B00VVLEKY4|Lacdo 13-13.3 Inc...|  IN|     1.49999999847|  IN| 13.99999998572|pounds|               0.25|       3|
|B00TKQEZV0|Enimay Hookah Hos...|  IN|    2.599999997348|  IN|13.199999986536|pounds|               0.25|       5|
|B01DV2OJYG|Best Microfiber C...|  IN|     5.99999999388|  IN| 11.99999998776|pounds| 1.8999999984066778|       3|
|B013HWXMJS|DocBear Bathroom ...|  IN|1.3999999985719997|  IN| 15.99999998368|pounds| 0.3499999997064932|       2|
|B00VXEWR6W|Full Body Moistur...|  IN|     3.49999999643|  IN| 3.899999996022|pounds|  1.410944074725587|       3|
|B01BBGYN0O|Simplicity Women'...|  IN|    2.899999997042|  IN|15.599999984088|pounds| 0.5499999995387752|       3|
|1629054755|Horses Wall Calen...|  IN|     0.42913385783|  IN| 13.38188975013|pounds|    0.6999897280762|       2|
|B00T3ROXI6|AmazonBasics Clea...|  IN|    1.249999998725|  IN| 11.49999998827|pounds| 1.6499999986163254|       3|
+----------+--------------------+----+------------------+----+---------------+------+-------------------+--------+
```

Here is how pysp2tfdemo pip package was created. 
```
rm -r dist/*
python setup.py sdist bdist_wheel 
twine upload dist/*
```

## Apply PySpark to analyze datasets with K8S cluster

Recent community effort has enabled PySpark to be launched on k8s cluster. You could apply the following step to build it. 

```
git clone https://github.com/apache-spark-on-k8s/spark.git spark-k8s
pushd spark-k8s
build/mvn -Pkubernetes -DskipTests clean package
```

We will use the k8s enabled Spark as SPARK_HOME, and apply a docker image "afeng95014/pysp2tfdemo:0.1" as our demo app.
The following commands demonstrate how Spark could be used in k8s cluster to access s3 dataset. It's very similar to our local deployment in previous section. You should see the identical result.
```
export SPARK_HOME=....
export K8S_MASTER=...

export PYSP2TF=local:///usr/lib/python2.7/site-packages/py2tf
export PYTHONPATH=$SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.10.6-src.zip:$PYTHONPATH

${SPARK_HOME}/bin/spark-submit \
--deploy-mode cluster \
--kubernetes-namespace default \
--master k8s://${K8S_MASTER} \
--conf spark.executor.instances=1 \
--conf spark.app.name=pyspark-tf-demo \
--conf spark.kubernetes.driver.docker.image=afeng95014/pysp2tfdemo:0.1 \
--conf spark.kubernetes.executor.docker.image=kubespark/spark-executor-py:v2.2.0-kubernetes-0.5.0 \
--jars $PYSP2TF/jars/hadoop-aws-2.7.5.jar,$PYSP2TF/jars/aws-java-sdk-1.7.4.jar,$PYSP2TF/jars/spark-tensorflow-connector_2.11-1.6.0.jar \
$PYSP2TF/ds_select.py tfsp-andyf $AWS_REGION $AWS_ACCESS_KEY_ID $AWS_SECRET_ACCESS_KEY
```

You could access application's driver log via kubectl logs with your POD, ex
```
kubectl -n=default logs -f pyspark-tf-demo-1521653814399-driver
```

Docker image "afeng95014/pysp2tfdemo:0.1" was created and published as below.
```
cp dockerfiles/Dockerfile ${SPARK_HOME}
pushd ${SPARK_HOME}
docker build . -f Dockerfile -t afeng95014/pysp2tfdemo:0.1
docker push afeng95014/pysp2tfdemo:0.1
popd
```

