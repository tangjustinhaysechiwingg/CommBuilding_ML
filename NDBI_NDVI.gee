// Satellite platform
var platform = 's2'; // set as 'L5', 'L7', 'L8' or 'S2'

//*******************************************************************************************
//                                SET ATMOSPHERIC CORRECTION
//
// Select whether to use atmospherically corrected surface reflectance imagery.
// If left blank top-of-atmosphere reflectance will be selected.
// More information on both forms of imagery is available in the link below.
// https://developers.google.com/earth-engine/landsat
// Enter 'SR' or 'sr' to select surface reflectance.

// Atmospheric correction
var atmos = 'sr'; // set as either sr for surface reflactance or leave blank for top-of-atmosphere

// Note: Sentinel-2 surface reflectance data is available from 2017-03-28 onwards.
// Sentinel-2 top-of-atmosphere data is available from 2015-06-23 onwards.

//*******************************************************************************************
//                                     SET TIME FRAME
//
// Set start and end dates for the composite. Seasonal or annual time frames are recommended.
// Select a time frame appropriate for the satellite platform chosen. Shorter time frames
// will contain more cloud cover depending on season and may contain data gaps.

// Landsat 5        1984-01-01 to 2012-05-05
// Landsat 7        1999-01-01 to Present
// Landsat 8        2013-04-11 to Present
// Sentinel-2 TOA   2015-06-23 to Present
// Sentinel-2 SR    2017-03-28 to Present

// Note: Choosing a time frame outside of a satellites lifespan will produce no results.

// Start date
var start = '2019-01-01'; // start date for compositing

// End date 
var end = '2019-12-31'; // end date for compositing

//*******************************************************************************************
//                                       SET STUDY AREA
//
// Define a study area using a polygon.
// Polygons can be created using the geometry tools in the top-left of the map window
// The polygon can be renamed (default is geometry). Doing so will require 
// that the variable in brackets below is changed accordingly.
// Otherwise the values below do not need to be modified.
// A study area can also be imported using the 'Assets' tab in the left-hand panel.

// Location boundary
var area = ee.FeatureCollection(geometry); // match the name in brackets to the desired boundary

// Set study area as map center and print to console.
Map.centerObject(area);
print(area);

//*******************************************************************************************
//----------------------->>> NOW HIT 'RUN' AT THE TOP OF THE WINDOW <<<----------------------
//----------------------A true-colour image will be displayed by default.--------------------
//----------Use the tickboxes under the "Layers" menu in the top-right of the map window-----
//----------------to turn on and off different band combinations and indices-----------------
//-------------Exportable imagery will be available in the tasks tab to the right------------
//*******************************************************************************************

//------------------------- Print time frame and platform to console ------------------------
if (platform == 'L5' | platform == 'l5') {
  if (atmos == 'SR' | atmos == 'sr'){
    var ImCol = 'LANDSAT/LT05/C01/T1_SR';
    var pl = 'Landsat 5 SR';
  } else {
    var ImCol = 'LANDSAT/LT05/C01/T1_TOA';
    var pl = 'Landsat 5 TOA';
}
} else if (platform == 'L7' | platform == 'l7') {
  if (atmos == 'SR' | atmos == 'sr'){
    var ImCol = 'LANDSAT/LE07/C01/T1_SR';
    var pl = 'Landsat 7 SR';
  } else {
    var ImCol = 'LANDSAT/LE07/C01/T1_TOA';
    var pl = 'Landsat 7 TOA';
}
} else if (platform == 'L8' | platform == 'l8') {
  if (atmos == 'SR' | atmos == 'sr'){
    var ImCol = 'LANDSAT/LC08/C01/T1_SR';
    var pl = 'Landsat 8 SR';
  } else {
    var ImCol = 'LANDSAT/LC08/C01/T1_TOA';
    var pl = 'Landsat 8 TOA';  
}
} else {
  if (atmos == 'SR' | atmos == 'sr'){
    var ImCol = 'COPERNICUS/S2_SR';
    var pl = 'Sentinel-2 SR';
  } else {
    var ImCol = 'COPERNICUS/S2';
    var pl = 'Sentinel-2 TOA';  
}
}

