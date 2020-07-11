# **Coursera Capstone Project**
## The Battle of Neighborhoods Final Project


## 1. Introduction
### 1.1 Background
New York City is the nation's largest short-term rental market, facilitated by hosting websites such as Airbnb, Booking.com and etc. According to data from analytics website AirDNA, New York City had over 38,000 Airbnb listings and generated over 800 million U.S. dollars as of June 2019. With such a variety of options, it is challenging but crucial for visitors to find the most suitable accommodation to optimize their visiting experience. In this project, I will leverage the data from Foursquare and Airbnb to help visitors to screen out the most desirable accommodation based on their preference.

### 1.2 Problem to be Solved
I will stay in New York for a day to explore the lower Manhattan area as much as possible, so I want to find an Airbnb in Manhattan based on my following criteria: 
+ Budget: between $150 and $500 per night
+ Room type: entire home or apartment
+ Reviews: at least 5 reviews before
+ Restaurants: the nearby restaurants should be similar to the ones in Manhattanville where I lived before
+ Venues: the more, the better
### 1.3 Potential Audience
The methodology and tools used in this project are helpful for any individual or family who wants to relocate to a new city to find the best accommodations based on their personal preference. Likewise, this approch is instrumental in seeking out the best location for companies that want to open a new business in a city. Meanwhile, the mapping and clustering techniques can also be used in comparing locations, cities, or neighborhood to assist stakeholders in making decisions.<br><br>

## 2. Data
### 2.1 New York City data
+ Data Source: https://cocl.us/new_york_dataset
+ Description: This data contains 5 Borough and 306 Neighborhoods of New York city along with their latitude and longitude. This data is is crucial for mapping and exploring the different borough and neighborhoods in New York City. We used it to create a 'Manhattan_data' dataframe which include all the neighborhoods in Manhattan borough and their corresponding geographical location
+ Processing: After downloading New York City data as in json file, we find that all the relevant data is in the _features_ key, so we create a new variable that includes this data. Then we transform the data into a _pandas_ dataframe for analysis purpose. Below is a sample of this dataframe:<br><br>

