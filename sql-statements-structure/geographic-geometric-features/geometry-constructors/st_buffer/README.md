# ST_BUFFER

## Syntax

```sql
ST_BUFFER(g1,r)
BUFFER(g1,r)
```

## Description

Returns a geometry that represents all points whose distance from geometry <em>`g1`</em> is less than or equal to distance, or radius, <em>`r`</em>.

Uses for this function could include creating for example a new geometry representing a buffer zone around an island.

BUFFER() is a synonym.

## Examples

Determining whether a point is within a buffer zone:

```sql
SET @g1 = ST_GEOMFROMTEXT('POLYGON((10 10, 10 20, 20 20, 20 10, 10 10))');

SET @g2 = ST_GEOMFROMTEXT('POINT(8 8)');

SELECT ST_WITHIN(@g2,ST_BUFFER(@g1,5));
+---------------------------------+
| ST_WITHIN(@g2,ST_BUFFER(@g1,5)) |
+---------------------------------+
|                               1 |
+---------------------------------+

SELECT ST_WITHIN(@g2,ST_BUFFER(@g1,1));
+---------------------------------+
| ST_WITHIN(@g2,ST_BUFFER(@g1,1)) |
+---------------------------------+
|                               0 |
+---------------------------------+
```