#eowb-cckp

Applications to enhance the [World Bank CCKP Portal](http://sdwebx.worldbank.org/climateportal/index.cfm) by integration of European EO-based datasets

##ReoWBcckp R package

ReoWBcckp is an R package to:

* Ease the access to OGC Web Coverage Service data 
* Access a country Exclusive Economic Zone (EEZ) spatial envelope
* Access Exclusive Economic Zone (EEZ) spatial data using the country ISO code

By putting those features together, extracting the European Space Agency (ESA) Climate Change Initiative data for a given country becomes straightforward.

The example code below retrieves the Sea Level Anomaly for Portugal's EEZ in 2010.

```coffee
library(devtools)
library(raster)
install_github("eowb-cckp", username="Terradue", subdir="/src/main/R/ReoWBcckp", ref="dev")
library("ReoWBcckp")

# PRT is Portugal's ISO 3166-1 alpha-3 code
country.code <- "PRT"

# get the WCS GET request template
wcs.template <- GetWCSTemplate()

# fill the values
wcs.template$value[wcs.template$param == "service"] <- "WCS" 
wcs.template$value[wcs.template$param == "version"] <- "1.0.0"
wcs.template$value[wcs.template$param == "request"] <- "GetCoverage"
wcs.template$value[wcs.template$param == "coverage"] <- "sla"
wcs.template$value[wcs.template$param == "format"] <- "NetCDF3"

# Get Portugal's minimum bounding box for the WCS request
wcs.template$value[wcs.template$param == "bbox"] <- GetCountryEnvelope(country.code)

# the list of WCS access points for 2010
coverages <- c(
  "http://catalogue.eowb-cckp.terradue.int/thredds/wcs/SeaLevel-ECV/V1.1_20131220/ESACCI-SEALEVEL-L4-MSLA-MERGED-20100115000000-fv01.nc",
  "http://catalogue.eowb-cckp.terradue.int/thredds/wcs/SeaLevel-ECV/V1.1_20131220/ESACCI-SEALEVEL-L4-MSLA-MERGED-20100215000000-fv01.nc",
  "http://catalogue.eowb-cckp.terradue.int/thredds/wcs/SeaLevel-ECV/V1.1_20131220/ESACCI-SEALEVEL-L4-MSLA-MERGED-20100315000000-fv01.nc",
  "http://catalogue.eowb-cckp.terradue.int/thredds/wcs/SeaLevel-ECV/V1.1_20131220/ESACCI-SEALEVEL-L4-MSLA-MERGED-20100415000000-fv01.nc",
  "http://catalogue.eowb-cckp.terradue.int/thredds/wcs/SeaLevel-ECV/V1.1_20131220/ESACCI-SEALEVEL-L4-MSLA-MERGED-20100515000000-fv01.nc",
  "http://catalogue.eowb-cckp.terradue.int/thredds/wcs/SeaLevel-ECV/V1.1_20131220/ESACCI-SEALEVEL-L4-MSLA-MERGED-20100615000000-fv01.nc",
  "http://catalogue.eowb-cckp.terradue.int/thredds/wcs/SeaLevel-ECV/V1.1_20131220/ESACCI-SEALEVEL-L4-MSLA-MERGED-20100715000000-fv01.nc", 
  "http://catalogue.eowb-cckp.terradue.int/thredds/wcs/SeaLevel-ECV/V1.1_20131220/ESACCI-SEALEVEL-L4-MSLA-MERGED-20100815000000-fv01.nc",
  "http://catalogue.eowb-cckp.terradue.int/thredds/wcs/SeaLevel-ECV/V1.1_20131220/ESACCI-SEALEVEL-L4-MSLA-MERGED-20100915000000-fv01.nc",
  "http://catalogue.eowb-cckp.terradue.int/thredds/wcs/SeaLevel-ECV/V1.1_20131220/ESACCI-SEALEVEL-L4-MSLA-MERGED-20101015000000-fv01.nc",
  "http://catalogue.eowb-cckp.terradue.int/thredds/wcs/SeaLevel-ECV/V1.1_20131220/ESACCI-SEALEVEL-L4-MSLA-MERGED-20101115000000-fv01.nc",
  "http://catalogue.eowb-cckp.terradue.int/thredds/wcs/SeaLevel-ECV/V1.1_20131220/ESACCI-SEALEVEL-L4-MSLA-MERGED-20101215000000-fv01.nc") 


r.stack <- c()

for (coverage in coverages) {

  # get the coverage by value (a netcdf)  
  r <- GetWCSCoverage(coverage, wcs.template, by.ref=FALSE)
  
  # shift the raster 
  r.shift <- shift(r, x=-360,y=0)
  
  # extract the values for the EEZ
  r.mask <- mask(r.shift, GetCountryEEZ(country.code))
  
  # add the clipped raster to the stack
  r.stack <- c(r.stack, r.mask)
}

# visualize with rasterVis package
library(rasterVis)

# create the monthly indexes and assign the month abbreviation to rasters  
idx <- seq(as.Date('2010-01-15'), as.Date('2010-12-15'), 'month')
my.stack <- setZ(stack(r.stack), idx)
names(my.stack) <- month.abb

# do a nice plot
levelplot(my.stack,layout=c(6, 2))

```

The image generated by the code above is shown below.

![alt text](examples/prt.png)

The data used in the example has been downloaded from http://www.esa-sealevel-cci.org and exposed as OGC WCS Coverage using [THREDDS](http://www.unidata.ucar.edu/software/thredds/current/tds/).


## Questions, bugs, and suggestions

Please file any bugs or questions as [issues](https://github.com/Terradue/eowb-cckp/issues/new) or send in a pull request.
