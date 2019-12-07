# Metropolitan Homelessness
The goal of this project is to develop estimates of homeless populations within Metropolitan Statistical Areas (MSAs) using datasets provided by the United States Census Bureau and by the Department of Housing and Urban Development (HUD).

Data related to homelessness comes from HUD’s annual point-in-time count and is collected within geographic boundaries called Continuums of Care (CoC). These boundaries are not defined in terms of standard geographies used by the Census Bureau. While a CoC often represents one or more counties, this isn’t necessarily the case and there are many exceptions. For example:

* Most CoCs represent a single county, but in some cases these boundaries aren’t 100% coincident. For example, Dupage County CoC (IL-514) in Illinois is roughly the same size, shape, and area as the county itself, but the boundary jogs in some areas, grabbing parts of neighboring counties in some locations and omitting parts of Dupage County in other locations.

* Some CoCs represent multiple counties. For example, Central Oregon CoC (OR-503) represents Crook, Deschutes, and Jefferson Counties. Wyoming Statewide CoC (WY-500) represents all 23 counties in the state.

* Some CoCs represent a small part of the county they are within. For example, Cambridge CoC (MA-509) and Somerville CoC (MA-517) are small CoCs located entirely within Middlesex County, Massachussettes.

* Some CoCs are more complicated in their relationships to counties. For example, Oklahoma City CoC (OK-502) spans across Cleveland, Canadian, Oklahoma, and Pottawatomie Counties but does not entirely subsume them.

## Apportioning Homeless Populations from CoCs to Counties
In order to derive homeless population estimates for Metropolitan Statistical Areas, those estimates must first be established for the counties that make up each MSA. In order to remap the homeless populations from CoCs to counties, the CoC region is broken up into small geographic sub-areas and the re-aggregated into counties. The way this works in practice is that county boundaries and CoC boundaries are “intersected” in GIS software to create smaller geographic areas that belong to only one county and only one CoC. The process is as follows:

1. The counties are split along CoC boundaries and the general population of each county is apportioned to the resulting sub-areas in proportion to their percentage of the county’s geographic area. For example, Cleveland County in Oklahoma falls within the boundaries of two different CoCs: Oklahoma City CoC (OK-502) and Norman/Cleveland County CoC (OK-504). Because 22.6% of Cleveland County’s area falls within Oklahoma City CoC, 22.6% of the county’s general population (61,978 of 274,024) is assumed to be located in the newly generated sub-area.

2. The overall population of a CoC is calculated by aggregating the general populations of each of the sub-areas that were derived in the previous step. For example, Oklahoma County CoC has a total general population of 469,201. This includes the partial population of the Cleveland County sub-area that was created in step one above (61,978) as well as 50.0% of Oklahoma County’s population (387,346), 14.9% of Canadian County’s population (19,833), and a tiny part of Pottawatomie County’s population (44).

3. The homeless population of the CoC is then apportioned to the sub-areas in proportion to their percentage of the CoC’s overall population. For example, the sub-area of Cleveland County that falls within Oklahoma City CoC represents 13.2% of the CoC’s general population (61,978 of 469,201) and is therefore assumed to represent the same percentage of the CoC’s homeless population (156 of 1,183).

4. Finally, the homeless population of each county is calculated by aggregating the homeless populations of each sub-area that falls within the county’s boundaries. So, for example, Cleveland County has a total homeless population of 520. This includes the 156 people from Oklahoma City CoC as well as the 364 people from Norman/Cleveland County CoC.

Using the process above, we can algorithmically generate homeless estimates for each county in the United States, and those estimates can be aggregated to create total counts for each of the country's 390 MSAs. Those counts are available in the file  [2018 MSA Homeless Population.csv](https://github.com/adekom/metropolitan-homelessness/blob/master/2018%20MSA%20Homeless%20Population.csv).

## A Note on Methodology
The above method for apportioning population is by no means perfect. One big source of error is the assumption that the general population of each county is evenly distributed throughout that county’s area. We know that this is not the case in the real world and that urban, suburban, and rural portions of counties can vary greatly in terms of population density.

To see how this affects how homeless populations are apportioned, let’s look at Albuquerque, New Mexico. Albuquerque CoC is a fraction of the size of Bernalillo County, representing only 16.5% of the county’s area. Our methodology assumes that the population of the county is uniformly distributed, and therefore 16.5% of Bernalillo County’s population (111,563 of 674,855) is apportioned to the sub-area that falls within the CoC.

As it turns out, the boundary of Albuquerque CoC corresponds to the City of Albuquerque, and we know that the population of that area in 2017 was closer to 558,545 -- five times larger than our "one size fits all" methodology assumes.

The effect of this inaccurate apportioning of the general population is that homelessness within Bernalillo County is likely over represented while homelessness in all other counties in New Mexico (which all fall within the New Mexico Balance of State CoC) is likely under represented. 

