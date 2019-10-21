---
title: How to install Elasticsearch 7.x on openSUSE
tags: [OpenSUSE_official,Linux,Open Source]
published: true
thumbnail: http://awsmp-logos.s3.amazonaws.com/elasticsearch-logo.png
---

<p align = "center">
<img src = "https://1bpezptkft73xxds029zrs59-wpengine.netdna-ssl.com/wp-content/uploads/800x400-blog-elasticsearchscylla.jpg">
</p>

This is basic knowledge about installation Elasticsearch 7.x on openSUSE Leap 15.1

# Introduction

Elasticsearch 7.0.0 has something for everyone. At Elastic, we constantly talk about speed, scale, and relevance: it's in our source code. Elasticsearch 7.0 goes to exemplify this, as it's the fastest, safest, most resilient, easiest to use version of Elasticsearch ever, and it comes with a boatload of enhancements and new features.

Before installing we need to check java version on openSUSE:

```
saikeo@openSUSE:~> java -version
If 'java' is not a typo you can use command-not-found to lookup the package that contains it, like this:
    cnf java
saikeo@openSUSE:~> 
```
In my case I didn't java yet. So we need to install java first.
For openSUSE Leap 15.1 run the following as root:
```
zypper addrepo https://download.opensuse.org/repositories/Java:Factory/openSUSE_Leap_15.1/Java:Factory.repo
zypper refresh
zypper install java-1_8_0-openjdk
```
Now lets check java version again:
```
saikeo@openSUSE:~> java -version
openjdk version "1.8.0_222"
OpenJDK Runtime Environment (IcedTea 3.13.0) (build 1.8.0_222-b10 suse-lp151.334.1-x86_64)
OpenJDK 64-Bit Server VM (build 25.222-b10, mixed mode)
saikeo@openSUSE:~>
```
## [](#header-2)Download Elasticsearch 7.0
Go to download Elasticsearch via this link https://www.elastic.co/downloads/elasticsearch
```
sudo wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.4.0-linux-x86_64.tar.gz
```
## [](#header-2)Installation

Extract installation file by using the follwing command:
```
sudo tar -zxvf elasticsearch-7.4.0-linux-x86_64.tar.gz
```
after extract now go to inside folder and run Elasticsearch:
```
saikeo@openSUSE:~> cd elasticsearch-7.4.0/bin/
saikeo@openSUSE:~/elasticsearch-7.4.0/bin> ./elasticsearch
```
You will get the error output as the following:
```
saikeo@openSUSE:~/elasticsearch-7.4.0/bin> ./elasticsearch
future versions of Elasticsearch will require Java 11; your Java version from [/usr/lib64/jvm/java-1.8.0-openjdk-1.8.0/jre] does not meet this requirement
Exception in thread "main" java.nio.file.AccessDeniedException: /home/saikeo/elasticsearch-7.4.0/config/jvm.options
	at sun.nio.fs.UnixException.translateToIOException(UnixException.java:84)
	at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:102)
	at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:107)
	at sun.nio.fs.UnixFileSystemProvider.newByteChannel(UnixFileSystemProvider.java:214)
	at java.nio.file.Files.newByteChannel(Files.java:361)
	at java.nio.file.Files.newByteChannel(Files.java:407)
	at java.nio.file.spi.FileSystemProvider.newInputStream(FileSystemProvider.java:384)
	at java.nio.file.Files.newInputStream(Files.java:152)
	at org.elasticsearch.tools.launchers.JvmOptionsParser.main(JvmOptionsParser.java:61)
saikeo@openSUSE:~/elasticsearch-7.4.0/bin> 
```
This error occurs because of file permission. We can check file permission by the following command:
```
saikeo@openSUSE:~> ll
total 285676
drwxr-xr-x 1 saikeo users         0 Oct 15 15:11 bin
drwxr-xr-x 1 root   root        144 Oct 21 15:32 elasticsearch-7.4.0
```
We need to give permission to current users.
```
saikeo@openSUSE:~> sudo chown -R saikeo:root elasticsearch-7.4.0
```
Now lets check the permission again:
```
saikeo@openSUSE:~> ll
total 285676
drwxr-xr-x 1 saikeo users         0 Oct 15 15:11 bin
drwxr-xr-x 1 saikeo root        144 Oct 21 15:32 elasticsearch-7.4.0
```
Go to run Elasticsearch again:
```
saikeo@openSUSE:~/elasticsearch-7.4.0/bin> ./elasticsearch
future versions of Elasticsearch will require Java 11; your Java version from [/usr/lib64/jvm/java-1.8.0-openjdk-1.8.0/jre] does not meet this requirement
[2019-10-21T15:46:12,585][INFO ][o.e.e.NodeEnvironment    ] [openSUSE] using [1] data paths, mounts [[/home (/dev/sda2)]], net usable_space [91.2gb], net total_space [97.9gb], types [btrfs]
[2019-10-21T15:46:12,589][INFO ][o.e.e.NodeEnvironment    ] [openSUSE] heap size [1007.3mb], compressed ordinary object pointers [true]
[2019-10-21T15:46:12,595][INFO ][o.e.n.Node               ] [openSUSE] node name [openSUSE], node ID [JF8JBJnOTAiHtEcYXSJvVg], cluster name [elasticsearch]
```
Now we can run Elasticsearch and we can verify this installation by using the following command:
```
saikeo@openSUSE:~> sudo curl http://localhost:9200?pretty
{
  "name" : "openSUSE",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "0S-a83_cRwSDpbIH3njlXQ",
  "version" : {
    "number" : "7.4.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "22e1767283e61a198cb4db791ea66e3f11ab9910",
    "build_date" : "2019-09-27T08:36:48.569419Z",
    "build_snapshot" : false,
    "lucene_version" : "8.2.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
saikeo@openSUSE:~>
```

**Congrats now you have successful install Elasticsearch on openSUSE Leap 15.1**