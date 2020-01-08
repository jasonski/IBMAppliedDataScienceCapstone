# Coursera Capstone Project Report
Battle of the Neighbourhoods - Burger Joint Berlin? - BJB

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

Germany had its last census in 2011 (www.zensus2011.de). The data can be accessed there directly. Conveniently, the web page 
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

*Table 1: Data example of citizen count for some zip codes in Berlin.*

The csv file directly provides the number of citizens for each zip area code.

### 2.2. Location data Berlin, Germany

Next, we need the location and shape of each zip code. This is also provided by the same web page in shp or kml file format. https://www.suche-postleitzahl.org/berlin.13f

*data example:*
```kml
<kml xmlns="http://www.opengis.net/kml/2.2"><Document><Folder><name>PLZ 5-stellig Stadt Berlin</name><Placemark><name>10115</name><description>10115 Berlin Mitte</description><Polygon><outerBoundaryIs><LinearRing><coordinates>13.3658603,52.535660399999998 13.366558899999999,52.5359129 13.3667012,52.5360631 13.3678819,52.5366213 13.368609299999999, ... ,52.535660399999998</coordinates></LinearRing></outerBoundaryIs></Polygon></Placemark><Placemark> ...
```
These are **GPS coordinates** which serve as the corner points of the polygon of the **zip code area** 10115 Berlin. Here an exemplary image of that zip code:

![Zip 10115](/images/10115.png?)<br>
*Figure 1: Exemplary zip code area 10115 in Berlin, Germany*

We download the list of coordinates for all Berlin zip codes.

### 2.3. Categorical Data provided by Foursquare

In order to obtain information on tourist numbers we will use the proxy hotel locations. Also, we seek office locations and burger joint locations. To gather those information we utilize the location platform of **Foursquare Labs Inc.** using a free developer account. We will use the *venue search query* to access the foursquare data base using their API. This returns a list of venues near a given location, optionally matching a search term. We can specify our three categories hotels, offices and burger joints.

Documentation on how to use this query is found here:
https://developer.foursquare.com/docs/api/venues/search

The FoursquareAPI response provides much more data than needed here. We will only make use of the following parameters:
name, latitude, longitude, zip code, and id as we are mainly interested in location and number of these places.

