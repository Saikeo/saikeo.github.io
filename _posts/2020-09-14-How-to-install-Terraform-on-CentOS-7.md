---
title: How to install Teraaform on CentOS 7
tags: [Devops,Automation]
published: true
description: This is method how to install Terraform on CentOS 7
thumbnail: https://miro.medium.com/max/480/1*9FQVJwRJrzPDncseivWmKg.png
---

<p align = "center">
<img src = "https://miro.medium.com/max/480/1*9FQVJwRJrzPDncseivWmKg.png">
</p>

This is basic method how to install Terraform on CentOS 7.

# 1. Update OS and Install nesessary tools
```
# yum update
# yum install wget unzip
 ```
# 2. Download Terraform
```
# wget https://releases.hashicorp.com/terraform/0.13.2/terraform_0.13.2_linux_amd64.zip
 ```
# 3. Unpack to /usr/local/bin
```
# unzip ./terraform_0.13.2_linux_amd64.zip -d /usr/local/bin/
 ```

# + Now install is done and now let's verify terraform version
```
# terraform -v
Terraform v0.13.2
```