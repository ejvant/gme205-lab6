# GmE 205: Laboratory Exercise 6
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