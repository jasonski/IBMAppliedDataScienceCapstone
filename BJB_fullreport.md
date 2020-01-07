# Coursera Capstone Project Report
Capstone project : Battle of the Neighbourhoods - Burger Joint Berlin - BJB

## 1. Introduction

This report is written as part of the IBM Data Science class on *coursera.org* in frame of the Applied Data Science Capstone. It deals with the following, simple question:

**Where should a possible investor open a new burger joint in the city of Berlin, Germany?**

Such a question might be of interest for either *entrepreneurs* that plan on opening such a place or for *investors* that want to open a new place or invest in an existing place that meet the here derived criteria. Either way a thorough analysis of preexisting location factors is practically *mandatory for a successful restaurant opening*.

We will restrict the analysis on some of the most significant location factors for finding the best areas in Berlin. We will look on the possible costumer side as well as on the competition aspect. The costumer side will be inferred from Berlin demographics, from tourist number proxies and from office locations. While burger joints have not been common in Berlin for a long time, the last decade has brought many burger places to Berlin and competition has risen. These places, however, are not distributed equally and many attractive locations remain available. We will therefore look in detail on the locations of existing burger joints, representing the market competition. 

We will take a clusterization approach to classify different areas in Berlin based on the above factors. We will perform the analysis on the  zip code level and try to find the best zip code areas for the new place to open.    

For the analysis we will use publicly available census data for Berlin  for the demographic component and we will perform some FouresquareAPI queries to obtain all other data such as hotel locations, office locations  and burger joint locations.    

The resulting deliverable of the effort taken here will be a list of zip codes classified by their potential to successfully open a new burger place.

## 2. Data
### 2.1. Demographic data Berlin, Germany

Germany had its last census in 2011 (www.zensus2011.de). The data can be accessed there directily. Conveniently, the web page 
www.suche-postleitzahl.org reworked the data and provides a list of the amount of inhabitants per zip code:

https://www.suche-postleitzahl.org/download_files/public/plz_einwohner.csv

*data example:*

| zip	| citizens|
|------|--------|
| 10115| 20313 |
| 10117| 12217 |
| 10119| 16363 |
| 10178| 12167 |
| 10179| 18664 |
| 10243| ... |

The csv file directly provides the number of citizens for each zip area code.

### 2.2. Location data Berlin, Germany

Next, we need the location and shape of each zip code. This is also provided by the same web page in shp or kml file format. https://www.suche-postleitzahl.org/berlin.13f

*data example:*
```kml
<kml xmlns="http://www.opengis.net/kml/2.2"><Document><Folder><name>PLZ 5-stellig Stadt Berlin</name><Placemark><name>10115</name><description>10115 Berlin Mitte</description><Polygon><outerBoundaryIs><LinearRing><coordinates>13.3658603,52.535660399999998 13.366558899999999,52.5359129 13.3667012,52.5360631 13.3678819,52.5366213 13.368609299999999, ... ,52.535660399999998</coordinates></LinearRing></outerBoundaryIs></Polygon></Placemark><Placemark> ...
```
These are **GPS coordinates** which serve as the corner points of the polygone of the **zip code area** 10115 Berlin. Here an exemplary image of that zip code:

![Zip 10115](/images/10115.png?)

We download the list of coordinates for all Berlin zip codes.

### 2.3. Categorical Data provided by Foursquare

In order to obtain information on tourist numbers we will use the proxy hotel locaions. Also we seek office locations and burger joint locations. To gather these information we utillize the location platform of **Foursquare Labs Inc.** using a free developer account. We will use the *venue search query* to access the foursquare data base using their API. This returns a list of venues near a given location, optionally matching a search term. We can specify our three categories hotels, offices and burger joints.

Documentation on how to use this query is found here:
https://developer.foursquare.com/docs/api/venues/search

The FoursquareAPI response provides much more data than needed here. We will only make use of the following parameters:
name, latitude, longitude, zip code, and id as we are mainly intereested in location and number of these places.

An exemplary response for a joint location, as provided by foursquare,  limited to only one answer, and reshaped into a python pandas dataframe would look linke this:

|name |	lat |	lon |	zip 	|id|
|----|----|----|----|----|
|Kladow Grill Burger Pizza |	52.453026 |	13.141852 |	14089 |	4f92ddb6e4b008256552e140|

A normal request will give a list of up to 50 results (for a free developer account) and we will have to pose multiple request to cover all Berlin zip codes as well as all above named categories.