![new york city data image](https://user-images.githubusercontent.com/67845270/87205315-27379c80-c2d5-11ea-8979-85e09f13e017.png)


### 2.2 Airbnb Listing data
+ Data Source: http://insideairbnb.com/get-the-data.html
+ Description: This data contains the entire Airbnb listings in New York city on 08 June, 2020. It provides the essential information of each listing including id, name, neighbourhood group, neighbourhood, latitude, longitude, room type, price, minimum nights, number of reviews, available days, and etc.. Those detailed information will help us to screen out the ones that match the searching criteria. And the geographical location of each listing will be linked with Foursquare data to find the surrounding venues for further analysis. 
+ Processing: The Airbnb Listing data is very inclusive and well structured. Some of the features like reveiws_per_month and calculated_host_listings_count are relatively irrelevant in our case, so we removed those column to get a more succinct data set. For analysis purpose, we change the columns' name 'neighbourhood_group' to 'Borough' and 'neighbourhood' to 'Neighborhood'. Below is a sample of this dataframe:<br><br>
![airbnb listing data image](https://user-images.githubusercontent.com/67845270/87207075-05401900-c2d9-11ea-9a94-6783b50bed37.png)

### 2.3 Foursquare Location data
+ Data Source: Foursquare API
+ Description: From Foursquare API, we can get the Manhattan Venue data which contains the top 100 venues that are in each Manhattan neighborhood within a radius of 500 meters. It will be filtered to get all the restaurants in each neighborhood which will be utilized for clustering purpose. Meanwhile, we can get Airbnb Listing Venue data which contains the top 100 venues that are around each Airbnb listing within a raidus of 500 meters. We can filter the Airbnb listings based on the number of venues around them.
+ Processing: We write a function 'getNearbyVenues' to acquire and transform the venues data from Foursquare API to a _pandas_ dataframe. Then we will combine it with the 'Manhattan_data' to create a new datafrme 'manhattan_venues' which contains each venue's name, latitude, longitude, category, they neighborhood they are in, as well as the neighborhood's geographical location. Below is a sample of this dataframe:<br><br>
![manhattan venue data image](https://user-images.githubusercontent.com/67845270/87211866-a20ab280-c2e9-11ea-9dd0-acf47f6d2fbc.png)


### 2.4 Borough Boundaries data
+ Data Source: https://data.cityofnewyork.us/City-Government/Borough-Boundaries/tqmj-j8zm
+ Description: This json file contains the geographic data of 5 borough boundaries. It will be paired with other data set to create a choropleth map of New York City for data visualization.<br><br>

## 3. Exploratory Data Analysis
### 3.1 Exploring Airbnb Listing Data
In the Airbnb Listing data, there are 49,531 listing in the New York City on 08 June, 2020. So the first step is to filter this large data set based on the criteria stated in section 1.2. After comparing the criteria with the data set, we find that the first three criteria (budget, room type, reviews) can be fulfilled by simply filtering the Airbnb Listing data. In addition, there are some hidden conditions. Since I will only stay in Manhattan for one night, so the minimum nights to stay should be 1, the availability can not be zero, and the borough is Manhattan. After implementing the 6 conditions (3 explicit, 3 hidden) to filter the data, we have 292 Airbnb listings satisfy the requirements.
Then we can extract the name, host_id, borough, neighborhood, latitude, longitude, and price to form a new dataframe for further analysis. Below is a sample of this dataframe:<br><br>
![mht airbnb data image](https://user-images.githubusercontent.com/67845270/87213050-fd3fa380-c2ef-11ea-9c18-c3b406ef4a3b.png)

### 3.2 Acquiring Venue Data from Foursquare API
For fulfilling the fourth condition (the nearby restaurants should be similar to the ones in Manhattanville) stated in section 1.2, we need to analyze the restaurant similarity among all neighborhoods in Manhattan and find the ones that are most simliar to Manhattanville. The first step is to acquiring the venue data from Frousquare API. So we write a function 'getNearbyVenues' to get the top 100 venues that are in each neighborhood within a radius of 500 meters, and run the function on each neighborhood in Manhattan to form a dataframe below: <br><br>
![manhattan 100 venues](https://user-images.githubusercontent.com/67845270/87213467-362d4780-c2f3-11ea-9685-d7123486f979.png)

### 3.3 Processing the Manhattan Venue Dataframe
Since the _Venue_Category_ column in the table above contains only categorical data which is difficult to quantify, we apply one-hot encoding to transform all the categorical data into binary values 1 or 0. Then we group rows by neighborhood and take the mean of the frequency of occurrence of each category for analysis purpose. Since we only interested in restaurants in each neighborhood, we only select the columns with 'Restaurant' in its column name, and we have the following dataframe:<br><br>
![restaurant image](https://user-images.githubusercontent.com/67845270/87213854-0cc1eb00-c2f6-11ea-89c3-2bf49a07c2b8.png)

### 3.4 Clustering Manhattan Neighborhood
For identifying the neighborhoods which have the similar restaurants as the ones in Manhattanville, we will use K-Means Clustering which can identify unknown groups in complex data sets, so that the neighboorhoods with te similar restaurants will assigned to the same cluster. Therefore, the first step is to determine the optimal number of clusters into which the data maybe clustered. By plotting the number of clusters against inertia which computes the sum of squared distances of samples to their closest cluster center, we apply the Elbow method and choose the number of clusters to be 6. Then we input the restaurant data we processd above to run the k-means clustering to cluster the neighborhood into 6 clusters. After clustering, we can construct a new dataframe with borough, neighborhood, latitude, longitude, cluster labels, and all the restaurants, then we can visualize the clusters by using Folium map.
From the image below, we can see that the neighborhoods with the same color are in the same cluster, meaning they are likely to have the similar restaurants. Since we are focusing on the neighborhoods that are similars to Manhattanville, which is in cluster 4, we can find Washington Heights, Inwood, Hamilton Heights, East Village, Manhattan Valley, Tudor City are also in that clusters. If we compare these neighborhood with the 'Neighborhood' column in the filtered Airbnb listing data, we can further filter the data  and have 42 available Airbnb listing left.

![mht_clustering image](https://user-images.githubusercontent.com/67845270/87214418-4ac10e00-c2fa-11ea-8fad-9a0599fe1de9.png)

### 3.5 Creating a Airbnb recommendation list
Futhermore, since I want to explore the lower Manhattan area, only the Airbnb listings in East Village match all the criteria for now. So we further filter the 'Neighborhood' column in the Airbnb listing data and we have 32 listings left. Now, we can move on to the last criteria on the section 1.2. We will apply the 'getNearbyVenues' function again using the airbnb's host id, latitude, and longitude to acquire the top venues around each Airbnb listing, and we only keep the listings that have 100 venues around them. Finally, we will achieve a Airbnb recommendation list with 32 listings left which meet all the requirements.<br><br>
![recommendation list image](https://user-images.githubusercontent.com/67845270/87214913-e94f6e00-c2fe-11ea-959d-3d9004307f0d.png)
![final recommendation image](https://user-images.githubusercontent.com/67845270/87214932-1bf96680-c2ff-11ea-90ec-fe9096bedea1.png)<br><br>


## 4. Conclusion 
As I mentioned in the introduction, New York City with over 49,000 Airbnb listings is one of the biggest market for Airbnb. After clearfully examing the visitors requirements, we are able to filter this large data set and form a recommendation list with only 32 Airbnb listings. I believe the data processing, clustering, and mapping techniques can also be applied in solving other similar problems. For example, we can help individuals who want to relocate to find the best fitting apartment, and we can help companies to explore the best location to open new business. In addition, in our case, we only have handful criteria that we need to fulfill, but we can also implement more conditions to get a more streamlined recommendation list. I felt rewarded after put all the efforts and spending time to complete this project, and I feel more confident in pursuing a career in data science.
