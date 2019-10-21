---
title: How to install Elasticsearch 7.x on openSUSE
tags: [OpenSUSE_official,Linux,Open Source]
published: true
thumbnail: http://awsmp-logos.s3.amazonaws.com/elasticsearch-logo.png
---

<p align = "center">
<img src = "https://1bpezptkft73xxds029zrs59-wpengine.netdna-ssl.com/wp-content/uploads/800x400-blog-elasticsearchscylla.jpg">
</p>


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
## [](#header-2)Import the Elasticsearch PGP Key

We sign all of our packages with the Elasticsearch Signing Key (PGP key D88E42B4, available from https://pgp.mit.edu) with fingerprint:

4609 5ACC 8548 582C 1A26 99A9 D27D 666C D88E 42B4

Download and install the public signing key:
```
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```
## [](#header-2)Installing from the RPM repository

Create a file called elasticsearch.repo in the /etc/yum.repos.d/ directory for RedHat based distributions, or in the /etc/zypp/repos.d/ directory for OpenSuSE based distributions, containing:
```
[elasticsearch-7.x]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```
And your repository is ready for use. You can now install Elasticsearch with one of the following commands:
```
sudo zypper install elasticsearch
```
# Running Elasticsearch with SysV init

Use the chkconfig command to configure Elasticsearch to start automatically when the system boots up:
```
sudo chkconfig --add elasticsearch
```
Elasticsearch can be started and stopped using the service command:
```
sudo -i service elasticsearch start
sudo -i service elasticsearch stop
```
