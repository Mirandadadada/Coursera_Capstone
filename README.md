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
+ Description: This data contains 5 Borough and 306 Neighborhoods of New York city along with their latitude and longitude. This data is is crucial for mapping and exploring the different neighborhoods in New York City. Below is a sample of this data set:
![new york city data image](https://user-images.githubusercontent.com/67845270/87205315-27379c80-c2d5-11ea-8979-85e09f13e017.png)

### 2.2 Airbnb Listing data
+ Data Source: http://insideairbnb.com/get-the-data.html
+ Description: This data contains the entire Airbnb listings in New York city on 08 June, 2020. It provides the essential information of each listing including id, name, neighbourhood group, neighbourhood, latitude, longitude, room type, price, minimum nights, number of reviews, available days, and etc.. Those detailed information will help us to screen out the ones that match the searching criteria. And the geographical location of each listing will be linked with Foursquare data to find the surrounding venues for further analysis. Below is a sample of this data set:
![airbnb listing data image](https://user-images.githubusercontent.com/67845270/87207075-05401900-c2d9-11ea-9a94-6783b50bed37.png)

### 2.3 Foursquare Location data
+ Data Source: Foursquare API
+ Description: From Foursquare API, we can get the Manhattan Venue data which contains the top 100 venues that are in each Manhattan neighborhood within a radius of 500 meters. It will be filtered to get all the restaurants in each neighborhood which will be utilized for clustering purpose. Meanwhile, we can get Airbnb Listing Venue data which contains the top 100 venues that are around each Airbnb listing within a raidus of 500 meters. We can filter the Airbnb listings based on the number of venues around them.

### 2.5 Borough Boundaries data
+ Data Source: https://data.cityofnewyork.us/City-Government/Borough-Boundaries/tqmj-j8zm
+ Description: This json file contains the geographic data of 5 borough boundaries. It will be paired with other data set to create a choropleth map of New York City for data visualization.

