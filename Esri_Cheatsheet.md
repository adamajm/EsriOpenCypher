# openCypher Syntax Cheatsheet
The below are some common examples of openCypher Syntax based upon the [iNaturalist Bee Observation dataset](https://developers.arcgis.com/javascript/latest/sample-code/sandbox/?sample=knowledgegraph-query) used in our JS Sample Code Sandbox environment.

## Read Query Structure
These clauses and subclauses provide the baseline pattern for search operations. OpenCypher keywords are not case-sensitive. Cypher IS case-sensitive for variables.

MATCH [WHERE]
```
MATCH (e:ENTITIES)-[:ARE_RELATED_TO]->(oe:OTHERENTITIES)
WHERE e.NUMBERPROPERTY > 10 
```

WITH [ORDER BY] [SKIP] [LIMIT] [WHERE]
```
MATCH (e:ENTITIES)-[:ARE_RELATED_TO]->(oe:OTHER_ENTITIES)
WHERE e.NUMBERPROPERTY > 10 
WITH oe ORDER BY oe.PRICE
```

RETURN [ORDER BY] [SKIP] [LIMIT]
```
MATCH (e:ENTITIES)-[:ARE_RELATED_TO]->(oe:OTHER_ENTITIES)
WHERE e.NUMBERPROPERTY > 10 
WITH oe ORDER BY oe.PRICE DESC
```

Extras: [UNION] [UNION ALL] [UNWIND]

Note: The [USE] function in Cypher is not relevant in the ArcGIS implementation as all applications of clients require specification and authentication to the Knowledge Graph Service provided by the ArcGIS Knowledge Server, which brokers access to the underlying graph database. And there is a 1:1 relationship between the service and the database. Optional Match is not supported by OpenCypher

### Common Patterns
#### Using Variables & Aliases 
You can use varibles to represent  Entity Types, Relationship Types, or collections of results.
Variables can be reused later in later clauses, operators or subqueries
Alias can be defined by putting "AS" after operators.
```
WITH distinct(person) AS UniquePeople
RETURN UniquePeople
```
#### Create Distinct Lists 
Use Distinct(expression)
```
MATCH (n:Person)-[r:KNOWS]-(m:Person)
RETURN DISTINCT n AS UniquePersonList
```
#### Collect & Unwind lists
collect(expression) | The function collect() returns a single aggregated list containing the values returned by an expression. 
unwind

## Example Queries - Reference Data Model
![image](https://github.com/user-attachments/assets/3953062e-2310-4379-a19a-bac9a23a81e2)

## <a name='TableofContents'></a>Table of Contents
<!-- vscode-markdown-toc -->
* [Read Queries](#ReadQueries)
	* [Pattern Matching](#PatternMatching)
		* [Find Entities by Type and Property](#FindNodesbyLabelandProperty)
		* [Find pattern](#Findpattern)
	* [Looping](#Looping)
		* [Fixed number of loops](#Fixednumberofloops)
		* [Range of hops](#Rangeofhops)
	* [Returning Values](#ReturningValues)
		* [Return everything](#Returneverything)
		* [Return property](#Returncolumn)
		* [Return property with alias](#Returncolumnwithalias)
		* [Return distinct values](#Returndistinctvalues)
		* [Order results ascending](#Orderresultsascending)
		* [Return count of results](#Returncountofresults)
		* [Limit results](#Limitresults)
		* [Join distinct set from two queries](#Joindistinctsetfromtwoqueries)
		* [Join combined set from two queries](#Joincombinedsetfromtwoqueries)
* [Operators](#Operators)
* [Functions](#Functions)

<!-- vscode-markdown-toc-config
	numbering=false
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

## <a name='ReadQueries'></a>Read Queries

### <a name='PatternMatching'></a>Pattern Matching
Pattern matching is the most basic actions one can do in openCypher and is the basis for all read queries.

#### <a name='FindNodesbyLabelandProperty'></a>Find Entities by Type/Label and Specific Property
```
MATCH (a:airport {code: 'SEA'}) RETURN a
```
```
MATCH (a:airport) WHERE a.code='SEA' RETURN a
```

#### <a name='Findpattern'></a>Find pattern
```
MATCH (a:airport {code: 'SEA'})-[:route]->(d) RETURN d
```
```
MATCH path=(a:airport {code: 'SEA'})-[:route]->(d) RETURN path
```
#### <a name='Findifsomethingexists'></a>Find if something exists
```
MATCH (a:airport {code: 'SEA'})-[:route]->(d) RETURN d
```

### <a name='Looping'></a>Looping
Looping through connections is a common pattern in property graphs and can be accomplished using either fixed or variable length paths.

#### <a name='Fixednumberofloops'></a>Fixed number of loops
```
MATCH (a:airport {code: 'SEA'}*2)-[:route]->(d) RETURN d
```
#### <a name='Rangeofhops'></a>Range of hops
```
MATCH (a:airport {code: 'SEA'})-[:route*1..3]->(d) RETURN d
```

### <a name='ReturningValues'></a>Returning Values
Each read query must specify how to return values from the query.

#### <a name='Returneverything'></a>Return everything
```
MATCH (a) RETURN *
```

#### <a name='Returncolumn'></a>Return property
```
MATCH (a:airport) RETURN a.city
```
#### <a name='Returncolumnwithalias'></a>Return property with alias
```
MATCH (a:airport) RETURN a.city AS dest
```
#### <a name='Returndistinctvalues'></a>Return distinct values
```
MATCH (a:airport) RETURN DISTINCT a.region
```
#### <a name='Orderresultsdescending'></a>Order results descending
```
MATCH (a:airport) RETURN a ORDER BY a.elev DESC
```

#### <a name='Returncountofresults'></a>Return count of results
```
MATCH (a:airport) RETURN count(a)
```

#### <a name='Limitresults'></a>Limit results
```
MATCH (a:airport) RETURN a LIMIT 5
```

#### <a name='Joindistinctsetfromtwoqueries'></a>Join distinct set from two queries
```
MATCH (a:airport {code:'SEA'}) 
RETURN a.city
UNION 
MATCH (a:airport {code:'ANC'}) 
RETURN a.elev
```
#### <a name='Joincombinedsetfromtwoqueries'></a>Join combined set from two queries
```
MATCH (a:airport {code:'SEA'})-->(d) 
RETURN d.city
UNION ALL
MATCH (a:airport {code:'ANC'})-->(d) 
RETURN d.city
```

## <a name='Operators'></a>Operators

Type | Operators
------------ | -------------
General | DISTINCT, x.y (property access)
Math | +, -, *, /, %, ^
Comparison | =, >, <, <>, <=, >=, IS NULL, IS NOT NULL
Boolean | AND, OR, NOT, XOR
String | STARTS WITH, ENDS WITH, CONTAINS, +
LIST | +, IN, []
Sorting | ORDER BY xxx ASC/DESC
Spatial | esri.graph.ST_Equals(,), esri.graph.ST_Contains(,), esri.graph.ST_Intersects(,), esri.graph.ST_GeoDistance(,), esri.graph.ST_WKTToGeometry(string)

## Working with DateTime

## <a name='Functions'></a>Functions

Type | Functions
------------ | -------------
Predicate | exists()
Scalar | **coalesce()**, endNode(), head(), id(0), last(), length(), properties(), size(), startNode(), timestamp(), toBoolean(), toFloat(), toInteger(), type()
Aggregating | avg(), **collect()**, **count()**, max(), min(), percentileCont(), percentileDisc(), stDev(), stDevP(), sum()
List | keys(), labels(), nodes(), range(), relationships(), reverse(), tail()
Dates | **localdatetime()**, date(), localtime(), **datetime()**, duration.between(a,b), duration.inMonths(a,b), duration.inDays(a,b), duration.inSeconds(a,b)
Math - numeric | abs(), ceil(), floor(), rand(), round(), sign()
Math - logarithmic | e(), exp(), log(), log10(), sqrt()
Math - trigonometric | acos(), asin(), atan(), atan2(), cos(), cot(), degree(), pi(), radians(), sin(), tan()
String | left(), lTrim(), replace(), reverse(), right(), rTrim(), split(), substring(), toLower(), toString(), toUpper(), trim()