// Print Satellite platform and dates to console
print(ee.String('Start: ').cat(start));
print(ee.String('End: ').cat(end));
print(ee.String('Platform: ').cat(pl));

//================================================================================================\\
//======================= 1 - CLOUD MASKING AND IMAGE COMPOSITING ================================\\
//================================================================================================\\

// Apply cloud masking appropriate to the selected platform
if (platform == 'L5' | platform == 'l5' | platform == 'L7' | platform == 'l7') {
  if (atmos == 'SR' | atmos == 'sr'){
    var mask = function(image) {
      var qa = image.select('pixel_qa');
      // If the cloud bit (5) is set and the cloud confidence (7) is high
      // or the cloud shadow bit is set (3), then it's a bad pixel.
      var cloud = qa.bitwiseAnd(1 << 5)
              .and(qa.bitwiseAnd(1 << 7))
              .or(qa.bitwiseAnd(1 << 3));
      // Remove edge pixels that don't occur in all bands
      var mask2 = image.mask().reduce(ee.Reducer.min());
      return image.updateMask(cloud.not()).updateMask(mask2).divide(10000);
    };
  } else {
    var mask = function(image) {
      var qa = image.select('BQA');
      /// Check that the cloud bit is off.
      // See https://landsat.usgs.gov/collectionqualityband
      var mask2 = qa.bitwiseAnd(1 << 4).eq(0);
      return image.updateMask(mask2);
    };
}
} else if (platform == 'L8' | platform == 'l8')
  if (atmos == 'SR' | atmos == 'sr'){
    var mask = function(image) {
      // Bits 3 and 5 are cloud shadow and cloud, respectively, Bit 4 is Snow.
      var cloudShadowBitMask = 1 << 3;
      var cloudsBitMask = 1 << 5;
      var snowBitMask = 1 << 4;
      // Get the pixel QA band.
      var qa = image.select('pixel_qa');
      // All flags should be set to zero, indicating clear conditions.
      var mask2 = qa.bitwiseAnd(cloudShadowBitMask).eq(0)
          .and(qa.bitwiseAnd(cloudsBitMask).eq(0))
          .and(qa.bitwiseAnd(snowBitMask).eq(0));
      var qa2 = image.select('radsat_qa');
      // Return the masked image, scaled to TOA reflectance, without the QA bands.
      return image.updateMask(mask2).divide(10000)
          .copyProperties(image, ["system:time_start"]);
};
} else {
    var mask = function(image) {
      var qa = image.select('BQA');
      /// Check that the cloud bit is off.
      // See https://landsat.usgs.gov/collectionqualityband
      var mask2 = qa.bitwiseAnd(1 << 4).eq(0);
      return image.updateMask(mask2);
};
} else {
    // Function to mask clouds using the Sentinel-2 QA band.
   var mask = function(image) {
     var qa = image.select('QA60');
     // Bits 10 and 11 are clouds and cirrus, respectively.
     var cloudBitMask = 1 << 10;
     var cirrusBitMask = 1 << 11;
     // Both flags should be set to zero, indicating clear conditions.
     var mask2 = qa.bitwiseAnd(cloudBitMask).eq(0).and(
                qa.bitwiseAnd(cirrusBitMask).eq(0));
     // Return the masked and scaled data, without the QA bands.
     return image.updateMask(mask2).divide(10000)
         .select("B.*")
         .copyProperties(image, ["system:time_start"]);
   };
}

// Create composite, mask and calculate median pixel
var composite_median_v1 = ee.ImageCollection(ImCol)
      .filterDate(start, end)
      .map(mask)
      .median()
      .clip(area)
      .toFloat();