## 3. Methodology
As explained the data sets used will be 1) the citizencount per zip code and 2) numbers of burger places, hotels and offices.  
### 3.1 Data Preparation
The data sets need to be cleaned and transformed into a handy format for the analysis. For this task the python library [pandas](https://pandas.pydata.org/). is a great tool The data sets are transformed into pandas dataframe format. Here are the 5 rows of the dataframes burger joints, hotels and offices. 

<table class="dataframe" >
  <thead>
    <tr style="text-align:right">
      <th></th>
      <th>name</th>
      <th>lat</th>
      <th>lon</th>
      <th>zip</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Kladow Grill Burger Pizza</td>
      <td>52.453026</td>
      <td>13.141852</td>
      <td>14089</td>
      <td>4f92ddb6e4b008256552e140</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Volcano Burger</td>
      <td>52.525032</td>
      <td>13.196861</td>
      <td>13595</td>
      <td>594d55f69d6a19266e4509f0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Cruise-In</td>
      <td>52.532584</td>
      <td>13.178774</td>
      <td>13581</td>
      <td>4bf2815baf659c744ffcd747</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bastis Currys, Burgers &amp; Fries</td>
      <td>52.533400</td>
      <td>13.190330</td>
      <td>13581</td>
      <td>5a17e311bfc6d03f268daa13</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Burger Route</td>
      <td>52.530632</td>
      <td>13.196140</td>
      <td>13581</td>
      <td>5921ba9c89e49063753304e5</td>
    </tr>
  </tbody>
</table>

<table class="dataframe" >
  <thead>
    <tr style="text-align:right">
      <th></th>
      <th>name</th>
      <th>lat</th>
      <th>lon</th>
      <th>zip</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Breakfast Room</td>
      <td>52.399327</td>
      <td>13.112184</td>
      <td>NaN</td>
      <td>5dd0f776ed8bfb0008fd2cf8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Concorde Hotel Forsthaus</td>
      <td>52.405273</td>
      <td>13.140936</td>
      <td>14109</td>
      <td>4ea0073e8b816206b9f66786</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Pension Zeitlos</td>
      <td>52.392231</td>
      <td>13.098609</td>
      <td>14482</td>
      <td>4bc793c193bdeee1538837ae</td>
    </tr>
    <tr>
      <th>3</th>
      <td>avendi Hotel am Griebnitzsee</td>
      <td>52.395366</td>
      <td>13.127460</td>
      <td>14482</td>
      <td>4b0b2b48f964a520fc2d23e3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Apartements Filmeck</td>
      <td>52.382681</td>
      <td>13.122606</td>
      <td>14480</td>
      <td>57a37946498e72a672128ef4</td>
    </tr>
  </tbody>
</table>

<table class="dataframe" >
  <thead>
    <tr style="text-align:right">
      <th></th>
      <th>name</th>
      <th>lat</th>
      <th>lon</th>
      <th>zip</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>MedienHaus Babelsberg - Zentrum für Film- und ...</td>
      <td>52.387894</td>
      <td>13.119310</td>
      <td>14482</td>
      <td>506daff2e4b0dd5636405aae</td>
    </tr>
    <tr>
      <th>1</th>
      <td>VCAT Consulting HQ</td>
      <td>52.388084</td>
      <td>13.119418</td>
      <td>14482</td>
      <td>4bdae2aac79cc9286c1880e9</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Vragments</td>
      <td>52.388901</td>
      <td>13.120517</td>
      <td>14482</td>
      <td>5b71911df2554e002c9e3f9b</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Point Cloud Technology GmbH</td>
      <td>52.387762</td>
      <td>13.121250</td>
      <td>14482</td>
      <td>5cc6ba94c0af57002cd0a6ac</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Radio TEDDY  HQ</td>
      <td>52.382264</td>
      <td>13.120453</td>
      <td>14482</td>
      <td>4b90c0a2f964a520419633e3</td>
    </tr>
  </tbody>
</table>

Some rows miss zip code values in all data frames. We have to replace them. Simultaneously the data frames need to be cleaned  from places that are not in Berlin and wich were accidentally added due to the implementation of the data acquisition (see jupyter notebook). To achieve this the latitude and longitude of each location is compared with the zip code polygones provided in the geojson file. To check if a given point is wihtin a given polygone the python package [shapely](https://pypi.org/project/Shapely/) with its methods Point and Polygon is utillized. 

In the cleaned data frames it is now possible to count facilities of each type for each zip code. The result is joined with the demographic data on the basis of their zip codes and the resulting data frame constitutes the data base for the following statistical anylsis. The first rows can be seen in the following table.
<table class="dataframe">
  <thead>
    <tr style="text-align:right">
      <th></th>
      <th>zip</th>
      <th>ct_count</th>
      <th>bj_count</th>
      <th>of_count</th>
      <th>ht_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10115</td>
      <td>20313</td>
      <td>2.0</td>
      <td>12.0</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10117</td>
      <td>12217</td>
      <td>5.0</td>
      <td>14.0</td>
      <td>21.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10119</td>
      <td>16363</td>
      <td>7.0</td>
      <td>15.0</td>
      <td>11.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10178</td>
      <td>12167</td>
      <td>7.0</td>
      <td>15.0</td>
      <td>14.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10179</td>
      <td>18664</td>
      <td>2.0</td>
      <td>11.0</td>
      <td>12.0</td>
    </tr>
  </tbody>
</table>

### 3.2 Data Analysis using Machine Learning 

## 4. Results

## 5. Discussion

## 6. Conslusion



