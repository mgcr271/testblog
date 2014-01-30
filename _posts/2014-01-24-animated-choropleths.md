---
layout: post
title: "Animated Choropleths"
quote: rMaps and DataMaps make it easy to create animated choropleths!
image: "assets/images/choropleths.jpg"
video: false
---

This post is motivated by a recent article by Vivek Patil, where he showed various ways to generate and animate choropleth maps from R. 

## Get Data

Let us fetch data from [Quandl](http://www.quandl.com/), which is an excellent source of fascinating datasets. I would strongly recommend that you check it out, if you haven't already :). 


```r
library(Quandl)
vcData = Quandl("FBI_UCR/USCRIME_TYPE_VIOLENTCRIMERATE")
kable(head(vcData[,1:9]), format = 'html')
```

<table class="table table-striped table-condensed non-fluid">
 <thead>
  <tr>
   <th> Year </th>
   <th> Alabama </th>
   <th> Alaska </th>
   <th> Arizona </th>
   <th> Arkansas </th>
   <th> California </th>
   <th> Colorado </th>
   <th> Connecticut </th>
   <th> Delaware </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td> 2010-12-31 </td>
   <td> 377.8 </td>
   <td> 638.8 </td>
   <td> 408.1 </td>
   <td> 505.3 </td>
   <td> 440.6 </td>
   <td> 320.8 </td>
   <td> 281.4 </td>
   <td> 620.9 </td>
  </tr>
  <tr>
   <td> 2009-12-31 </td>
   <td> 450.1 </td>
   <td> 633.4 </td>
   <td> 426.5 </td>
   <td> 515.8 </td>
   <td> 473.3 </td>
   <td> 338.8 </td>
   <td> 300.9 </td>
   <td> 645.4 </td>
  </tr>
  <tr>
   <td> 2008-12-31 </td>
   <td> 451.3 </td>
   <td> 650.9 </td>
   <td> 481.2 </td>
   <td> 504.6 </td>
   <td> 506.2 </td>
   <td> 347.1 </td>
   <td> 306.5 </td>
   <td> 706.1 </td>
  </tr>
  <tr>
   <td> 2007-12-31 </td>
   <td> 447.9 </td>
   <td> 662.3 </td>
   <td> 514.5 </td>
   <td> 532.6 </td>
   <td> 522.6 </td>
   <td> 350.6 </td>
   <td> 301.1 </td>
   <td> 711.5 </td>
  </tr>
  <tr>
   <td> 2006-12-31 </td>
   <td> 425.2 </td>
   <td> 686.8 </td>
   <td> 545.4 </td>
   <td> 557.2 </td>
   <td> 533.3 </td>
   <td> 394.8 </td>
   <td> 298.6 </td>
   <td> 701.0 </td>
  </tr>
  <tr>
   <td> 2005-12-31 </td>
   <td> 433.0 </td>
   <td> 632.0 </td>
   <td> 512.0 </td>
   <td> 528.0 </td>
   <td> 526.0 </td>
   <td> 397.0 </td>
   <td> 273.0 </td>
   <td> 633.0 </td>
  </tr>
</tbody>
</table>



## Reshape Data

Our dataset is in the wide-form. So let us first convert it into the long-form, as it is usually more convenient for visualization purposes. We remove data for the US as a whole, as well as for DC, so that the crime rates are comparable. 


```r
library(reshape2)
datm <- melt(vcData, 'Year', 
  variable.name = 'State',
  value.name = 'Crime'
)
datm <- subset(na.omit(datm), 
  !(State %in% c("United States", "District of Columbia"))
)
kable(head(datm), format = 'html')
```

<table class="table table-striped table-condensed non-fluid">
 <thead>
  <tr>
   <th> Year </th>
   <th> State </th>
   <th> Crime </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td> 2010-12-31 </td>
   <td> Alabama </td>
   <td> 377.8 </td>
  </tr>
  <tr>
   <td> 2009-12-31 </td>
   <td> Alabama </td>
   <td> 450.1 </td>
  </tr>
  <tr>
   <td> 2008-12-31 </td>
   <td> Alabama </td>
   <td> 451.3 </td>
  </tr>
  <tr>
   <td> 2007-12-31 </td>
   <td> Alabama </td>
   <td> 447.9 </td>
  </tr>
  <tr>
   <td> 2006-12-31 </td>
   <td> Alabama </td>
   <td> 425.2 </td>
  </tr>
  <tr>
   <td> 2005-12-31 </td>
   <td> Alabama </td>
   <td> 433.0 </td>
  </tr>
</tbody>
</table>


---

## Discretize Crime Rates

For choropleth maps, we would like to discretize crime rates. We do this by dividing crime rates into sextiles. 


```r
datm2 <- transform(datm,
  State = state.abb[match(as.character(State), state.name)],
  fillKey = cut(Crime, quantile(Crime, seq(0, 1, 1/5)), labels = LETTERS[1:5]),
  Year = as.numeric(substr(Year, 1, 4))
))
kable(head(datm2), format = 'html')
```

<table class="table table-striped table-condensed non-fluid">
 <thead>
  <tr>
   <th> Year </th>
   <th> State </th>
   <th> Crime </th>
   <th> fillKey </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td> 2010 </td>
   <td> AL </td>
   <td> 377.8 </td>
   <td> C </td>
  </tr>
  <tr>
   <td> 2009 </td>
   <td> AL </td>
   <td> 450.1 </td>
   <td> C </td>
  </tr>
  <tr>
   <td> 2008 </td>
   <td> AL </td>
   <td> 451.3 </td>
   <td> C </td>
  </tr>
  <tr>
   <td> 2007 </td>
   <td> AL </td>
   <td> 447.9 </td>
   <td> C </td>
  </tr>
  <tr>
   <td> 2006 </td>
   <td> AL </td>
   <td> 425.2 </td>
   <td> C </td>
  </tr>
  <tr>
   <td> 2005 </td>
   <td> AL </td>
   <td> 433.0 </td>
   <td> C </td>
  </tr>
</tbody>
</table>


---

## Choose Fill Colors



```r
library(RColorBrewer)
fills = as.list(setNames(rev(brewer.pal(4, 'PiYG')), LETTERS[1:4]))
```


---

## Create Payload for DataMaps


```r
library(plyr)
library(rCharts)
options(rcharts.cdn = TRUE)
dat2 <- dlply(datm2, "Year", function(x){
  y = toJSONArray2(x, json = F)
  names(y) = lapply(y, '[[', 'State')
  return(y)
})
```


---

## Create Simple Choropleth



```r
map <- Datamaps$new()
map$set(
  dom = 'chart_1',
  scope = 'usa',
  fills = fills,
  data = dat2[[1]],
  legend = TRUE,
  labels = TRUE
)
map
```

<iframe src='{{site.url}}/assets/images/simplechoro.html' scrolling='no' seamless
class='rChart datamaps '
id=iframe-
chart_1
></iframe>
<style>iframe.rChart{ width: 100%; height: 400px;}</style>


---

## Animated Choropleth


```r
map$set(
  bodyattrs = "ng-app ng-controller='rChartsCtrl'"
)
map$addAssets(
 jshead = "http://cdnjs.cloudflare.com/ajax/libs/angular.js/1.2.1/angular.min.js"
)
map$setTemplate(chartDiv = sprintf("
  <div class='container'>
    <input id='slider' type='range' min=1960 max=2010 ng-model='year' width=200>
    {{ year }}
    <div id='chart_1' class='rChart datamaps'></div>  
  </div>
  <script>
    var newData = %s
    function rChartsCtrl($scope){
      $scope.year = 1960;
      $scope.$watch('year', function(newYear){
        mapchart_1.updateChoropleth(newData[newYear]);
      })
    }
  </script>", rjson::toJSON(dat2)
))
map
```

<iframe src='{{site.url}}/assets/images/choro.html' scrolling='no' seamless class='rChart datamaps ' 
  id=iframe-chart_1></iframe>
<style>iframe.rChart{ width: 100%; height: 400px;}</style>


<style>
  iframe.rChart {height: 500px;} 
  ol.linenums {margin-left: -5px;}
  table.non-fluid {width: auto; margin: 0 auto;}
</style>
