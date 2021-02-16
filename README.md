# baseball stats with Z

> NOTE: This is a work in progress and needs to be updated to the latest
> version of zq with the baked ZSON format and the shape aggregator.

This repository contains a simple demonstration of the Z data model
using baseball statistics as an example.

The Z data model is a new approach to data that embraces both the schema-rigid
model of relational tables as well as the semi-structured model of JSON.
[`zq`](https://github.com/brimsec/zq)
is a command-line tool for desktop search and analytics over
ZNG files, which blends UX concepts from log search and SQL.

In this example, we will take a large set of relational tables
conforming to a
[complicated relational model](https://github.com/WebucatorTraining/lahman-baseball-mysql/blob/master/lahman-model.png)
and dump them all into a "micro data lake" (i.e., a single file on your laptop)
represented as  a file the ZNG data format (ZNG is a row-structured data format
that  implements the Z data model).

If you want to use MySQL instead, this
[github repo](https://github.com/WebucatorTraining/lahman-baseball-mysql)
has detailed instructions for setting it all up and loading the data
into your server.

R is also a great way to
[wrangle this data](https://cran.r-project.org/web/packages/Lahman/Lahman.pdf).

Or you can just make a ZNG file and use `zq`.

## Setup

You will need:
* the Sean Lahman data set from 2019 in CSV format,
* the `zq` CLI tool mentioned above, and
* `cz` the simple tool in this repo for converting CSV to ZNG.

First, install zq.  You can download a
[pre-built binary from Brim](https://www.brimsecurity.com/download/), or
if you have Go installed, you can grab it and related tooling from zq github repo:
```
go get github.com/brimsec/zq
```
Next you'll need to install `cz` (a very small Go program that converts CSV to JSON):
```
go get github.com/mccanne/cz
```
Finally you need the data, in CSV form.  You can download the
[2019 – comma-delimited version](https://github.com/chadwickbureau/baseballdatabank/archive/master.zip)
from Sean Lahman's
[download page](http://www.seanlahman.com/baseball-archive/statistics/).

The Lahman data will appear as a zip file called `baseballdatabank-master.zip`,
which you should unzip into ``./baseballdatabank-master` and run the following command:
```
unzip baseballdatabank-master.zip
find baseballdatabank-master -name \*.csv -exec echo "cat" {} "| cz" \; | sh | zq -o bb.zng nullify -
```

This gives you a file called `bb.zng`, which holds all of the tables from
the Lahamn data set as a stream of heterogenous records in accordance with
the Z data model.  The file should be 25222375 bytes.  You can also confirm
the number of rows that we collected:
```
zq -f table "count()" bb.zng
```
which should output
```
COUNT
586970
```

## Examples

The nice thing about zq is it's kind of like search but it's also OLAP-y so
you can do analytics with it too.

Let's search for Joe Dimaggio...
```
zq -f zson dimaggio bb.zng
===
{"bats":"R","bbrefID":"dimagdo01","birthCity":"San Francisco","birthCountry":"USA","birthDay":12,"birthMonth":2,"birthState":"CA","birthYear":1917,"deathCity":"Marion","deathCountry":"USA","deathDay":8,"deathMonth":5,"deathState":"MA","deathYear":2009,"debut":"1940-04-16","finalGame":"1953-05-09","height":69,"nameFirst":"Dom","nameGiven":"Dominic Paul","nameLast":"DiMaggio","playerID":"dimagdo01","retroID":"dimad101","throws":"R","weight":168}
{"bats":"R","bbrefID":"dimagjo01","birthCity":"Martinez","birthCountry":"USA","birthDay":25,"birthMonth":11,"birthState":"CA","birthYear":1914,"deathCity":"Hollywood","deathCountry":"USA","deathDay":8,"deathMonth":3,"deathState":"FL","deathYear":1999,"debut":"1936-05-03","finalGame":"1951-09-30","height":74,"nameFirst":"Joe","nameGiven":"Joseph Paul","nameLast":"DiMaggio","playerID":"dimagjo01","retroID":"dimaj101","throws":"R","weight":193}
{"bats":"R","bbrefID":"dimagvi01","birthCity":"Martinez","birthCountry":"USA","birthDay":6,"birthMonth":9,"birthState":"CA","birthYear":1912,"deathCity":"North Hollywood","deathCountry":"USA","deathDay":3,"deathMonth":10,"deathState":"CA","deathYear":1986,"debut":"1937-04-19","finalGame":"1946-06-06","height":71,"nameFirst":"Vince","nameGiven":"Vincent Paul","nameLast":"DiMaggio","playerID":"dimagvi01","retroID":"dimav101","throws":"R","weight":183}
```
If you have `jq` installed, you can view this JSON a bit more clearly...
```
zq -f ndjson dimaggio bb.zng | jq
===
{
  "bats": "R",
  "bbrefID": "dimagdo01",
  "birthCity": "San Francisco",
  "birthCountry": "USA",
  "birthDay": 12,
  "birthMonth": 2,
  "birthState": "CA",
  "birthYear": 1917,
  "deathCity": "Marion",
  "deathCountry": "USA",
  "deathDay": 8,
  "deathMonth": 5,
  "deathState": "MA",
  "deathYear": 2009,
  "debut": "1940-04-16",
  "finalGame": "1953-05-09",
  "height": 69,
  "nameFirst": "Dom",
  "nameGiven": "Dominic Paul",
  "nameLast": "DiMaggio",
  "playerID": "e",
  "retroID": "dimad101",
  "throws": "R",
  "weight": 168
}
{
  "bats": "R",
  "bbrefID": "dimagjo01",
  "birthCity": "Martinez",
  "birthCountry": "USA",
  "birthDay": 25,
  "birthMonth": 11,
  "birthState": "CA",
  "birthYear": 1914,
  "deathCity": "Hollywood",
  "deathCountry": "USA",
  "deathDay": 8,
  "deathMonth": 3,
  "deathState": "FL",
  "deathYear": 1999,
  "debut": "1936-05-03",
  "finalGame": "1951-09-30",
  "height": 74,
  "nameFirst": "Joe",
  "nameGiven": "Joseph Paul",
  "nameLast": "DiMaggio",
  "playerID": "dimagjo01",
  "retroID": "dimaj101",
  "throws": "R",
  "weight": 193
}
{
  "bats": "R",
  "bbrefID": "dimagvi01",
  "birthCity": "Martinez",
  "birthCountry": "USA",
  "birthDay": 6,
  "birthMonth": 9,
  "birthState": "CA",
  "birthYear": 1912,
  "deathCity": "North Hollywood",
  "deathCountry": "USA",
  "deathDay": 3,
  "deathMonth": 10,
  "deathState": "CA",
  "deathYear": 1986,
  "debut": "1937-04-19",
  "finalGame": "1946-06-06",
  "height": 71,
  "nameFirst": "Vince",
  "nameGiven": "Vincent Paul",
  "nameLast": "DiMaggio",
  "playerID": "dimagvi01",
  "retroID": "dimav101",
  "throws": "R",
  "weight": 183
}
```
> TBD: we should make the zq `-pretty` for ZSON also work for JSON
> so we don't need to pipe to jq here

Hmm, turns out there were two other Dimaggios.

The nice thing here is that you didn't have to look at a data model
or sift through SQL schemas
to figure out what the data looks like.  Just throw some searches at zq.

You can do SQL-like projections using log-search style syntax and view
the data in other formats, e.g.,
```
zq -f table "dimaggio | cut nameGiven,nameLast,bbrefID" bb.zng
===
NAMEGIVEN    NAMELAST BBREFID
Dominic Paul DiMaggio dimagdo01
Joseph Paul  DiMaggio dimagjo01
Vincent Paul DiMaggio dimagvi01
```
I chose the `bbrefID` field in this cut thinking it might be a join key.
Let's look for the Vincent Paul using his ID:
```
zq -f table dimagvi01 bb.zng
```
Interesting. This gives a lot of data so clearly that ID is the key to
all the rows.  Note here there is one row from a first table, then multiple
rows from a second table and so forth (the results here are truncated at `...`):
```
AWARDID                    LGID NOTES PLAYERID  TIE YEARID
Baseball Magazine All-Star NL   CF    dimagvi01 -   1943
A  CS DP E  G   GS INNOUTS PB PO  POS SB WP ZR LGID PLAYERID  STINT TEAMID YEARID
21 -  2  7  130 -  -       -  351 OF  -  -  -  NL   dimagvi01 1     BSN    1937
0  -  0  0  1   -  -       -  4   2B  -  -  -  NL   dimagvi01 1     BSN    1938
19
...
```
What if we could see just one row from each table to see how many tables there
are here?  In Z, each row has a type (i.e., a "record type").  When we used  
zq to convert the CSV, each file induces a record type for each of the
rows in that file which is more or less equivalent to the table's schema.

In Z, types are "first class", meaning there are "type values", which sounds
abstract, but is really using because anything that's a value can be an
aggregation key.  So, I can do a search for the key `dimagvi01` and
group the results by the type of each row (aka Z record), and in this case
apply the `first()` aggregation function to just give the first row that
appears for each type, as follows:
```
zq -f ndjson "dimagvi01 | any(.) by typeof(.)" bb.zng | jq
```
Note that `.` refers the entire row of each row in the query and any
is an "aggregation" to reduces a group of records to any one of the records
in the group.  This is a handy way to pick out a random value
in a group by.

This tables are very wide so I used JSON output with jq and we get this:
```
{
  "any": {
    "awardID": "Baseball Magazine All-Star",
    "lgID": "NL",
    "notes": "CF",
    "playerID": "dimagvi01",
    "tie": null,
    "yearID": 1943
  },
  "typeof": "{playerID:string,awardID:string,yearID:float64,lgID:string,tie:string,notes:string}"
}
{
  "any": {
    "A": 21,
    "CS": null,
    "DP": 2,
    "E": 7,
    "G": 130,
    "GS": null,
    "InnOuts": null,
    "PB": null,
    "PO": 351,
    "POS": "OF",
    "SB": null,
    "WP": null,
    "ZR": null,
    "lgID": "NL",
    "playerID": "dimagvi01",
    "stint": 1,
    "teamID": "BSN",
    "yearID": 1937
  },
  "typeof": "{playerID:string,yearID:float64,stint:float64,teamID:string,lgID:string,POS:string,G:float64,GS:float64,InnOuts:float64,PO:float64,A:float64,E:float64,DP:float64,PB:float64,WP:float64,SB:float64,CS:float64,ZR:float64}"
}
{
  "any": {
    "bats": "R",
    "bbrefID": "dimagvi01",
    "birthCity": "Martinez",
    "birthCountry": "USA",
    "birthDay": 6,
    "birthMonth": 9,
    "birthState": "CA",
    "birthYear": 1912,
    "deathCity": "North Hollywood",
    "deathCountry": "USA",
    "deathDay": 3,
    "deathMonth": 10,
    "deathState": "CA",
    "deathYear": 1986,
    "debut": "1937-04-19",
    "finalGame": "1946-06-06",
    "height": 71,
    "nameFirst": "Vince",
    "nameGiven": "Vincent Paul",
    "nameLast": "DiMaggio",
    "playerID": "dimagvi01",
    "retroID": "dimav101",
    "throws": "R",
    "weight": 183
  },
  "typeof": "{playerID:string,birthYear:float64,birthMonth:float64,birthDay:float64,birthCountry:string,birthState:string,birthCity:string,deathYear:float64,deathMonth:float64,deathDay:float64,deathCountry:string,deathState:string,deathCity:string,nameFirst:string,nameLast:string,nameGiven:string,weight:float64,height:float64,bats:string,throws:string,debut:string,finalGame:string,retroID:string,bbrefID:string}"
}
{
  "any": {
    "GS": 126,
    "G_1b": 0,
    "G_2b": 0,
    "G_3b": 0,
    "G_all": 132,
    "G_batting": 132,
    "G_c": 0,
    "G_cf": 129,
    "G_defense": 130,
    "G_dh": 0,
    "G_lf": 0,
    "G_of": 130,
    "G_p": 0,
    "G_ph": 2,
    "G_pr": 0,
    "G_rf": 1,
    "G_ss": 0,
    "lgID": "NL",
    "playerID": "dimagvi01",
    "teamID": "BSN",
    "yearID": 1937
  },
  "typeof": "{yearID:float64,teamID:string,lgID:string,playerID:string,G_all:float64,GS:float64,G_batting:float64,G_defense:float64,G_p:float64,G_c:float64,G_1b:float64,G_2b:float64,G_3b:float64,G_ss:float64,G_lf:float64,G_cf:float64,G_rf:float64,G_of:float64,G_dh:float64,G_ph:float64,G_pr:float64}"
}
{
  "any": {
    "awardID": "MVP",
    "lgID": "NL",
    "playerID": "dimagvi01",
    "pointsMax": 336,
    "pointsWon": 10,
    "votesFirst": 0,
    "yearID": 1941
  },
  "typeof": "{awardID:string,yearID:float64,lgID:string,playerID:string,pointsWon:float64,pointsMax:float64,votesFirst:float64}"
}
{
  "any": {
    "2B": 18,
    "3B": 4,
    "AB": 493,
    "BB": 39,
    "CS": null,
    "G": 132,
    "GIDP": 9,
    "H": 126,
    "HBP": 1,
    "HR": 13,
    "IBB": null,
    "R": 56,
    "RBI": 69,
    "SB": 8,
    "SF": null,
    "SH": 6,
    "SO": 111,
    "lgID": "NL",
    "playerID": "dimagvi01",
    "stint": 1,
    "teamID": "BSN",
    "yearID": 1937
  },
  "typeof": "{playerID:string,yearID:float64,stint:float64,teamID:string,lgID:string,G:float64,AB:float64,R:float64,H:float64,\"2B\":float64,\"3B\":float64,HR:float64,RBI:float64,SB:float64,CS:float64,BB:float64,SO:float64,IBB:float64,HBP:float64,SH:float64,SF:float64,GIDP:float64}"
}
{
  "any": {
    "Gcf": 129,
    "Glf": 0,
    "Grf": 1,
    "playerID": "dimagvi01",
    "stint": 1,
    "yearID": 1937
  },
  "typeof": "{playerID:string,yearID:float64,stint:float64,Glf:float64,Gcf:float64,Grf:float64}"
}
{
  "any": {
    "GP": 1,
    "gameID": "ALS194307130",
    "gameNum": 0,
    "lgID": "NL",
    "playerID": "dimagvi01",
    "startingPos": null,
    "teamID": "PIT",
    "yearID": 1943
  },
  "typeof": "{playerID:string,yearID:float64,gameNum:float64,gameID:string,teamID:string,lgID:string,GP:float64,startingPos:float64}"
}
```

What's cool here is you don't have to look at any models or schemas.  Just
throw all the data into a zng file and run type-oriented queries to
_introspectively_ discover the shape of the data.

Ok, you're wondering did we find all of the schemas here by looking
at this one player ID?  Maybe there were other tables in the data
set we don't know about.  Let's check...

If we count the schemas from our search...
```
zq -f table "dimagvi01 | any(.) by typeof(.) | count()" bb.zng
===
COUNT
8
```
Hmm, weren't there a lot more CSV files when we unzip'd the data archive?
Maybe `dimagvi01` isn't in every table.  Let's count up all of record types:
```
zq -f table "by typeof(.) | count()" bb.zng
===
COUNT
25
```
That sounds about right is in lines with all tables in the
[the SQL model mentioned above.](https://github.com/WebucatorTraining/lahman-baseball-mysql/blob/master/lahman-model.png)

> Note that the SQL model actually has a few more tables as it turns out the
> same of the tables share exactly the same schema and the shell command
> from above lost the information about which file held which CSV table.
> In other article, we will write about how you can shape and tag your
> data using Z when you transform or ingest data from CSV or JSON into Z.

How about we just select a few fields and show them
as a table...
```
zq -f table "dimaggio | cut nameGiven,nameLast,weight,year" bb.zng
```

### back to the join key

Ok, we had guessed bbrefID was a join key, but what are the other columns
where we might want to join to?

> We could look through things by hand but it would be nice to have a columnsOf(value)
> function that would give field names that matched the given value as an type ([string])

In the meantime, we can look at all the types of that have `dimagvi01` somewhere
in the row and eyeball... we can get a little help from grep...

```
zq -f ndjson "dimagvi01 | any(.) by typeof(.)" bb.zng | jq | grep dimagvi01
===
    "playerID": "dimagvi01",
    "playerID": "dimagvi01",
    "playerID": "dimagvi01",
    "playerID": "dimagvi01",
    "playerID": "dimagvi01",
    "playerID": "dimagvi01",
    "bbrefID": "dimagvi01",
    "playerID": "dimagvi01",
    "playerID": "dimagvi01",
```

Ah okay, it's really about the playerID field.  Let's pivot on that...

```
zq -f ndjson "count() by playerID" bb.zng
===
{"count":5,"playerID":"beldeir01"}
{"count":5,"playerID":"smithja01"}
{"count":7,"playerID":"graharo01"}
{"count":15,"playerID":"fowlebo01"}
{"count":11,"playerID":"stephwa01"}
{"count":6,"playerID":"metcato01"}
{"count":13,"playerID":"johnsmi04"}
{"count":9,"playerID":"baezsa01"}
{"count":24,"playerID":"prothdo01"}
{"count":32,"playerID":"colmafr01"}
{"count":22,"playerID":"bacsimi02"}
{"count":39,"playerID":"rominau01"}
{"count":70,"playerID":"bondto01"}
{"count":13,"playerID":"mclauwa01"}
{"count":9,"playerID":"appleed01"}
{"count":49,"playerID":"harrimi01"}
{"count":77,"playerID":"bellgu01"}
...
```
That's a big list... how many players are there?
```
zq -f table "count() by playerID | count()" bb.zng
===
COUNT
20091
```
20k sounds about right.


```
zq -f ndjson "exists(RBI,HR) | by typeof(.)" bb.zng
zq -f ndjson "exists(RBI,HR) | any(.) by typeof(.) | cut fields(any)" bb.zng
zq -f ndjson "exists(RBI,HR) | not exists(stint) | any(.) by typeof(.) | cut fields(any)" bb.zng
```

Now grab the relevant type...
```
 zq -f zson -pretty=0 "exists(RBI,HR) | not exists(stint) | by typeof(.)" bb.zng
```

### Webucator examples

From [Webucator](https://github.com/WebucatorTraining/lahman-baseball-mysql),
here's "most homeruns in 1977"...

```
SELECT p.nameFirst, p.nameLast, b.HR, t.name AS team, b.yearID
FROM batting b
    JOIN people p ON p.playerID = b.playerID
    JOIN teams t ON t.ID = b.team_ID
WHERE b.YearID = 1977
ORDER BY b.HR DESC
LIMIT 5;
```
Let's add some "table" names to the data for convenience (this could have been
done at "ingest" time ... more on that later).

fields()
has()
is()
sort(fields())

XXX need switch for this

```
zq "( AB!=null OR AB=null | put table=batting ; " bb.zng > bb2.zng
```

We don't have different tables but the rows in the zng file aren't denormalized
so we still need to do joins (i.e., self joins passing over the same data)
to pull this type of output together.

```
yearID=1977 | nameFirst, nameList  | sort -r HR | head 5
```

# NOTES

## analytics
```
zq -f ndjson 'count() by teamID' bb.zng
zq -f ndjson 'count() by teamID | sort teamID' bb.zng
zq -f ndjson 'count() by teamID | sort teamID | all=collect(to_lower(teamID))' bb.zng
zq -f ndjson 'count() by teamID | sort teamID | all=collect(to_lower(teamID)) | cut ana=all[1]' bb.zng
zq -f ndjson 'count() by teamID | sort teamID | all=collect(to_lower(teamID)) | cut last10=all[-10:] | put len(last10)' bb.zng
```

## search

zq -f ndjson 'dimagg' bb.zng
zq -f ndjson 'yearID < 1930 | head 5' bb.zng

## combined search and analytics

zq -f ndjson 'yearID < 1930 | union(teamID) by yearID | sort yearID' bb.zng

## shape exploration

zq -f ndjson 'by typeof(.)' bb.zng | jq
zq -f ndjson 'by typeof(.) | count()' bb.zng | jq
zq -f ndjson 'typeof(.)="{yearID:float64,teamID:string,lgID:string,playerID:string,salary:float64}"' bb.zng | jq | less

zq -o salaries.zng 'typeof(.)="{yearID:float64,teamID:string,lgID:string,playerID:string,salary:float64}"' bb.zng
zq -f ndjson 'count()' salaries.zng
zq -f ndjson 'by typeof(.)' salaries.zng

zq -f ndjson 'typeof(.)="{yearID:float64,teamID:string,lgID:string,playerID:string,salary:float64}" or typeof(.)="{schoolID:string,name_full:string,city:string,state:string,country:string}"' bb.zng | jq | less
zq -f ndjson 'typeof(.)="{yearID:float64,teamID:string,lgID:string,playerID:string,salary:float64}" or typeof(.)="{schoolID:string,name_full:string,city:string,state:string,country:string}" | fuse' bb.zng | jq | less
zq -f ndjson 'typeof(.)="{yearID:float64,teamID:string,lgID:string,playerID:string,salary:float64}" or typeof(.)="{schoolID:string,name_full:string,city:string,state:string,country:string}" | fuse | by typeof(.)' bb.zng


zq -o salaries.zng 'typeof(.)="{yearID:float64,teamID:string,lgID:string,playerID:string,salary:float64}" | max(salary) by playerID | sort playerID' bb.zng
