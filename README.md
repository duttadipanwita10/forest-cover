//Forest Cover classification in North Bengal
//Author: Dipanwita Dutta
//Date: 2023-2024

//var Landsat8_T1_SR_2023 = ee.ImageCollection('LANDSAT/LC08/C02/T1_L2')
//                    .filterDate('start date', 'end date')
//                    .filterBounds(aoi)
//                    .map(maskL8);
//Import Image, Mask Clouds and Median Value
function maskS2clouds(image){
var qa = image.select('QA60');

// Bits 10 and 11 are clouds and cirrus, respectively.
var cloudBitMask = 1 << 10;
var cirrusBitMask = 1 << 11;

// Both flags should be set to zero, indicating clear conditions.
var mask = qa.bitwiseAnd(cloudBitMask).eq(0)
     .and(qa.bitwiseAnd(cirrusBitMask).eq(0));

  return image.updateMask(mask).divide(10000)}
  
var Sentinel_S2_SR = ee.ImageCollection('COPERNICUS/S2_SR')
                    .filterDate('start date', 'end date')
                    .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 5))
                    .filterBounds(roi)
                    .map(maskS2clouds)
                    .median();
                    
var NIR_year = Sentinel_S2_SR.select('B8').clip(roi);
var Red_year = Sentinel_S2_SR.select('B4').clip(roi);
var NDVI_year_original = NIR_year.subtract(Red_year).divide(NIR_year.add(Red_year));

var NDVI_year_histogram = ui.Chart.image.histogram({image: NDVI_year_original, region: roi, scale: 30, maxPixels: 1e13});
print(NDVI_year_histogram);

var NDVI_year_mask = NDVI_year_original.where(NDVI_year_original.lte(0), 0).where(NDVI_year_original.gt(0), 1);
var NDVI_year = NDVI_year_original.multiply(NDVI_year_mask);

var min_year = ee.Number(NDVI_year.reduceRegion({
  reducer: ee.Reducer.min(),
  geometry: roi,
  scale: 30,
  maxPixels: 1e9
}).values().get(0));

print(min_year);

var max_year = ee.Number(NDVI_year.reduceRegion({
  reducer: ee.Reducer.max(),
  geometry: roi,
  scale: 30,
  maxPixels: 1e9
}).values().get(0));

print(max_year);

var NDVIfc_year = NDVI_year.subtract(min_year).divide(max_year.subtract(min_year));

var NDVIfc_year_histogram = ui.Chart.image.histogram({image: NDVIfc_year, region: roi, scale: 30, maxPixels: 1e13});
print(NDVIfc_year_histogram);

var NDVIfc_year_reclass = NDVIfc_year.where(NDVIfc_year.lt(0.20), 0)
                             .where(NDVIfc_year.gte(range start value).and(NDVIfc_year.lt(range end value)), 1)
                             .where(NDVIfc_year.gte(range start value).and(NDVIfc_year.lt(range end value)), 2)
                             .where(NDVIfc_year.gte(range start value).and(NDVIfc_year.lt(range end value)), 3);
Map.addLayer(NDVIfc_year_reclass, {min: 0, max: 3, palette: ['000000', 'D9F55E', '35CF23', '035706']}, 'Forest Density year', false);

