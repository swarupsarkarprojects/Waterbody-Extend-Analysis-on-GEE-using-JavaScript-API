
# Surface water extent time series analysis on GEE using JavaScript API


Unlock the power of Google Earth Engine (GEE) with our comprehensive tutorial on conducting surface water extent time series analysis. In this tutorial, we delve into utilizing the JavaScript API of GEE to analyze changes in surface water over time.

Our step-by-step guide covers everything from data acquisition and preprocessing to visualization and interpretation. You'll learn how to harness GEE's vast repository of satellite imagery and geospatial datasets to derive meaningful insights into surface water dynamics.


## API Reference

#### From Google Earth Engine for Python API

```http
  https://developers.google.com/earth-engine
```

| Parameter | Type     | Description                |
| :-------- | :------- | :------------------------- |
| `Earth Enigne API` | `Geospatial Imegary` | **Required**. Access the Geospatial Datasets |




## Follow the step for the Project

Create Sentinel 2 ImageCollection and filters datasets as per required:

```bash
var datasets = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
                 .filterBounds(geometry)
                 .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE',30))
```

Build a function for NDWI index which helps to identify the Surface Waterbody and Add with ImageCollection using ".map" function:

```bash
.map(function(img){
                   var bands = img.select('B*').multiply(0.0001);
                   var ndwi = img.normalizedDifference(['B3','B8']).rename('ndwi');
                   return ndwi
                   .copyProperties(img, img.propertyNames())
                 })
```
## Create DateRange or sequence for listed all spatial images

```bash
var startyear = 2021;
var endyear = 2021;
var year = ee.List.sequence(startyear, endyear);
var month = ee.List.sequence(1,12);
```
## Create a function for listing all monthly datasets in a Variable:

```bash
var monthly = ee.ImageCollection.fromImages(
  year.map(function(year){
    return month.map(function(month){
        return datasets.filter(ee.Filter.calendarRange(year,year,'year'))
                    .filter(ee.Filter.calendarRange(month,month,'month'))
                    .median()
                    .set('year',year)
                    .set('month',month);
    });
  }).flatten()
);
```

## Create a Clip function for subset the datasets according to yout ROI:

```bash
var clip_all = function(img){
  return img.clip(geometry)
}
var clip = monthly.map(clip_all);
```

## Listing all datasets and calculate the total image collection for the Next Loop:

```bash
var list = clip.toList(clip.size());
var n = list.size().getInfo();
```

## Create a For-Loop for calculate one-by-one image for Waterbody area and store in a Variable:

```bash
var array = [];

for (var i=0; i<n; i++){
  var img = ee.Image(list.get(i));
  var classify = img.gt(0);
  var mask = classify.updateMask(classify);
  var func = ee.Image.pixelArea().multiply(mask);
  var area = func.reduceRegion({
    reducer: ee.Reducer.sum(),
    geometry:geometry,
    scale:10,
    maxPixels: 1e13
  })
  var area_sqkm = ee.Number(area.get('area')).divide(1e6).round();
  array[i]= area_sqkm
}
```

## Create a chart for the area plot:

```bash
var chart = ui.Chart.array.values({array:ee.List(array), axis:0})
                  .setOptions({
                    title: 'Waterbody Area'
                  });
```


## 🔗 Visit on Profile
[![Guihub](https://img.shields.io/badge/my_portfolio-000?style=for-the-badge&logo=ko-fi&logoColor=white)](https://github.com/swarupsarkarprojects?tab=repositories)
[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/swarup-sarkar-9371251b4/)


