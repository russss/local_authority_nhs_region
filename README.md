# English Local Authority District to NHS Region Mapping

This repo contains a *rough* mapping between English Local Authority Districts and NHS England Regions.
I've produced this to allow COVID-19 data from Public Health England (which is available broken down by local
authority) to be displayed alongside NHS data in comparable regions.

As NHS England region boundaries helpfully do not coincide directly with local authority boundaries,
LAs are assigned to the NHS region which covers the majority of their area. In most cases this is a
relatively good match, but in four cases (East Northamptonshire, High Peak, Copeland, and Craven),
more than 10% of a LA's area is in a different NHS region. The area is available in the `area_match`
column.

## Method

This is matched using PostGIS, with the "Local Authority Districts" and "NHS England Regions" shapefiles
from [ONS](https://geoportal.statistics.gov.uk/) imported as `local_authorities` and `nhs_regions`:

```sql
SELECT DISTINCT ON (lad19cd) lad19cd AS la_gss, lad19nm AS la_name, nhser20cd AS nhs_gss, nhser20nm AS nhs_name,
	round((ST_Area(ST_Intersection(nhs.geom, la.geom)) / ST_Area(la.geom) * 100)::numeric, 2) AS area_match
	FROM local_authorities la, nhs_regions nhs
	WHERE nhs.geom && la.geom
	AND ST_Area(ST_Intersection(nhs.geom, la.geom)) / ST_Area(la.geom) * 100 > 1
	ORDER BY lad19cd, ST_Area(ST_Intersection(nhs.geom, la.geom)) DESC;
```