// Create composite for greenest pixel
var composite_greenest_v1 = ee.ImageCollection(ImCol)
      .filterDate(start, end)
      .map(mask);

// Rename bands for visualisation and index calculations
if (platform == 'l5' | platform == 'L5' | platform == 'l7' | platform == 'L7'){
  var composite_median = composite_median_v1
      .select(['B1', 'B2', 'B3', 'B4', 'B5', 'B7'],['blue', 'green', 'red', 'nir', 'swir1', 'swir2']);
  var composite_greenest_v2 = composite_greenest_v1
      .select(['B1', 'B2', 'B3', 'B4', 'B5', 'B7'],['blue', 'green', 'red', 'nir', 'swir1', 'swir2']);
} else if (platform == 'l8' | platform == 'L8'){
  var composite_median = composite_median_v1
      .select(['B2', 'B3', 'B4', 'B5', 'B6', 'B7'],['blue', 'green', 'red', 'nir', 'swir1', 'swir2']);
  var composite_greenest_v2 = composite_greenest_v1
      .select(['B2', 'B3', 'B4', 'B5', 'B6', 'B7'],['blue', 'green', 'red', 'nir', 'swir1', 'swir2']);
} else {
  var composite_median = composite_median_v1
      .select(['B2', 'B3', 'B4', 'B8', 'B11', 'B12'],['blue', 'green', 'red', 'nir', 'swir1', 'swir2']);
  var composite_greenest_v2 = composite_greenest_v1
      .select(['B2', 'B3', 'B4', 'B8', 'B11', 'B12'],['blue', 'green', 'red', 'nir', 'swir1', 'swir2']);
}

// Add NDVI band for quality mosaic
  var addNDVI = function(composite_greenest_v2) {
  var ndvi = composite_greenest_v2.normalizedDifference(['nir', 'red']).rename('NDVI')
      .clip(area);
  return composite_greenest_v2.addBands(ndvi);
};

// Final greenest pixel composite calculation
var withNDVI = composite_greenest_v2.map(addNDVI);
var composite_greenest = withNDVI.qualityMosaic('NDVI');

//================================================================================================\\
//====================== 2 - IMAGE VISUALISATION AND INDEX CALCULATIONS ==========================\\
// Calculate useful image band combinations and spectral indices, then display them on the map.   \\
//================================================================================================\\

//**************************************************************************************************
//                                    TRUE COLOUR IMAGE
// Composite true colour image at median pixel values across selected time period..

  Map.addLayer(composite_median, {bands: ['red', 'green', 'blue'], min: 0, max: 0.3}, 'RGB');

//**************************************************************************************************
//                                 GREENEST PIXEL COMPOSITE
// Composite true colour image at maximum NDVI. Water and impermeable surfaces may appear cloudy as 
// cloud cover may have a higher NDVI value than these surfaces.

// True-colour image at maximum NDVI
  Map.addLayer(composite_greenest, {bands: ['red', 'green', 'blue'], min: 0, max: 0.3}, 'Greenest pixel composite', false);

//**************************************************************************************************
//                             VEGETATION FALSE COLOUR COMPOSITE
//
// Vegetation appears vibrant red. Hardwood trees often appear lighter than Conifers.
// Soils vary from dark to light browns and urban areas can appear cyan blue, grey and yellow.
// Grasslands appear yellow. Other vegetation appears as less vibrant shades of green.

  // Vegetation false-colour image.
  Map.addLayer(composite_median, {bands: ['nir', 'red', 'green'], min: 0, max: 0.3}, 'Infra', false);
  
//**************************************************************************************************
//                             AGRICULTURE FALSE COLOUR COMPOSITE
//
// Crops appear bright green. Bare earth appears magenta. Grasslands appear yellow.
// Other vegetation appears as less vibrant shades of green.

  Map.addLayer(composite_median, {bands: ['swir1', 'nir', 'blue'], min: 0, max: 0.3}, 'Agriculture', false);

