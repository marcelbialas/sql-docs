---
title: "Query spatial data for nearest neighbor"
description: "Query spatial data for nearest neighbor, used to find the closest spatial objects to a specific spatial object."
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: mlandzic, jovanpop
ms.date: 11/04/2024
ms.service: sql
ms.topic: conceptual
ms.custom:
  - ignite-2024
monikerRange: "=azuresqldb-current || >=sql-server-2016 || >=sql-server-linux-2017 || =azuresqldb-mi-current || =fabric"
---
# Query spatial data for nearest neighbor

[!INCLUDE [SQL Server Azure SQL Database Azure SQL Managed Instance Fabric SQL endpoint Fabric DW FabricSQLDB](../../includes/applies-to-version/sql-asdb-asdbmi-fabricse-fabricdw-fabricsqldb.md)]  

  A common query used with spatial data is the nearest neighbor query. Nearest neighbor queries are used to find the closest spatial objects to a specific spatial object. For example, a store locator for a web site often must find the closest store locations to a customer location.  
  
 A nearest neighbor query can be written in various valid query formats, but for the nearest neighbor query to use a spatial index the following syntax must be used.  
  
## Syntax
  
```syntaxsql
SELECT TOP ( number )  
        [ WITH TIES ]  
        [ * | expression ]   
        [, ...]  
    FROM spatial_table_reference, ...   
        [ WITH   
            (   
                [ INDEX ( index_ref ) ]   
                [ , SPATIAL_WINDOW_MAX_CELLS = <value>]   
                [ ,... ]   
            )   
        ]  
    WHERE   
        column_ref.STDistance ( @spatial_ object )   
            {   
                [ IS NOT NULL ] | [ < const ] | [ > const ]   
                | [ <= const ] | [ >= const ] | [ <> const ] ]   
            }  
            [ AND { other_predicate } ]   
    }  
    ORDER BY column_ref.STDistance ( @spatial_ object ) [ ,...n ]  
[ ; ]  
```  
  
## Nearest neighbor query and spatial indexes

 In [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)], `TOP` and `ORDER BY` clauses are used to perform a nearest neighbor query on spatial data columns. The `ORDER BY` clause contains a call to the `STDistance()` method for the spatial column data type. The `TOP` clause indicates the number of objects to return for the query.  
  
 The following requirements must be met for a nearest neighbor query to use a spatial index:  
  
1. A spatial index must be present on one of the spatial columns and the `STDistance()` method must use that column in the `WHERE` and `ORDER BY` clauses.  
  
1. The `TOP` clause cannot contain a `PERCENT` statement.  
  
1. The `WHERE` clause must contain a `STDistance()` method.  
  
1. If there are multiple predicates in the `WHERE` clause then the predicate containing `STDistance()` method must be connected by an `AND` conjunction to the other predicates. The `STDistance()` method cannot be in an optional part of the `WHERE` clause.  
  
1. The first expression in the `ORDER BY` clause must use the `STDistance()` method.  
  
1. Sort order for the first `STDistance()` expression in the `ORDER BY` clause must be `ASC`.  
  
1. All the rows for which `STDistance` returns `NULL` must be filtered out.  
  
> [!WARNING]  
>  Methods that take **geography** or **geometry** data types as arguments will return `NULL` if the SRIDs are not the same for the types.  
  
 It is recommended that the new spatial index tessellations be used for indexes used in nearest neighbor queries. For more information on spatial index tessellations, see [Spatial Data](spatial-data-sql-server.md).  
  
## Example 1
 The following code example shows a nearest neighbor query that can use a spatial index. The example uses the `Person.Address` table in the [!INCLUDE [sssampledbobject-md](../../includes/sssampledbobject-md.md)] sample database.  
  
```sql  
USE AdventureWorks2022  
GO  
DECLARE @g geography = 'POINT(-121.626 47.8315)';  
SELECT TOP(7) SpatialLocation.ToString(), City FROM Person.Address
WHERE SpatialLocation.STDistance(@g) IS NOT NULL  
ORDER BY SpatialLocation.STDistance(@g);  
```  
  
 Create a spatial index on the column SpatialLocation to see how a nearest neighbor query uses a spatial index. For more information on creating spatial indexes, see [Create, Modify, and Drop Spatial Indexes](create-modify-and-drop-spatial-indexes.md).  
  
## Example 2
 The following code example shows a nearest neighbor query that cannot use a spatial index.  
  
```sql  
USE AdventureWorks2022  
GO  
DECLARE @g geography = 'POINT(-121.626 47.8315)';  
SELECT TOP(7) SpatialLocation.ToString(), City FROM Person.Address  
ORDER BY SpatialLocation.STDistance(@g);  
```  
  
 The query lacks a `WHERE` clause that uses `STDistance()` in a form specified in the syntax section so the query cannot use a spatial index.  
  
## Related content

- [Spatial Data](spatial-data-sql-server.md)