An exemplary response for a joint location, as provided by foursquare,  limited to only one answer, and reshaped into a python [pandas](https://pandas.pydata.org/) data frame would look link this:

|name |	lat |	lon |	zip 	|id|
|----|----|----|----|----|
|Kladow Grill Burger Pizza |	52.453026 |	13.141852 |	14089 |	4f92ddb6e4b008256552e140|

A normal request will give a list of up to 50 results (for a free developer account) and we will have to pose multiple request to cover all Berlin zip codes as well as all above named categories.

## 3. Methodology
As explained the data sets used will be 1) the citizen count per zip code and 2) numbers of burger places, hotels and offices.  
### 3.1 Data Preparation
The data sets need to be cleaned and transformed into a handy format for the analysis. For this task the python library [pandas](https://pandas.pydata.org/) is a great tool The data sets are transformed into pandas dataframe format. Here are five exemplary rows of each of the dataframes burger joints, hotels and offices. 

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

*Table 2: Burger Joint data frame*

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

*Table 3: Hotel data frame*

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

*Table 4: Office data frame*

Some rows miss zip code values in all data frames. We have to replace them. Simultaneously the data frames need to be cleaned  from places that are not in Berlin and which were accidentally added due to the implementation of the data acquisition (see jupyter notebook). To achieve this the latitude and longitude of each location is compared with the zip code polygons provided in the geojson file. To check if a given point is within a given polygon the python package [shapely](https://pypi.org/project/Shapely/) with its methods Point and Polygon is utilized. With the help with the python package [folium](https://pypi.org/project/folium/) the data set can be visualized in map form.

![map berlin_burgershotelsoffices](/images/berlin_burhotoff.png?)<br>
*Figure 2: Map of Berlin with color coded locations of Burger Places, Hotels, and Offices.*

In the cleaned data frames it is now possible to count facilities of each type for each zip code. The result is joined with the demographic data on the basis of their zip codes and the resulting data frame constitutes the data base for the following statistical analysis. The first rows can be seen in the following table.

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

*Table 5: Complete data frame comprising the four categories used in this study: citizens, burger places, hotels, and offices.*

For completeness the demographic data is visualized in the following figure. This is achieved by combining the geojason file and the data frame column citizen count using the python folium package.

![map berlin_citizens](/images/berlin_citizens.png?)<br>
*Figure 3: Choropleth map style folium map of Berlin zip code areas color coded by number of inhabitants.*

### 3.2 Data Analysis using Machine Learning 
After preparation the data can now be analyzed. In order to decide what technique is useful for the analysis a data visualization is often of great help. Here, the normalized numbers of each category against zip code is shown.

![data_explr1](/images/data_explr1.png?)<br>
*Figure 4: Normalized numbers  of citizens, hotels, offices and burger places for each zip code.*

Another possibility is the plot of one variable against another. So here is an example of the three categories citizens, hotels, and offices as functions of number of burger places. 

![data_explr2](/images/data_explr2.png?)<br>
*Figure 5:  Normalized numbers  of citizens, hotels, and offices against number of burger places for each zip code.*

The above examples demonstrate that there is no easy visual relation between the data categories like i.e. linear or polynomial. However, Figure 5 seems to show some separation of features i.e. areas where one color seems to dominate.  We therefore decide for a clusterization algorithm that will divide our data set into clusters. We chose the [kmeans++](https://en.wikipedia.org/wiki/K-means_clustering) approach which is implemented as a method in the python package [sklearn](https://scikit-learn.org/stable/).  The kmeans++ algorithm allows to choose the number of wanted clusters and of seeds for cluster initiation. After some testing we chose to use four clusters as sufficient because we have to understand the differences in the clusters afterwards to able to rank them as good for our cases or not. Cluster seeds are set to 500. Because the data ranges are very different in each category we work with normalized data. The normalization allows to manipulate the weighting of the categories for clusterization. We will weight the influence of citizens down to w_ct = 0.5 because the ordinary person in Berlin presumably is a once in a while burger consumer. On the other hand burger place competition is important. Thus, we double the weight of that parameter w_bj=2. 

## 4. Results
The kmeans++ clusterization works fine and we find four clusters that appear to represent and separate our data set well. We check this again by plotting different categories against one another with a color overlay corresponding to the clusters obtained by the kmeans++ approach.

![data_cluster1](/images/data_cluster1.png?)<br>
*Figure 6:  Office count against number of burger places for each zip code color coded by cluster number 0-3.*

When the result is plotted onto the Berlin map we can identify the different clusters by color. 

![map_cluster1](/images/map_cluster1.png?)<br>
*Figure 7:  Map of Berlin zip code areas color coded by cluster affiliation.*

Now the affiliation of the zip codes with the four clusters is made clear. To make use of the performed cluterization we still have to understand if there is an interpretable scheme behind the clusterization.

## 5. Discussion
In order to get a better grasp on the meaning of the clusterization, we average each category cluster wise and then normalize each category by dividing by the maximum value between clusters. We obtain a new data frame displayed in the following table. 

<table class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ct_count</th>
      <th>bj_count</th>
      <th>of_count</th>
      <th>ht_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.907749</td>
      <td>0.044835</td>
      <td>0.303245</td>
      <td>0.246536</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.930925</td>
      <td>0.394625</td>
      <td>0.422400</td>
      <td>0.401786</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.746667</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.925165</td>
      <td>0.101839</td>
      <td>1.000000</td>
      <td>0.776786</td>
    </tr>
  </tbody>
</table>

*Table 6: Data categories counted and normalized for each column.*

We see that the cluster wise average citizen count is very similar between clusters, while the other categories vary significantly stronger. The largest deviation is found for burger joints. To better visualize this the following figure is very useful.

![rose_cluster1](/images/rose_cluster1.png?)<br>
*Figure 8:  Cluster wise averaged categories burger joints, office count, hotel count and office count. The numbers are normalized by the maximum of each category. Cluster coloring is consistent with Figures 6+7.*

With the help of Figure 8 it is now possible to rank the clusters and thus answer our initial research question.

The best areas to open a new burger place in Berlin, Germany can be found in the **zip code areas of cluster (3)**. Here, we have an average number of citizens, the highest number of offices, a high number of hotels, i.e., many tourists and finally very low competing burger places.

A second option are the **areas of cluster (0)** where we find even less burger places, but also significantly fewer offices and tourist. In these zip code areas one might expect primarily residents as customers and should adjust to their needs.

The other two **categories (1) and (2)** show significantly more competition and less favorable conditions for opening the new burger place. However, if the competition can be overcome by an superior burger offering of any kind the areas in cluster (2) might be interesting as well.

## 6. Conclusion
We found a clear answer to the initial raised question where to open a new burger place in Berlin, Germany. We performed data acquisition and reshaping and then analyzed the data using the kmeans++ algorithm. In this way we obtained four clusters which we interpreted to be of very different quality for opening the new burger place. **The zip code areas from cluster (0) show the best factors and should be considered first.**

Note, that we have to make clear that the analysis presented here is based only on a very limited amount of data categories. It is obvious that many other factors will influence the success of a new restaurant. This analysis should therefore be considered as an exemplary work that could be extended easily by including more data such as the amount of rent to be paid, available places fore hire, customer income, proximity to touristic sights, ore else. 

To see the computations that produced the given figures check the following two jupyter notebooks:


