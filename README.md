# Metropolitan Homelessness
The goal of this project is to develop estimates of homeless populations within Metropolitan Statistical Areas (MSAs) using datasets provided by the United States Census Bureau and by the Department of Housing and Urban Development (HUD).

Data related to homelessness comes from HUD’s annual point-in-time count and is collected within geographic boundaries called Continuums of Care (CoC). These boundaries are not defined in terms of standard geographies used by the Census Bureau. While a CoC often represents one or more counties, this isn’t necessarily the case and there are many exceptions. For example:

* Most CoCs represent a single county, but in some cases these boundaries aren’t 100% coincident. For example, Dupage County CoC (IL-514) in Illinois is roughly the same size, shape, and area as the county itself, but the boundary jogs in some areas, grabbing parts of neighboring counties in some locations and omitting parts of Dupage County in other locations.

* Some CoCs represent multiple counties. For example, Central Oregon CoC (OR-503) represents Crook, Deschutes, and Jefferson Counties. Wyoming Statewide CoC (WY-500) represents all 23 counties in the state.

* Some CoCs represent a small part of the county they are within. For example, Cambridge CoC (MA-509) and Somerville CoC (MA-517) are small CoCs located entirely within Middlesex County, Massachussettes.

* Some CoCs are more complicated in their relationships to counties. For example, Oklahoma City CoC (OK-502) spans across Cleveland, Canadian, Oklahoma, and Pottawatomie Counties but does not entirely subsume them.

## Apportioning Homeless Populations from CoCs to Counties
In order to derive homeless population estimates for Metropolitan Statistical Areas, homeless population estimates must first be established for the counties that make up each MSA. In order to remap the homeless populations from CoCs to counties, the CoC region is broken up into small geographic areas and the re-aggregated into counties. The way this works in practice is that counties boundaries and CoC boundaries are “intersected” in GIS software to create smaller geographic areas that belong to only one county and only one CoC. The process is as follows:

1. The counties are split along CoC boundaries and the general population is apportioned to the resulting sub-areas in proportion to their percentage of the county’s geographic area. For example, Cleveland County in Oklahoma falls within the boundaries of two different CoCs, Oklahoma City CoC (OK-502) and Norman/Cleveland County CoC (OK-504). Because 22.6% of Cleveland County’s area falls within Oklahoma City CoC, 22.6% of the county’s general population (61,978 of 274,024) is assumed to be located in the newly generated sub-area.

2. The overall population of a CoC is calculated by aggregating the general populations of each of the sub-areas that were derived in the previous step. For example, Oklahoma County CoC has a total general population of 469,201. This includes the partial population of the Cleveland County sub-area that was created above (61,978) as well as 50.0% of Oklahoma County’s population (387,346), 14.9% of Canadian County’s population (19,833), and a tiny part of Pottawatomie County’s population (44).

3. The homeless population of the CoC is then apportioned to the sub-areas in proportion to their percentage of the CoC’s overall population. For example, the sub-area of Cleveland County that falls within Oklahoma City CoC represents 13.2% of the CoC’s general population (61,978 of 469,201) and is therefore assumed to represent the same percentage of the CoC’s homeless population (156 of 1,183).

4. Finally, the homeless population of each county is calculated by aggregating the homeless populations of each sub-area that falls within the county’s boundaries. So, for example, Cleveland County has a total homeless population of 520. This includes the 156 people from Oklahoma City CoC as well as the 364 people from Norman/Cleveland County CoC.

Using the process above, we can generate homeless estimates for each county, and those estimates can be aggregated to create total counts for each of the 390 MSAs.

## A Note on Methodology
The above method for apportioning population is by no means perfect. One big source of error is the assumption that the general population of each county is evenly distributed throughout that county’s area. We know that this is not the case in the real world and that urban, suburban, and rural portions of counties can vary greatly in terms of population density.

To see how this affects how homeless populations are apportioned, let’s look at Albuquerque, New Mexico. Albuquerque CoC is a fraction of the size of Bernalillo County, representing only 16.5% of the county’s area. Our methodology assumes that the population of the county is uniformly distributed, and therefore 16.5% of Bernalillo County’s population (111,563 of 674,855) is apportioned to the sub-area that falls within the CoC.

As it turns out, the boundary of Albuquerque CoC corresponds to the City of Albuquerque, and we know that the population of that area in 2017 was closer to 558,545 -- five times larger than our methodology assumes.

The effect of this inaccurate apportioning of the general population is that homelessness within Bernalillo County is likely over represented while homelessness in all other counties in New Mexico (which all fall within the New Mexico Balance of State CoC) is likely under represented. 

Because homeless populations are apportioned based on the general population of the underlying sub-areas, Bernalillo County not only receives 100% of Albuquerque CoC’s homeless population, it also gets a larger proportion of the Balance of State CoC’s homeless population. This is because the sub-area of the County that is outside of Albuquerque CoC has 446,982 more people than it should.

This is a somewhat unique condition -- there are very few CoCs that are smaller than Counties -- but it exposes a weakness in the methodology for apportioning populations. A more accurate way to apportion population would be to use boundaries that are representative of changes in population density. One such boundary would be “Urban Areas” as defined by the Census Bureau. However, while the shapefiles for these geographies are available online, the corresponding population counts from the American Community Survey are not, likely because the ACS does not collect the same information as the decennial census.

## Datasets Used
The following publicly available data was used to derive homeless population counts for each MSA.

[US Census County Shapefile](https://www.census.gov/cgi-bin/geo/shapefiles/index.php) - This was used to obtain county boundaries. Because coastlines are not represented in the shapefile, the geometry was intersected with the CoC shapefile to generate more accurate land areas (although interior bodies of water such as lakes are not accounted for). Counties are also merged to create MSAs.

[HUD CoC Shapefile](https://www.hudexchange.info/programs/coc/gis-tools/) - This was used to obtain CoC Boundaries. Because the CoC shapefile and the county shapefile come from two different sources, boundaries can differ slightly even when the two shapefiles intend to represent the same geographic area (such as a county). To avoid sliver areas resulting from an intersection of the two shapefiles, the CoC boundaries were rebuilt using the county shapefile. Where CoC and county boundaries are similar, the county boundary is used. This will lead to some unknown margin of error, but it is assumed to be negligible.

[US Census 2017 American Community Survey](https://data.census.gov/) - This was used to obtain general population statistics for each county. Ideally the 2018 ACS would be used in order to match the survey year of the HUD data, but the online data that was available for that year at the time of publishing this was incomplete.

[HUD 2018 Point in Time Count](https://www.hudexchange.info/resource/3031/pit-and-hic-data-since-2007/) - This was used to obtain homeless population statistics for each CoC.

## Files Generated
**Continuum of Care.gpkg** - This shapefile is a rebuilt version of the HUD shapefile with boundaries that more closely follow those of underlying counties.

**County.gpkg** - This shapefile is a rebuilt version of the Census Bureau shapefile with coastlines based on the HUD shapefile. Each feature includes fields related to general population and homeless population.

**Metropolitan Statistical Area.gpkg** - This shapefile merges together county boundaries and statistics.

**MSA Homeless Counts.csv** - This tabulation estimates a total homeless population for each Metropolitan Statistical Area.

**MSA Homeless Representation.csv** - This tabulation uses the estimates above to generate a “representation ratio” for each MSA. This is calculated by dividing the MSA’s share of the national homeless population by the MSA’s share of the national general population. A value of 1 indicates that the rate of homelessness is equal to the national rate of homelessness. A value of 2 indicates that the rate of homelessness is double the national rate of homelessness, ie, twice as many homeless people per capita.
