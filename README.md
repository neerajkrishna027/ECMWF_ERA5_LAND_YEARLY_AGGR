# ECMWF_ERA5_LAND_YEARLY_AGGR
// Code for Downloading the total_precipitation from ECMWF/ERA5_LAND/MONTHLY of a study area (.shp), that has been converted into mm.





// ------------------------------
// USER INPUTS
// ------------------------------

// Import your study area (rename table -> studyArea if uploaded as shapefile asset)
var studyArea = table;  

// ERA5-Land Monthly Aggregated dataset
var dataset = ee.ImageCollection("ECMWF/ERA5_LAND/MONTHLY");

// Define year range
var startYear = 1951;
var endYear = 2024;

// ------------------------------
// FUNCTION: Get yearly total precipitation
// ------------------------------
function getYearlyPrecip(year) {
  // Filter for that year
  var yearCollection = dataset
    .filterDate(ee.Date.fromYMD(year, 1, 1), ee.Date.fromYMD(year + 1, 1, 1))
    .select("total_precipitation");

  // Sum over all months
  var yearlyTotal = yearCollection.sum()
    // Convert from meters → millimeters
    .multiply(1000)
    .rename("total_precipitation_mm")
    .set("year", year);

  return yearlyTotal.clip(table);
}

// ------------------------------
// LOOP THROUGH YEARS + EXPORT
// ------------------------------
var years = ee.List.sequence(startYear, endYear);

years.getInfo().forEach(function(y) {
  var yearlyPrecip = getYearlyPrecip(y);
  
  Export.image.toDrive({
    image: yearlyPrecip,
    description: "ERA5LAND_total_precipitation_" + y,
    folder: "GEE_exports",
    fileNamePrefix: "ERA5LAND_total_precipitation_" + y,
    region: table,
    scale: 10000,   // adjust resolution (~0.1° ≈ 10 km)
    crs: "EPSG:4326",
    maxPixels: 1e13
  });
});
