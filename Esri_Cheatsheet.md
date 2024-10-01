# openCypher Syntax Cheatsheet
Please also see our [ArcGIS openCypher Query Documentation](https://enterprise.arcgis.com/en/knowledge/latest/introduction/query-a-knowledge-graph.htm) for some additional explanations.This is not an exhaustive list of openCypher functionality implemented in ArcGIS.

* [Clauses](#Clauses)
* [Functions](#Functions)
* [Operators](#Operators)

## <a name='Clauses'></a>Read Query Clauses & Structure
These clauses and subclauses provide the baseline pattern for search operations. openCypher keywords are not case-sensitive. openCypher IS case-sensitive for variables.
You must have a MATCH and RETURN clause. You can have multiple MATCH statements in a query, but only one RETURN. Others clauses/subclauses are Optional. WITH acts like a RETURN statement in the middle of your query, only carrying forward the variables you specify into the next part of your query. 

MATCH [WHERE]
```
MATCH (e:ENTITIES)-[:ARE_RELATED_TO]->(oe:OTHERENTITIES)
WHERE e.NUMBERPROPERTY > 10 
```

WITH [ORDER BY] [LIMIT] [WHERE]
```
MATCH (e:ENTITIES)-[:ARE_RELATED_TO]->(oe:OTHER_ENTITIES)
WHERE e.NUMBERPROPERTY > 10 
WITH oe 
```

RETURN [ORDER BY] [SKIP] [LIMIT]
```
MATCH (e:ENTITIES)-[:ARE_RELATED_TO]->(oe:OTHER_ENTITIES)
WHERE e.NUMBERPROPERTY > 10 
WITH oe ORDER BY oe.PRICE DESC
WHERE oe.PRICE > 50
RETURN e.Name LIMIT 10
```

Additional Key Subclauses: [UNION] [UNION ALL] [UNWIND]

Note: The [USE] function in Cypher is not relevant in the ArcGIS implementation as all applications of clients require specification and authentication to the Knowledge Graph Service provided by the ArcGIS Knowledge Server, which brokers access to the underlying graph database. And there is a 1:1 relationship between the service and the database. Optional Match is not supported by OpenCypher

## <a name='Functions'></a>Functions

Type | Functions
------------ | -------------
Predicate | exists()
Scalar | **coalesce()**, endNode(), **head()**, **id(0)**, last(), length(), properties(), size(), startNode(), timestamp(), toBoolean(), toFloat(), toInteger(), type()
Aggregating | avg(), **collect()**, **count()**, max(), min(), percentileCont(), percentileDisc(), stDev(), stDevP(), sum()
List | keys(), labels(), nodes(), range(), relationships(), reverse(), tail()
Dates | **localdatetime()** *, date() *, localtime() *, **datetime()** *, duration.between(a,b)^, duration.inMonths(a,b)^, duration.inDays(a,b)^, duration.inSeconds(a,b)^
Math - numeric | abs(), ceil(), floor(), rand(), **round()**, sign()
Math - logarithmic | e(), exp(), log(), log10(), sqrt()
Math - trigonometric | acos(), asin(), atan(), atan2(), cos(), cot(), degree(), pi(), radians(), sin(), tan()
String | left(), lTrim(), replace(), reverse(), right(), rTrim(), split(), substring(), toLower(), toString(), toUpper(), trim()
[Spatial](#spatial) | esri.graph.ST_Equals(,), esri.graph.ST_Contains(,), esri.graph.ST_Intersects(,), esri.graph.ST_GeoDistance(,), esri.graph.ST_WKTToGeometry(string)

* Only available in ArcGIS Enterprise 11.2+

^ Only available in ArcGIS Enterprise 11.3+

### Common Functions
Function | Syntax | Description
------ | ----------- | ---------------
coalesce() | coalesce(expression [, expression]*) | Returns the first non-null value in a list of expressions. E.g. Return either p.Name or p.NameAlias if some p.Names are null
head() | head(expression reterning a list) | The function head() returns the first element in a list, if the expression returns a list.
id() | id(expression returning node or relationship) | The function id() returns a node or a relationship identifier, unique by database, as an integer value.
collect() | collect(input) | Returns a single list containing all the values returned by an expression. Nulls ignored.
count() | count(input) | Returns the number of values or rows as integer value.

## <a name='Operators'></a>Operators

Type | Operators
------------ | -------------
General | DISTINCT, Entity.y ("y" property access)
Math | +, -, *, /, %, ^
Comparison | =, >, <, <>, <=, >=, IS NULL, IS NOT NULL
Boolean | AND, OR, NOT, XOR
String | STARTS WITH, ENDS WITH, CONTAINS, +
LIST | +, IN, []
Sorting | ORDER BY xxx [ASC/DESC] 

### Working with DateTime Functions
You must use ArcGIS datetime field types.

Field type | function | Syntax | Example
------------  | -------------- | --------------------- | -------------------------
Date | localdatetime() | localdatetime('YYYY-MM-DDThh:mm:ss.sss') | localdatetime('2015-07-24T21:40:53.142')
Date only | date() | date('YYYY-MM-DD') | date('2015-07-24')
Time only | localtime() | localtime('hh:mm:ss.sss') | localtime('21:40:53.142')
Timestamp offset | datetime() | datetime('YYYY-MM-DDThh:mm:ss.sssZ') or datetime('YYYY-MM-DDThh:mm:ss.sss+00:00') | datetime('2015-07-24T21:40:53.142Z') or datetime('2015-07-21T21:40:53.142-08:00')

**Common DateTime Patterns**

* Create a date from a typed-in-string, or convert a datetime value on an existing property. These operations are often done to filter matched records by a relative date (e.g. find employees that started working at the company before 2024-01-01). Comparison dates should be converted to the same datetime property type.
```
MATCH path=(:Person)-[ea:employeedAt]->(:Employer {EmployerName: "Company XYZ"})
WHERE ea.hireDate < datetime('2024-01-01T12:00:00.000Z)')
RETURN path
``` 
* Duration - Compare two dates to return a number value between them (e.g. how many days were between these two events). Filter matched records by a duration value (e.g. find employees that has worked for more than 60 days). Use the ISO 8601 time duration format and ADD/SUBTRACT it from another datetime field. For example, a duration of P1Y2M3DT4H5M6S represents a time interval of one year, two months, three days, four hours, five minutes, and six seconds.

Example: To find people who sold a vehicle that they owned for less than two years, use a query such as
```
MATCH path=(:Person)-[hv:HasVehicle]->(:Vehicle) WHERE hv.endDate < (hv.acquisitionDate + duration('P2Y')) RETURN path, duration.between(hv.acquisitionDate,hv.endDate) as TimeOwned
```
All paths are returned where the HasVehicle relationship has an endDate property and the duration between the endDate and the acquisitionDate property is less than two years. The query returns both the path and the duration of time that the vehicle was owned.


### <a name='spatial'></a>Working with Spatial Functions
These functions would be used in a WHERE clause
Function | Syntax | Description | Example 
------ | --------- | --------------- | ----------------------------
ST_Equals | esri.graph.ST_Equals (geometry1, geometry2) | Returns entities with equal geometries. | _MATCH (p1:Person), (p2:Person) WHERE esri.graph.ST_Equals(p1.shape, p2.shape) RETURN p1, p2_  - Returns each  different pair of Person entities that have identical location geometries.
ST_Intersects | esri.graph.ST_Intersects (geometry1, geometry2) | Returns entities with intersecting geometries. | _MATCH (e:Employee), (f:Facility) WHERE esri.graph.ST_Intersects(e.shape, f.shape) RETURN e, f_ - Returns Employee entities and Facility entities, e and f, respectively, where geometries for the Employee and Facility entities intersect.
ST_Contains | esri.graph.ST_Contains (geometry1, geometry2) | Returns entities whose geometries are contained by the first specified geometry. | _MATCH (f:Facility) WHERE esri.graph.ST_Contains(esri.graph.ST_WKTToGeometry ("POINT (-117.1964763 34.0572046)"), f.shape) RETURN f_  - Returns Facility entities, f, whose geometries contain the specified point.
ST_GeoDistance | esri.graph.ST_GeoDistance (geometry1, geometry2) | Returns the distance between the two geometries | _MATCH (n), (e) RETURN esri.graph.ST_GeoDistance(n.shape, e.shape) as distance_ - Returns each node in the graph and their the distance variable in the WHERE clause stores the geodesic distance that is calculated between the entities n and e.

Note that Geodesic distance is returned in metric units of meters. Other units of meature must be calculated. For example, kilometers and miles would be, respectively
```
esri.graph.ST_GeoDistance (geometry1, geometry2)/1000 AS DistanceInKM
```
```
esri.graph.ST_GeoDistance (geometry1, geometry2)*0.000621371 AS DistanceInMiles
```

**Constructing a geometry out of Well Known Text**
You can also create a geometry that represents a spatial location from a string using the operator esri.graph.ST_WKTToGeometry(string). The string parameter must be an OGC simple feature specified in the well-known text format. For example, to create a geometry representing the coordinates 117.1964763°W 34.0572046°N, you would use the operator esri.graph.ST_WKTToGeometry("POINT (-117.1964763 34.0572046)"). 

## Common Patterns
### Using Variables & Aliases 
You can use varibles to represent Entity Types, Relationship Types, paths or lists of results from an expression.
Variables can be reused later in later clauses, operators or subqueries
Alias can be defined by putting "AS" after operators.
```
MATCH (**person**:Person)-[:hasPet]->(dog:Pet {type:"dog"})
WITH distinct **person** AS **UniqueDogOwner**
RETURN UniqueDogOwner
```
### Create Distinct Lists 
Use DISTINCT(expression) | Returns only unique instances or nodes within the expression
```
MATCH (n:Person)-[r:KNOWS]-(m:Person)
RETURN DISTINCT n AS UniquePersonList
```
### Collect & Unwind lists
COLLECT(expression) | Returns a single aggregated list containing the values returned by an expression. 
UNWIND(expression) | 
```
// Find enemy units with a lot of reconnaissance equipment, and return the list of equipment, grouped by unit

MATCH (u:Unit)-[:HasEquipment]->(e:Equipment)
WITH u, COLLECT(e) AS equipmentList
RETURN u, equipmentList
```
Result will should return:

UnitA, [equipment1, equipment2, equipment3...]
UnitB, [equipment2, equipment18]
...
# openCypher Query Examples
The below are some common examples of openCypher Syntax based upon the [iNaturalist Bee Observation dataset](https://developers.arcgis.com/javascript/latest/sample-code/sandbox/?sample=knowledgegraph-query) used in our JS Sample Code Sandbox environment. 

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

<!-- vscode-markdown-toc-config
	numbering=false
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

## <a name='PatternMatching'></a>Pattern Matching
Pattern matching is the most basic actions one can do in openCypher and is the basis for all read queries.

### <a name='FindNodesbyLabelandProperty'></a>Find Entities by Type/Label and Specific Property
```
MATCH (a:airport {code: 'SEA'}) RETURN a
```
```
MATCH (a:airport) WHERE a.code='SEA' RETURN a
```

### <a name='Findpattern'></a>Find pattern
```
MATCH (a:airport {code: 'SEA'})-[:route]->(d) RETURN d
```
```
MATCH path=(a:airport {code: 'SEA'})-[:route]->(d) RETURN path
```
### <a name='Findifsomethingexists'></a>Find if something exists
```
MATCH (a:airport {code: 'SEA'})-[:route]->(d) RETURN d
```

## <a name='Looping'></a>Looping
Looping through connections is a common pattern in property graphs and can be accomplished using either fixed or variable length paths.

### <a name='Fixednumberofloops'></a>Fixed number of loops
```
MATCH (a:airport {code: 'SEA'}*2)-[:route]->(d) RETURN d
```
### <a name='Rangeofhops'></a>Range of hops
```
MATCH (a:airport {code: 'SEA'})-[:route*1..3]->(d) RETURN d
```

## <a name='ReturningValues'></a>Returning Values
Each read query must specify how to return values from the query.

### <a name='Returneverything'></a>Return everything
```
MATCH (a) RETURN *
```

### <a name='Returncolumn'></a>Return property
```
MATCH (a:airport) RETURN a.city
```
### <a name='Returncolumnwithalias'></a>Return property with alias
```
MATCH (a:airport) RETURN a.city AS dest
```
### <a name='Returndistinctvalues'></a>Return distinct values
```
MATCH (a:airport) RETURN DISTINCT a.region
```
### <a name='Orderresultsdescending'></a>Order results descending
```
MATCH (a:airport) RETURN a ORDER BY a.elev DESC
```

### <a name='Returncountofresults'></a>Return count of results
```
MATCH (a:airport) RETURN count(a)
```

### <a name='Limitresults'></a>Limit results
```
MATCH (a:airport) RETURN a LIMIT 5
```

### <a name='Joindistinctsetfromtwoqueries'></a>Join distinct set from two queries
```
MATCH (a:airport {code:'SEA'}) 
RETURN a.city
UNION 
MATCH (a:airport {code:'ANC'}) 
RETURN a.elev
```
### <a name='Joincombinedsetfromtwoqueries'></a>Join combined set from two queries
```
MATCH (a:airport {code:'SEA'})-->(d) 
RETURN d.city
UNION ALL
MATCH (a:airport {code:'ANC'})-->(d) 
RETURN d.city
```
