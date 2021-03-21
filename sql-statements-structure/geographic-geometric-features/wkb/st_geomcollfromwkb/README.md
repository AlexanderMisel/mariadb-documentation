# ST_GeomCollFromWKB

## Syntax

```sql
ST_GeomCollFromWKB(wkb[,srid])
ST_GeometryCollectionFromWKB(wkb[,srid])
GeomCollFromWKB(wkb[,srid])
GeometryCollectionFromWKB(wkb[,srid])
```

## Description

Constructs a GEOMETRYCOLLECTION value using its [WKB](/sql-statements-structure/geographic-geometric-features/wkb/well-known-binary-wkb-format/)  representation and SRID.

`ST_GeomCollFromWKB()`, `ST_GeometryCollectionFromWKB()`, `GeomCollFromWKB()` and `GeometryCollectionFromWKB()` are synonyms.

## Examples

```sql
SET @g = ST_AsBinary(ST_GeomFromText('GEOMETRYCOLLECTION(POLYGON((5 5,10 5,10 10,5 5)),POINT(10 10))'));

SELECT ST_AsText(ST_GeomCollFromWKB(@g));
+----------------------------------------------------------------+
| ST_AsText(ST_GeomCollFromWKB(@g))                              |
+----------------------------------------------------------------+
| GEOMETRYCOLLECTION(POLYGON((5 5,10 5,10 10,5 5)),POINT(10 10)) |
+----------------------------------------------------------------+
```