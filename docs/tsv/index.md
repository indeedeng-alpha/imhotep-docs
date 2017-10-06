---
layout: default
title: How to Convert TSVs on the Command Line
permalink: /docs/tsv/
---

## Introduction

A typical Imhotep cluster will process incoming TSV uploads with a cron job that
runs the TsvConverter Java class. This utility scans a given directory location 
(which can be local, S3, or HDFS) for new TSV uploads and attempts to convert them.

If you wish to do manual conversion of a TSV, you can use the same utility and a temporary
directory structure. Here's how.

## Install Java

If you don't already have Java installed, install the JDK from Oracle. These instructions
have been tested with Java version 1.8.0_102, but should also work on Java 1.7.

## Install TSV Converter

    mkdir imhotepTsvConverter
    cd imhotepTsvConverter
    wget -O imhotepTsvConverter.tar.gz https://indeedeng-imhotep-build.s3.amazonaws.com/tsv-builder-1.0.1-SNAPSHOT-complete.tar.gz
    tar xzf imhotepTsvConverter.tar.gz
    ln -s $PWD/tsv-builder-* shardBuilder
    mkdir conf
    mkdir logs
    curl https://raw.githubusercontent.com/indeedeng/imhotep-tsv-converter/master/tsv-converter/src/main/resources/log4j.xml > conf/log4j.xml
    sed -i "s#/opt/imhotepTsvConverter#$PWD#" conf/log4j.xml

## Create a temporary directory structure containing your TSV

This example shows creating a structure for importing into a dataset called `nasa`. A single
day of TSV data for this dataset is placed into the directory structure for processing.

    d=`mktemp -d -p .`
    echo "Temp directory: $d"
    mkdir -p $d/tsv/incoming $d/tsv/success $d/tsv/failed $d/index $d/build
    mkdir $d/tsv/incoming/nasa
    curl https://raw.githubusercontent.com/indeedeng/imhotep/gh-pages/files/nasa_19950801.tsv > $d/tsv/incoming/nasa/nasa_19950801.tsv

## Run the converter

    export CLASSPATH="./shardBuilder/lib/*:./conf:"$CLASSPATH
    java -Dlog4j.configuration=conf/log4j.xml com.indeed.imhotep.builder.tsv.TsvConverter --index-loc $d/tsv/incoming --success-loc $d/tsv/success --failure-loc $d/tsv/failed --data-loc $d/index --build-loc $d/build
    
For the nasa example above, you should see the following lines in your output:

    INFO  [EasyIndexBuilderFromTSV] Reading TSV data from file:/.../imhotepTsvConverter/tmp.Njxp9K81x0/tsv/incoming/nasa/nasa_19950801.tsv
    INFO  [EasyIndexBuilderFromTSV] Scanning the file to detect int fields
    INFO  [EasyIndexBuilderFromTSV] Int fields detected: bytes,response,time
    INFO  [EasyIndexBuilder] Creating index...
    INFO  [EasyIndexBuilder] Calling builder loop...
    INFO  [EasyIndexBuilderFromTSV] Reading TSV data from file:/.../imhotepTsvConverter/tmp.Njxp9K81x0/tsv/incoming/nasa/nasa_19950801.tsv
    INFO  [EasyIndexBuilder] Wrote 30969 Documents
    INFO  [EasyIndexBuilder] finalizing index
    INFO  [SimpleFlamdexWriter] merging 4 readers with a total of 30969 docs
    INFO  [EasyIndexBuilder] done
    INFO  [TsvConverter] Progress: 100%

You can also review the log of the run in the `logs/shardBuilder.log`` file.

## Check the results

If your build was successful, you will have a new dataset shard directory (Flamdex format)
for the import. Here's what it looks like for the NASA example:

    $ ls $d/index/nasa/
    index19950801.00-19950802.00.20171006095305

    $ ls $d/index/nasa/index19950801.00-19950802.00.20171006095305/
    fld-allbit.strdocs    fld-host.strterms     fld-referer.strindex     fld-url.strdocs
    fld-allbit.strindex   fld-logname.strdocs   fld-referer.strterms     fld-url.strindex
    fld-allbit.strterms   fld-logname.strindex  fld-response.intdocs     fld-url.strterms
    fld-bytes.intdocs     fld-logname.strterms  fld-response.intindex64  fld-useragent.strdocs
    fld-bytes.intindex64  fld-method.strdocs    fld-response.intterms    fld-useragent.strindex
    fld-bytes.intterms    fld-method.strindex   fld-unixtime.intdocs     fld-useragent.strterms
    fld-host.strdocs      fld-method.strterms   fld-unixtime.intindex64  metadata.txt
    fld-host.strindex     fld-referer.strdocs   fld-unixtime.intterms
