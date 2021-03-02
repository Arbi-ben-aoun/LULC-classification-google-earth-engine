// delineate the zone of balatopn lake 
// import landsat data 
var landsat = ee.ImageCollection('LANDSAT/LC08/C01/T1_TOA');

// filter the balation region
var region = landsat.filterBounds(balaton);

// filter the time period
var time = region.filterDate('2019-03-01', '2019-10-31');

// select the landsat data with least cloud cover
var least_cloudy= time.sort('CLOUD_COVER').first();
var study_area=least_cloudy.clip(balaton);
print(study_area);
//ndvi image
var getNDVI = function(image) {
  return image.normalizedDifference(['B5', 'B4']) };
var ndvi_image=getNDVI(study_area);

// show the region in the workplace , true, false color composite and ndvi
var visParams={bands:['B4', 'B3', 'B2'],min:0,max:0.3} ;
Map.addLayer(study_area,visParams,'true color image');
var FalsevisParams={bands:['B5', 'B4', 'B3']} ;
Map.addLayer(study_area,FalsevisParams,'false color image');
Map.addLayer(ndvi_image,{},'ndvi');

// training points from the sites
// we create 3 landcover classes 0 for 'water' ,1 for 'forest', 2 for 'urban'
var features= water.merge(forest).merge(urban);

// we select now the now the training bands and the title of column storing the labels
var bands = ['B2', 'B3', 'B4', 'B5', 'B6', 'B7','B10', 'B11'];

// create our training set 
var train_set = study_area.select(bands).sampleRegions({
  collection : features,
  properties : ['class'] ,
  scale : 30
});
print(train_set.size());
// Random forst classifier and see the classifier's results 
var classifier =ee.Classifier.smileRandomForest(20).train({
  features : train_set,
  classProperty: 'class'
});
print('results',classifier.explain());

// create our test set 
var test_features= water_test.merge(forest_test).merge(urban_test);
var test_set = study_area.select(bands).sampleRegions({
  collection : test_features,
  properties : ['class'] ,
  scale : 30
});
print(test_set.size());

// see metrics 

print(train_set.classify(classifier).errorMatrix('class', 'classification').accuracy(),'Training vs. Testing - Separate Polygons');
//see the landcover map
var classification_map= study_area.classify(classifier);
Map.addLayer(classification_map,{min: 0, max: 2, palette: ['blue', 'green', 'red']},'land use map');
Map.centerObject(balaton);

Export.image.toDrive({ image: classification_map, description: 'land cover map balaton',scale: 30, region: balaton, fileFormat: 'GeoTIFF'});
Export.image.toDrive({ image: ndvi_image, description: 'ndvi balaton',scale: 30, region: balaton,fileFormat: 'GeoTIFF'});
Export.image.toDrive({ image: study_area.toFloat(), description: 'true image', scale: 30,region: balaton, fileFormat: 'GeoTIFF'});