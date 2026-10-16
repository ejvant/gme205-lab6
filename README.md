# GmE 205: Laboratory Exercise 5
**Introductory Spatial Database Modeling**
## Overview
*This laboratory exercise focuses on converting an object model into a relational spatial database using PostgreSQL and PostGIS. It involves mapping object attributes to tables, defining primary and foreign keys, storing geometries with explicit SRIDs, and verifying data persistence through relational queries and spatial operations.*

## Database Preflight

| Item | Value |
|---|---|
| Database name | `gme205_lab6` |
| PostgreSQL version | PostgreSQL 18.6 on x86_64-windows (64-bit) |
| PostGIS version | POSTGIS="3.6.2" (GEOS 3.14.1, PROJ 8.2.1) |

PostGIS was enabled in `gme205_lab6` with `CREATE EXTENSION IF NOT EXISTS postgis;`.

Preflight queries used:

```sql
SELECT current_database();
SELECT version();
SELECT postgis_full_version();
```