//**************************************************************************************************
//                               URBAN FALSE COLOUR COMPOSITE
//
// Vegetation appears green. Urbanized areas are represented by white, gray, or purple.
// Soils, sand, and minerals are shown in a variety of colors.
// Snow and ice appear as dark blue, and water as black or blue.

  Map.addLayer(composite_median, {bands: ['swir2', 'swir1', 'red'], min: 0, max: 0.3}, 'Urban', false);

//**************************************************************************************************
//                           NORMALISED DIFFERENCE VEGETATION INDEX
//
// The value range of an NDVI is -1 to 1.                       
// Negative values of NDVI (values approaching -1) correspond to water.           
// Values close to zero (-0.1 to 0.1) generally correspond to barren areas of rock, sand, or snow.
// Low, positive values represent shrub and grassland (approximately 0.2 to 0.4),                 
// while high values indicate temperate and tropical rainforests (values approaching 1).          

// Calculate NDVI
  var NDVI = composite_median.expression(
  '(nir - red) / (nir + red)',
  {
  red: composite_median.select('red'),
  nir: composite_median.select('nir')
  });


// Rename and apply mask to DNs outside range
  var NDVI = NDVI.select('nir').rename('NDVI');
  var NDVI = NDVI.select('NDVI');
  var maskGT = NDVI.gt(-1);
  var maskLT = NDVI.lt(1);
  var NDVI = NDVI.updateMask(maskGT).updateMask(maskLT);


// Adjusted NDVI
if (platform == 'L5' | platform == 'l5') {
  if (atmos == 'SR' | atmos == 'sr') {
    var NDVI_adjusted = NDVI.expression(
    '0.0149 + 1.0035 * L5_SR',
    {
    L5_SR: NDVI.select('NDVI')
    });
  } else {
    var NDVI_adjusted = NDVI.expression(
    '0.0306 + 0.9824 * L5_TOA',
    {
    L5_TOA: NDVI.select('NDVI')
    });
}
} else if (platform == 'L7' | platform == 'l7') {
  if (atmos == 'SR' | atmos == 'sr') {
    var NDVI_adjusted = NDVI.expression(
    '0.0235 + 0.9723 * L7_SR',
    {
    L7_SR: NDVI.select('NDVI')
    });
  } else {
    var NDVI_adjusted = NDVI.expression(
    '0.0490 + 0.9352 * L7_TOA',
    {
    L7_TOA: NDVI.select('NDVI')
    });
  }
} else if (platform == 'L8' || platform == 'l8'){
    if (atmos == 'SR' | atmos == 'sr') {
    var NDVI_adjusted = NDVI.expression(
    '(0.0110*-1) + 0.9690 * L8_SR',
    {
    L8_SR: NDVI.select('NDVI')
    });
  } else {
    var NDVI_adjusted = NDVI.expression(
    '0.0490 + 0.9352 * L8_TOA',
    {
    L8_TOA: NDVI.select('NDVI')
    });
  }
} else {}


// Rename and apply mask to DNs outside range
if (platform == "l5" | platform == "L5" | platform == "l7" | platform == "L7" | platform == "l8" | platform == "L8"){
  var maskGT = NDVI_adjusted.gt(-1);
  var maskLT = NDVI_adjusted.lt(1);
  var NDVI_adjusted = NDVI_adjusted.updateMask(maskGT).updateMask(maskLT);
} else {}
  
// Display NDVI
  Map.addLayer (NDVI, {min: -0.5, max: 1, palette: ['505050', '505050', 'E8E8E8', '00FF33', '003300']},
'NDVI', false);

// Display Greenest Pixel NDVI
  Map.addLayer(composite_greenest, {bands: ['NDVI'], min: -0.5, max: 1, palette: ['505050', '505050', 'E8E8E8', '00FF33', '003300']},
'NDVI_Max', false);

