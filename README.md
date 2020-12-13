# baseball stats with Z

This repository contains a simple demonstration of the Z data model
using baseball statistics as an example.

The Z data model is a new approach to data that embraces both the schema-rigid
model of relational tables as well as the semi-structured model of JSON.
[`zq`](https://github.com/brimsec/zq)
is a command-line tool for desktop search and analytics over
ZNG files, which blends UX concepts from log search and SQL.

In this example, we will take a large set of relational tables
conforming to a
(complicated relational model)[https://github.com/WebucatorTraining/lahman-baseball-mysql/blob/master/lahman-model.png]
and dump them all into a "micro data lake" (i.e., a single file on your laptop)
represented as  a file the ZNG data format (ZNG is a row-structured data format
that  implements the Z data model).

If you want to use MySQL instead, this
(github repo)[https://github.com/WebucatorTraining/lahman-baseball-mysql]
has detailed instructions for setting it all up and loading the data
into your server.

Or you can just make a ZNG file and use `zq`.

## Setup

You will need:
* the Sean Lahman data set from 2019 in CSV format,
* the `zq` CLI tool mentioned above, and
* `cj` the simple tool in this repo for converting CSV to JSON.

First, install zq.  You can download a
[pre-built binary from Brim](https://www.brimsecurity.com/download/), or
if you have Go installed, you can grab it and related tooling from zq github repo:
```
go get https://github.com/brimsec/zq
```
Next you'll need to install `cj` (a very small Go program that converts CSV to JSON):
```
go get https://github.com/mccanne/cj
```
Finally you need the data, in CSV form.  You can download the
[2019 – comma-delimited version](https://github.com/chadwickbureau/baseballdatabank/archive/master.zip)
from Sean Lahman's
[download page](http://www.seanlahman.com/baseball-archive/statistics/).

The Lahman data will appear as a zip file called `baseballdatabank-master.zip`,
which you should unzip into baseballdatabank-master and run the following command:
```
find baseballdatabank-master -name \*.csv \
    -exec echo "cat" {} "| cj" \; | sh | zq -i ndjson -o bb.zng -
```
This gives you a file called `bb.zng`,
