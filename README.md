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
unzip baseballdatabank-master.zip
find baseballdatabank-master -name \*.csv -exec echo "cat" {} "| cj" \; | sh | zq -i ndjson -o bb.zng -
```
This gives you a file called `bb.zng`, which holds all of the tables from
the Lahamn data set as a stream of heterogenous records in accordance with
the Z data model.  The file should be 25874793 bytes.  You can also confirm
the number of rows that we collected:
```
zq -f table "count()" bb.zng
```
which should output
```
COUNT
586970
```

> NOTE: once the ZSON parser is merged we can have cj emit ZSON and have
> some simple heuristics to recognize date strings and turn them into type time.

## Examples

The nice thing about zq is it's kind of like search but it's also OLAP-y so
you can do analytics with it too.

Let's search for Joe Dimaggio...
```
zq -f ndjson dimaggio bb.zng
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

Hmm, turns out there were two other Dimmagios.

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
zq -f ndjson "dimagvi01 | r=first(.) by typeof(.)" bb.zng | jq
```
Note that `.` refers the entire row of each row in the query.
This tables are very wide so I used JSON output with jq and we get this:
```
{
  "r": {
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
  }
}
{
  "r": {
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
}
{
  "r": {
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
  }
}
{
  "r": {
    "awardID": "MVP",
    "lgID": "NL",
    "playerID": "dimagvi01",
    "pointsMax": 336,
    "pointsWon": 10,
    "votesFirst": 0,
    "yearID": 1941
  }
}
{
  "r": {
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
  }
}
{
  "r": {
    "Gcf": 129,
    "Glf": 0,
    "Grf": 1,
    "playerID": "dimagvi01",
    "stint": 1,
    "yearID": 1937
  }
}
{
  "r": {
    "GP": 1,
    "gameID": "ALS194307130",
    "gameNum": 0,
    "lgID": "NL",
    "playerID": "dimagvi01",
    "startingPos": null,
    "teamID": "PIT",
    "yearID": 1943
  }
}
{
  "r": {
    "awardID": "Baseball Magazine All-Star",
    "lgID": "NL",
    "notes": "CF",
    "playerID": "dimagvi01",
    "tie": null,
    "yearID": 1943
  }
}
```
> TBD: TAKE OUT 'r' above

What's cool here is you don't have to look at any models or schemas.  Just
throw all the data into a zng file and run type-oriented queries to
_introspectively_ discover the shape of the data.

Ok, you're wondering did we find all of the schemas here by looking
at this one player ID?  Maybe there were other tables in the data
set we don't know about.  Let's check...

If we count the schemas from our search...
```
zq -f table "dimagvi01 | first(.) by typeof(.) | count()" bb.zng
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
131
```
What a minute, there weren't that many csv files.
```
find baseballdatabank-master | grep \.csv | wc
```
says there are 28.  How did we get 131?!

Well, this is a shortcoming of CSV and JSON.  When the data comes in to zq
row by row, sometimes there are fields missing (i.e., JSON nulls) then
`zq` doesn't know if it's a number or a string so it assigns the type each
null field the Z type "null".  This creates many variations of types
where different null values appear in different fiedls.

For example, let's just take the joint presence of types of a few stats,
say PB, RP, and SB:
```
zq -f table "by pb=typeof(PB),wp=typeof(WP),sb=typeof(SB)" bb.zng
===
PB      WP      SB
string  string  string
float64 string  float64
float64 string  string
float64 float64 float64
```
> What?!  Turns out zq ndjson parser is assigning string(null) to null
> fields instead of null(null).

Of course, with a comprehensive look over all the
data, zq could be used to infer accurate types of everything but it requires
more work, and the beauty if this approach is that zq is forgiving and
adaptive.  It will work just fine with the 131 types.
```
zq "* | ( filter PB=null | put PB=null ; filter PB != null ) | filter *" bb.zng > bb2.zng
zq -f table "by pb=typeof(PB),wp=typeof(WP),sb=typeof(SB)" bb2.zng
===
PB      WP      SB
null    string  string
float64 string  string
float64 float64 float64
float64 string  float64
```

How about we just select a few fields and show them
as a table...
```
zq -f table "dimaggio | cut nameGiven,nameLast,weight,year" namebb.zng
```

### back to the join key

Ok, we had guessed bbrefID was a join key, but what are the other columns
where we might want to join to?

> We could look through things by hand but it would be nice to have a columnsOf(value)
> function that would give field names that matched the given value as an type ([string])

In the meantime, we can look at all the types of that have `dimagvi01` somewhere
in the row and eyeball... we can get a little help from grep...

zq -f ndjson "dimagvi01 | first(.) by typeof(.)" bb.zng | jq | grep dimagvi01
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
```
COUNT
20091
```
20k sounds about right.

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