// Display Adjusted NDVI
if (platform == "l5" | platform == "L5" | platform == "l7" | platform == "L7" | platform == "l8" | platform == "L8"){
  Map.addLayer (NDVI_adjusted, {min: -0.5, max: 1, palette: ['505050', '505050', 'E8E8E8', '00FF33', '003300']},
'NDVI_Adjusted', false);
} else {}


//**************************************************************************************************
//                                 ENHANCED VEGETATION INDEX
//
// Seeks to address the limitation of NDVI which can oversaturate in high biomass areas.
// The range of values for the EVI is -1 to 1.                
// Healthy vegetation generally falls between values of 0.20 to 0.80. 

// Calculate EVI
  var EVI = composite_median.expression(
  '2.5 * (nir - red) / (nir + 6 * red - 7.5 * blue + 1)',
  {
  blue: composite_median.select('blue'),
  red: composite_median.select('red'),
  nir: composite_median.select('nir')
  });

// Display EVI
  Map.addLayer (EVI, {min: -0.5, max: 1, palette: ['505050', '505050', 'E8E8E8', '00FF33', '003300']},
'EVI', false);

//**************************************************************************************************
//                           MODIFIED SOIL-ADJUSTED VEGETATION INDEX 2
//
// Seeks to address the limitation of NDVI when applied to areas with a high          
// degree of exposed soil. Unlike other soil-adjusted vegetation indices,             
// MSAVI2 removes the need to explicitly specify the soil brightness correction factor.   
// MSAVI2 represents vegetation greenness with values ranging from -1 to +1.              

// Calculate MSAVI
  var MSAVI = composite_median.expression(
  '(2 * nir + 1 - sqrt(pow((2 * nir + 1), 2) - 8 * (nir - red)) ) / 2',
  {
  red: composite_median.select('red'),
  nir: composite_median.select('nir')
  });

// Display MSAVI
  Map.addLayer (MSAVI, {min: -0.5, max: 1, palette: ['505050', '505050', 'E8E8E8', '00FF33', '003300']},
'MSAVI', false);

//**************************************************************************************************
//                             NORMALISED DIFFERENCE WATER INDEX
//
// Values of water bodies are larger than 0.5. 
// Vegetation has much smaller values         
// Built-up features have positive values between zero and 0.2.  

// Calculate NDWI
  var NDWI = composite_median.expression(
  '(nir - swir1) / (nir + swir1)',
  {
  nir: composite_median.select('nir'),
  swir1: composite_median.select('swir1')
  });

// Display NDWI
  Map.addLayer (NDWI, {min: -1, max: 1, palette: ['505050', '505050', 'BFEFFF', '0000FF', '000080']},
'NDWI', false);

//**************************************************************************************************
//                             NORMALISED DIFFERENCE SNOW INDEX
//
// Snow is highly reflective in the visible part of the EM spectrum 
// and highly absorptive in the near-infrared or short-wave infrared part of the spectrum, 
// whereas the reflectance of most clouds remains high in those same parts of the spectrum.
// Values range from -1 to 1 with large positive values indicating snow or cloud cover.   

// Calculate NDSI
  var NDSI = composite_median.expression(
  '(green - swir1) / (green + swir1)',
  {
  green: composite_median.select('green'),
  swir1: composite_median.select('swir1')
  });

// Display NDSI
  Map.addLayer (NDSI, {min: -0.5, max: 1, palette: ['505050', '505050', 'BFEFFF', '0000FF', '000080']},
'NDSI', false);

//**************************************************************************************************
//                                        URBAN INDEX
//
// Build-up areas and bare soil reflects more SWIR than NIR.                                     
// Higher values generally represent the impervious surfaces found in build-up areas.            
// Negative value of UI represent water bodies.                                                  
// UI value for vegetation is low.                                                               

