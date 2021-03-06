ExposurePathway
=================

Introduction
============
This R package creates an HTML visualzation of a exposure pathway based on a given sql query.

Features
========
- Uses SQL render to produce queries on the following platforms:
  - SQL Server
  - Oracle
  - Postgres
  - Amazon Redshift
  - MySQL
  - Impala
- Specify Max Path to limit number of steps in the path.
- Renders visualization using D3

Examples
========

```r
server = ExposurePathway::buildServer(name="Server Label", 
                                      dbms="{dialect}", 
                                      hostname="{hostname}", 
                                      port={port}, 
                                      databases = list(
                                        ExposurePathway::buildDatabase("DB Label", "{dbName}", "{schema}")
                                      ), 
                                      user="{username}",
                                      password="{password}!"
);

ExposurePathway::execute(servers=list(server), maxPathSize = 4, sequenceSql="{pathToSql}");
```
Parameters:
- servers: list of servers to execute
- maxPathSize: limits the number of exposures to show. Users with more than maxPathSize will show 'truncated' as the final node in the path.

sequenceSql is a path to a CDM query which will be executed to create the REQUIRED temp table: #EXPOSURE_SEQUENCE.  Example is provided in the package (to find first exposure to JNJ products):

```sql

with cteConcepts(concept_id) as
(
 select concept_id from @cdm_database_schema.CONCEPT where concept_id in (19047423,40239056,1125315,19059528,1103552,43012518,1336825,19039227,43526465,1192710,1594973,795113,1350310,19097481,797617,19054825,997881,42874220,21014127,35605744,1304643,1756831,19024728,1103006,19037833,1713905,1338512,739323,940864,1703069,1301125,715939,1548195,19049038,1549786,19050488,1758536,1154029,19055183,43534839,757627,19041065,766529,1126658,44507848,937368,1703653,19085688,985708,1389464,907553,1742253,1790692,991876,1107830,1794280,1503297,909841,705944,42900469,907879,19086100,1714319,735843,1338985,1521369,923081,19071314,918906,703244,926487,990340,43532451,43532122,924151,745790,19093225,21604277,19094980,21014157,911735,19016749,40238930,735979,40241331,19037983,44818461,44785086,950637,19026459,40239330,1710281,941472,742267,35603017,1103314,903643,21602139,40161532) and invalid_reason is null
),
cteExposures(PERSON_ID, DRUG_CONCEPT_ID, START_DATE, RN) as
(
  SELECT PERSON_ID, DRUG_CONCEPT_ID, DRUG_ERA_START_DATE as START_DATE, ROW_NUMBER() OVER (PARTITION BY PERSON_ID, DRUG_CONCEPT_ID ORDER BY DRUG_ERA_START_DATE) as RN
  FROM @cdm_database_schema.DRUG_ERA de
  JOIN cteConcepts c on de.DRUG_CONCEPT_ID = c.CONCEPT_ID
),
cteFirstExposures(PERSON_ID, DRUG_CONCEPT_ID, ORDINAL) as
(
  SELECT PERSON_ID, DRUG_CONCEPT_ID, ROW_NUMBER() OVER (PARTITION BY PERSON_ID ORDER BY START_DATE) as ORDINAL
  FROM cteExposures
  WHERE RN = 1
)
select PERSON_ID, DRUG_CONCEPT_ID, ORDINAL
INTO #EXPOSURE_SEQUENCE
from cteFirstExposures
;

```

The output of the package will be placed into an 'output' folder in the working directory.  The folder path will be:

```
OUTPUT
\---{Server Label}
    \---{Database 1}
    \---{Database 2}
    \---{Database 3}
    --- {Database 1}.HTML
    --- {Database 2}.HTML
    --- {Database 3}.HTML
    --- sequences.css
    --- sequences.js
    
```

To view the visualization, open the HTML file in Google Chrome.


Dependencies
============
DatabaseConncector
SqlRender

Getting Started
===============
Use the following commands in R to install the DatabaseConnector package:

```r
install.packages("devtools")
library(devtools)

install.packages("drat")
drat::addRepo("OHDSI")
install.packages("DatabaseConnector")
install.packages("SqlRender")

install_github("chrisknoll/ExposurePathway")

```


License
=======
ExposurePathway is licensed under Apache License 2.0. The JDBC drivers fall under their own respective licenses.

Development
===========
ExposurePathway is being developed in R Studio.


