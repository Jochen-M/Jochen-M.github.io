---
title: Flink学习笔记
date: 2017-05-14 22:36:03
tags:
    - Flink
categories:
    - 大数据
---

Apache Flink is an open source platform for distributed stream and batch data processing. Flink’s core is a streaming dataflow engine that provides data distribution, communication, and fault tolerance for distributed computations over data streams. Flink also builds batch processing on top of the streaming engine, overlaying native iteration support, managed memory, and program optimization.

<!-- more -->

#### Building Flink from Source
To clone from git, enter:
```
git clone https://github.com/apache/flink
```
The simplest way of building Flink is by running
```
// 1)For maven 3.0.x, 3.1.x, and 3.2.x
//   In the root directory of Flink code base:
mvn clean install -DskipTests

// 2)For maven 3.3.x
//   The build has to be done in two steps:
//   First in the base directory, then in the distribution project
mvn clean install -DskipTests
cd flink-dist
mvn clean install
```
*Note:* To check your Maven version, run <code>mvn --version</code>.

#### Create Project
```
mvn archetype:generate                               \
    -DarchetypeGroupId=org.apache.flink              \
    -DarchetypeArtifactId=flink-quickstart-java      \
    -DarchetypeVersion=1.4-SNAPSHOT
```
This allows you to name your newly created project. It will interactively ask you for the groupId, artifactId, and package name.

#### Build Project
Go to your project directory and enter the command:
```
mvn clean install -Pbuild-jar 
```
You will find a jar that runs on every Flink cluster with a compatible version, target/original-your-artifact-id-your-version.jar. There is also a fat-jar in target/your-artifact-id-your-version.jar which, additionally, contains all dependencies that were added to the Maven project.

#### DataStream API Examples
##### Map: DataStream → DataStream
