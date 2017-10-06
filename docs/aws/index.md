---
layout: default
title: Installing Imhotep on AWS
permalink: /docs/aws/install-java/
---

## Introduction

Before getting started, familiarize yourself with the [Imhotep architecture](../architecture-overview/).

These instructions require an AWS account and explain how to set up your S3 storage and three EC2 instance types:
    1. Zookeeper
    2. ImhotepDaemon
    2. Imhotep Frontend

For fault tolerance, you should run replicated Zookeeper instances in a quorum. See the [Zookeeper instructions](http://archive.cloudera.com/cdh5/cdh/5/zookeeper/zookeeperStarted.html) for more information.

For scalability, you may want to adapt these instructions to create an auto scaling group to run multiple ImhotepDaemon instances as needed.

## Instructions

1. [Set up S3 storage](s3-storage/)
1. [Set up SSH access and security groups](security/)
1. [Set up Zookeeper](zookeeper/)
1. [Set up ImhotepDaemon](imhotep-daemon/)
1. [Set up ImhotepFrontend](imhotep-frontend/)

Once you've completed all these steps, you will have a working Imhotep cluster with web frontend enpoints /iql and /iupload.