Because homeless populations are apportioned based on the general population of the underlying sub-areas, Bernalillo County not only receives 100% of Albuquerque CoC’s homeless population, it also gets a larger proportion of the Balance of State CoC’s homeless population. This is because the sub-area of the County that is outside of Albuquerque CoC has 446,982 more people than it should.

This is a somewhat unique condition -- there are very few CoCs that are smaller than Counties -- but it exposes a weakness in the approach for apportioning populations across diffrent geographic boundaries. A more accurate way to apportion population would be to use boundaries that denote changes in population density rather than boundaries that represent governmental/administrative areas. One such boundary would be “Urban Areas” as defined by the Census Bureau. However, while the shapefiles for these geographies are available online, the corresponding population counts from the American Community Survey are not, likely because the ACS does not provide the same in-depth breakdown of information as the decennial census.

## Calculating Rates of Homelessness
A “representation ratio” has also been generated for each MSA. This is calculated by dividing the MSA’s share of the nation’s homeless population by the MSA’s share of the nation’s general population. A value of 1 indicates that the rate of homelessness is equal to the national rate of homelessness. The representation ratios are available in the file [2018 MSA Homeless Representation.csv](https://github.com/adekom/metropolitan-homelessness/blob/master/2018%20MSA%20Homeless%20Representation.csv).

It's worth noting that with the representation ratios as well, the shortcomings of the apportioning methodology are visible. For MSAs that fall within large CoCs: because the homeless population is apportioned relative to each county's general population, the rates of homelessness within those counties will be equal to that of the CoC. In reality rates of homelessness are likely higher or lower for each individual county.

We can see this play out in the data when we look at the representation ratios for the four MSAs that fall within Oregon Balance of State CoC (OR-505). These MSAs (Grant's Pass, Salem, Albany, and Corvallis) all have representation ratios within .05 of one another. The reason that they are not equal is because homeless counts are apportioned to counties using the 2017 ACS data for county populations while the representation ratio for MSAs is calculated using the 2018 ACS data for MSAs.

Homeless population is not 100% correlated to general population, so typically we can expect variation in rates of homelessnes from one MSA to the next. That is precisely what the representation ratio aims to show, but it can be a highly imperfect measure when the underlying homeless counts are apportioned from a much broader area.

## Datasets Used
The following publicly available data was used to derive homeless population counts for each MSA.

[US Census County Shapefile](https://www.census.gov/cgi-bin/geo/shapefiles/index.php)

This was used to obtain county boundaries. Because coastlines are not represented in the shapefile, the geometry was intersected with the CoC shapefile to generate more accurate land areas (although interior bodies of water such as lakes are not accounted for). Counties are also merged to create MSAs.

[HUD CoC Shapefile](https://www.hudexchange.info/programs/coc/gis-tools/)

This was used to obtain CoC Boundaries. Because the CoC shapefile and the county shapefile come from two different sources, boundaries can differ slightly even when the two shapefiles intend to represent the same geographic area (such as a county). To avoid sliver areas resulting from an intersection of the two shapefiles, the CoC boundaries were rebuilt using the county shapefile. Where CoC and county boundaries are similar, the county boundary is used. This will lead to some unknown margin of error, but it is assumed to be negligible.

[US Census 2017 American Community Survey](https://data.census.gov/)

This was used to obtain general population statistics for each county. Ideally the 2018 ACS would be used in order to match the survey year of the HUD data, but the online data that was available for that year at the time of publishing this was incomplete.

[HUD 2018 Point in Time Count](https://www.hudexchange.info/resource/3031/pit-and-hic-data-since-2007/)

This was used to obtain homeless population statistics for each CoC.

## Files Generated
[Continuum of Care.gpkg](https://drive.google.com/open?id=1aYlBJGMMk8CF4_ZBXqwn2_PrPyoLD-Pl)

This shapefile is a rebuilt version of the HUD shapefile with boundaries that more closely follow those of underlying counties.

[County.gpkg](https://drive.google.com/open?id=1Bs6XCIgENnkKaE6mzRuFM_Cl79wbceQS)

This shapefile is a rebuilt version of the Census Bureau shapefile with coastlines based on the HUD shapefile. Each feature includes fields related to general population and homeless population.

[Metropolitan Statistical Area.gpkg](https://drive.google.com/open?id=1b6g8zR7ZxGQY798gTWbsyUZmf0tIVqgu)

This shapefile merges together county boundaries and statistics.

[MSA Homeless Population.csv](https://github.com/adekom/metropolitan-homelessness/blob/master/MSA%20Homeless%20Population.csv)

This tabulation estimates a total homeless population for each Metropolitan Statistical Area.

[MSA Homeless Representation.csv](https://github.com/adekom/metropolitan-homelessness/blob/master/MSA%20Homeless%20Representation.csv)

This tabulation uses the estimates above to generate a “representation ratio” for each MSA. This is calculated by dividing the MSA’s share of the national homeless population by the MSA’s share of the national general population. A value of 1 indicates that the rate of homelessness is equal to the national rate of homelessness. A value of 2 indicates that the rate of homelessness is double the national rate of homelessness, ie, twice as many homeless people per capita.