// Calculate UI
  var UI = composite_median.expression(
  '(swir2 - nir) / (swir2 + nir)',
  {
  nir: composite_median.select('nir'),
  swir2: composite_median.select('swir2')
  });

// Display UI
  Map.addLayer (UI, {min: -1, max: 1, palette: ['505050', '505050', 'E8E8E8', 'FF0000', '800080']},
'UI', false);

//**************************************************************************************************
//                               NORMALISED DIFFERENCE BUILT-UP INDEX
//
// Build-up areas and bare soil reflects more SWIR than NIR.        
// Higher values generally represent the impervious surfaces found in build-up areas.
// Negative value of NDBI represent water bodies.   
// NDBI value for vegetation is low.                

// Calculate NDBI
  var NDBI = composite_median.expression(
  '(swir1 - nir) / (swir1 + nir)',
  {
  nir: composite_median.select('nir'),
  swir1: composite_median.select('swir1')
  });

// Display NDBI
  Map.addLayer (NDBI, {min: -1, max: 1, palette: ['505050', '505050', 'E8E8E8', 'FF0000', '800080']},
'NDBI', false);

//================================================================================================\\
//======================================= 3 - EXPORT =============================================\\
//================================================================================================\\
// Scale calculation
if (platform == 'S2' || platform == 's2'){
  var scale = 10;
} else {
  var scale = 30;
}

// Export the image, specifying scale and region.
Export.image.toDrive({
  image: composite_median_v1,
  description: 'composite_median',
  region: area.geometry().bounds(),
  fileNamePrefix: platform + '_composite_median',
  maxPixels: 1e13,
});
Export.image.toDrive({
  image: composite_greenest,
  description: 'composite_greenest',
  region: area.geometry().bounds(),
  fileNamePrefix: platform + '_composite_greenest',
  scale: scale,
  maxPixels: 1e13,
});
Export.image.toDrive({
  image: NDVI,
  description: 'NDVI',
  region: area.geometry().bounds(),
  fileNamePrefix: platform + '_NDVI',
  scale: scale,
  maxPixels: 1e13,
});
Export.image.toDrive({
  image: EVI,
  description: 'EVI',
  region: area.geometry().bounds(),
  fileNamePrefix: platform + '_EVI',
  scale: scale,
  maxPixels: 1e13,
});
Export.image.toDrive({
  image: MSAVI,
  description: 'MSAVI',
  region: area.geometry().bounds(),
  fileNamePrefix: platform + '_MSAVI',
  scale: scale,
  maxPixels: 1e13,
});
Export.image.toDrive({
  image: NDWI,
  description: 'NDWI',
  region: area.geometry().bounds(),
  fileNamePrefix: platform + '_NDWI',
  scale: scale,
  maxPixels: 1e13,
});
Export.image.toDrive({
  image: NDSI,
  description: 'NDSI',
  region: area.geometry().bounds(),
  fileNamePrefix: platform + '_NDSI',
  scale: scale,
  maxPixels: 1e13,
});
Export.image.toDrive({
  image: UI,
  description: 'UI',
  region: area.geometry().bounds(),
  fileNamePrefix: platform + '_UI',
  scale: scale,
  maxPixels: 1e13,
});
Export.image.toDrive({
  image: NDBI,
  description: 'NDBI',
  region: area.geometry().bounds(),
  fileNamePrefix: platform + '_NDBI',
  scale: scale,
  maxPixels: 1e13,
});
Export.image.toDrive({
  image: composite_greenest.select('NDVI'),
  description: 'NDVI_Max',
  region: area.geometry().bounds(),
  fileNamePrefix: platform + '_NDVI_MAX',
  scale: scale,
  maxPixels: 1e13,
});

Export.image.toDrive({
  image: NDVI_adjusted,
  description: 'NDVI_Adjusted',
  region: area.geometry().bounds(),
  fileNamePrefix: platform + '_NDVI_Adjusted',
  scale: scale,
  maxPixels: 1e13,
});
