---
toc: true
layout: post
description: Running spark on kubernetes: Getting started
categories: [markdown]
title: Spark on Kubernetes on M1 Mac
---
## Purpose
To get started with spark on kubernetes, I had to make a few changes to steps in the excellent blog 
https://dzlab.github.io/ml/2020/07/14/spark-kubernetes/, mostly due to version changes. 

## Resources
https://dev.to/stack-labs/my-journey-with-spark-on-kubernetes-in-python-1-3-4nl3

## Steps
Assuming minikube and kubectl are installed, here are the steps followed:

```shell
minikube start --memory 8192 --cpus 6

# add the help repo for spark operator
helm repo add spark-operator https://googlecloudplatform.github.io/spark-on-k8s-operator

# create namespace, serice account and ClusterRoleBinding from 
wget https://gist.githubusercontent.com/dzlab/b546a450a9e8cfa5c8c3ff0a7c9ff091/raw/a7487fe13f96c0a5ad576aad8548c342e9781994/spark-operator.yaml
kubectl create -f spark-operator.yaml
helm install sparkoperator spark-operator/spark-operator --namespace spark-operator --set sparkJobNamespace=spark-apps,enableWebhook=true

kubectl apply -f spark-pi.yaml

# check status
kubectl describe SparkApplication spark-pi -n spark-apps
```

Contents of spark-pi.yaml (minor changes to https://github.com/GoogleCloudPlatform/spark-on-k8s-operator/blob/master/examples/spark-pi.yaml)

```yaml
apiVersion: "sparkoperator.k8s.io/v1beta2"
kind: SparkApplication
metadata:
  name: spark-pi
  namespace: spark-apps
spec:
  type: Scala
  mode: cluster
  image: "gcr.io/spark-operator/spark:v3.1.1"
  imagePullPolicy: Always
  mainClass: org.apache.spark.examples.SparkPi
  mainApplicationFile: "local:///opt/spark/examples/jars/spark-examples_2.12-3.1.1.jar"
  sparkVersion: "3.1.1"
  restartPolicy:
    type: Never
  volumes:
    - name: "test-volume"
      hostPath:
        path: "/tmp"
        type: Directory
  driver:
    cores: 1
    coreLimit: "1200m"
    memory: "512m"
    labels:
      version: 3.1.1
    serviceAccount: spark
    volumeMounts:
      - name: "test-volume"
        mountPath: "/tmp"
  executor:
    cores: 1
    instances: 1
    memory: "512m"
    labels:
      version: 3.1.1
    volumeMounts:
      - name: "test-volume"
        mountPath: "/tmp"
```

Output is:

```shell
  Type    Reason                     Age                  From            Message
  ----    ------                     ----                 ----            -------
  Normal  SparkApplicationAdded      2m44s                spark-operator  SparkApplication spark-pi was added, enqueuing it for submission
  Normal  SparkApplicationSubmitted  2m25s                spark-operator  SparkApplication spark-pi was submitted successfully
  Normal  SparkDriverRunning         2m23s                spark-operator  Driver spark-pi-driver is running
  Normal  SparkExecutorPending       100s (x2 over 100s)  spark-operator  Executor [spark-pi-46267481f0b0217b-exec-1] is pending
  Normal  SparkExecutorRunning       97s                  spark-operator  Executor [spark-pi-46267481f0b0217b-exec-1] is running
  Normal  SparkExecutorCompleted     10s                  spark-operator  Executor [spark-pi-46267481f0b0217b-exec-1] completed
  Normal  SparkDriverCompleted       8s                   spark-operator  Driver spark-pi-driver completed
  Normal  SparkApplicationCompleted  8s                   spark-operator  SparkApplication spark-pi completed
```

### submit job with spark-submit
This worked for a scala job. 

```shell
spark-submit \
--master k8s://https://127.0.0.1:50559 \
--deploy-mode cluster \
--name spark-pi \
--class org.apache.spark.examples.SparkPi \
--conf spark.executor.instances=1 \
--conf spark.kubernetes.container.image=gcr.io/spark-operator/spark:v3.1.1 \
--conf spark.kubernetes.namespace=spark-apps \
--conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
local:///opt/spark/examples/jars/spark-examples_2.12-3.1.1.jar
```

For pyspark, I had to change 
  - the image to `gcr.io/spark-operator/spark:v3.1.1-py3.6`
  - file to `local:///opt/spark/examples/src/main/python/pi.py`

```shell
spark-submit \
--master k8s://https://127.0.0.1:50559 \
--deploy-mode cluster \
--name spark-pi \
--class org.apache.spark.examples.SparkPi \
--conf spark.executor.instances=1 \
--conf spark.kubernetes.container.image=gcr.io/spark-operator/spark-py:v3.1.1 \
--conf spark.kubernetes.namespace=spark-apps \
--conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
local:///opt/spark/examples/src/main/python/pi.py
```


