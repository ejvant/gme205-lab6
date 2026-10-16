# GmE 205: Laboratory Exercise 6
**Introductory Spatial Database Modeling**
## Overview
*This laboratory exercise focuses on converting an object model into a relational spatial database using PostgreSQL and PostGIS. It involves mapping object attributes to tables, defining primary and foreign keys, storing geometries with explicit SRIDs, and verifying data persistence through relational queries and spatial operations.*

## Prerequisites

- **PostgreSQL + PostGIS** – spatial database (versions listed in Database Preflight below)
- **pgAdmin 4** – runs the SQL scripts through the Query Tool
- **VS Code** – edits SQL and Markdown files
- **Git / GitHub** – version control and submission
- **diagrams.net** – relational model diagram

**How to run:** in the pgAdmin Query Tool on `gme205_lab6`, run `sql/01_schema.sql` → `sql/02_seed.sql` → `sql/03_queries.sql`.

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

## Persistence Decision Table

Persistence question: *If the Python program stops, which facts must still exist when the system starts again?*

| Object-model element | Persist? | Reason |
|---|---|---|
| `Parcel.id`, `land_use`, `geometry` | Yes | These identify and define the parcel. Without these, the parcel cannot be recognized again after the program restarts. |
| `Parcel.area()` | No | This is a method (behavior), not a stored fact. The area can always be recomputed from the stored geometry. |
| `Building.id`, `floors`, `geometry` | Yes | A building is a real feature with identity, height (in floors), and footprint which must remain available after the program ends. |
| Building → Parcel relationship | Yes | Knowing which parcel a building belongs to is a fact, not a calculation. The association between a building and its parcel must be preserved using a foreign key, such as `parcel_id`. |
| `Road.id`, `road_class`, `geometry` | Yes | The road's identity, classification, and line geometry are persistent spatial data. |
| `SpatialRule.evaluate(parcel)` | No | This is executable rule. It stays in the application code. The database stores the data used by the rule. |
| `Parcel.area_sqm` (Lab 5 attribute) | No | `area_sqm` was passed in and stored as an attribute. In the database it should not be a separate column, because area can always be derived. Storing it would duplicate information. |

## Verification Evidence

The four verification queries are in [`sql/03_queries.sql`](sql/03_queries.sql). Their full results, screenshots, and interpretations are documented in [`evidence/query_results.md`](evidence/query_results.md).

| Query | Result |
|---|---|
| 1 – `ST_Area` | P-001 = 10000 m² |
| 2 – JOIN | B-001 → P-001 |
| 3 – `ST_Within` | `true` |
| 4 – `ST_DWithin` | `true` |

Scripts should be run in this order in the pgAdmin Query Tool:
1. `sql/01_schema.sql`
2. `sql/02_seed.sql`
3. `sql/03_queries.sql`

## Consistency Review

| Question | Database evidence |
|---|---|
| What identifies one Parcel? | `parcels.parcel_id` PRIMARY KEY |
| What Parcel state survives? | `land_use` + `geom` |
| How does Building belong to Parcel? | `buildings.parcel_id` FOREIGN KEY |
| Where did `area()` go? | Stays as behavior; `ST_Area` derives it from `geom` |
| How is road proximity represented? | Not stored; derived with `ST_DWithin` |

## Reflection

1. `Parcel.land_use` was represented as the `land_use` column because it describes a parcel's recorded fact and cannot be derived from its geometry or other attributes.Therefore, it must still exist after the program stops.
2. `Parcel.area()` and `area_sqm` from my Lab 5 model were not stored as separate columns because the parcel's area can be calculated directly from its polygon geometry using ST_Area. Storing it would create a copy that could lead to inconsistencies if the geometry is changed.
3. The object model's relationship, where a building belongs to a parcel, was implemented through `buildings.parcel_id`. It is a `NOT NULL` foreign key referencing `parcels.parcel_id`. This preserves the association in the database and allows it to be retrieved using a JOIN.
4. A road being near a parcel is not ownership, and it depends on distance. `ST_DWithin` determines which roads are near a parcel by examining their stored geometries, especially when storing a potentially misleading `parcel_id` in the roads table.
5. Defining the geometry type and SRID in the database schema helps the database rejecting wrong data. This is for spatial data consistency. For example, the parcels table rejects LineString geometries, while geometries with a different coordinate reference system are not accepted. Using EPSG:32651 also supports consistent distance and area measurements in meters and square meters, respectively. 

## Author
Enoch Joshua V. Antonio  
MS Geomatics Engineering

## References

- PostgreSQL Documentation — Tables, primary keys, and foreign keys: https://www.postgresql.org/docs/
- PostGIS Documentation — Geometry types, ST_Area, ST_Within, and ST_DWithin: https://postgis.net/documentation/
- pgAdmin Documentation: https://www.pgadmin.org/docs/

Edited on GitHub web interface and VS Code