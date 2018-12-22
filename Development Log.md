# CCB - Closed Clubs Berlin

## About
Berlin is constantly transforming and changing, and so is its notorious electronic music & club scene. The idea is to visualise this evolution, by focusing on the locations & sounds of former night clubs in Berlin. One potential further use case for such a map could be creating a mash-up with Soundcloud/Spotify playlists and creating a guided tour of Berlin by reliving the sound of the past clubs that made Berlin the Mecca for electronic music & club culture worldwide.

## Getting data
After some browsing around, I found a suitable data sample from [ResidentAdvisor](https://www.residentadvisor.com), which keeps an own category of [closed/former club names & locations](https://www.residentadvisor.net/clubs.aspx?ai=34&status=closed). There are more than 200 club & bar locations listed as closed on ResidentAdvisor. The scraping was done via [Grepsr](https://app.grepsr.com/app/concierge/welcome), resulting in a JSON file like this:

```
{
   "Club_Name_Closed":"Bar25",
   "Cub_Address_Closed":"Holzmarktstrasse 25 Friedrichshain 10243 Berlin   Germany",
   "Club_Ranking_RA_Closed":"1,831",
   "Club_Description_Closed":"This venue has closed. Bar 25 was an infamous Berlin afterhours party shack. It closed in 2010."
},
```

The raw JSON output was then further formatted to create an input file to be used for geocoding. For this, [Codebeautify](https://codebeautify.org) and [JSON Formatter](https://jsonformatter.curiousconcept.com) was used.

The final format used for the geocoding was a CSV file with pipe delimiters:

```
req_id|searchText|country
1|Alt-Stralau 1-2, 10245 Berlin,|Germany
2|Stralauer Platz 29 Friedrichshain 10243 Berlin|Germany
3|Revalerstrasse 99 Friedrichshain 10245 Berlin|Germany
```

## Geocoding
Using the [HERE Batch Geocoder](https://developer.here.com/documentation/batch-geocoder/topics/quick-start.html), the list of clubs was then geocoded with lat/lon coordinates, e.g. for visualization on a map. For the requests and responses, I used [Postman](https://www.getpostman.com).



### POST a geocoding job
```
https://batch.geocoder.api.here.com/6.2/jobs/
&app_id=12345
&app_code=12345
&mailto=myemailaddress
&outdelim=|&outcols=displayLatitude,displayLongitude,locationLabel,houseNumber,street,district,city,postalCode,county,state,country
&outputcombined=false
&indelim=|
```
### PUT geocoding job into execution

```
https://batch.geocoder.api.here.com/6.2/jobs/ZN83GqYYvT9YtiI6dGuRExmTTZfauiLP?action=run
&app_id=12345
&app_code=12345
&mailto=myemailaddress
&outdelim=|&outcols=displayLatitude,displayLongitude,locationLabel,houseNumber,street,district,city,postalCode,county,state,country
&outputcombined=false
&indelim=|
```

### GET geocoding results
```
https://batch.geocoder.api.here.com/6.2/jobs/ZN83GqYYvT9YtiI6dGuRExmTTZfauiLP/result
?app_id=12345&app_code=12345
```


## Data Post-Processing

I scanned the geocoding results and eliminated duplicates. Duplicates are indicated in the geocoder results by a sequence length > 1 (which essentially just means that the query returned more than one geocoding result).

```
49|1|4|52.50958|13.42263|Köpenicker Straße 55, 10179 Berlin, Deutschland|55|Köpenicker Straße|Mitte|Berlin|10179|Berlin|Berlin|DEU
49|2|4|52.42322|13.5133|Köpenicker Straße 55, 12355 Berlin, Deutschland|55|Köpenicker Straße|Rudow|Berlin|12355|Berlin|Berlin|DEU
49|3|4|52.5008128|13.5619892|Köpenicker Straße 55, 12683 Berlin, Deutschland|55|Köpenicker Straße|Biesdorf|Berlin|12683|Berlin|Berlin|DEU
49|4|4|52.4234099|13.5423132|Köpenicker Straße 55, 12524 Berlin, Deutschland|55|Köpenicker Straße|Altglienicke|Berlin|12524|Berlin|Berlin|DEU
```


## Visualization via XYZ Studio

The cleaned up data set was converted from CSV to GeoJSON using a converter from [MyGeoData](https://mygeodata.cloud/converter/csv-to-geojson):

```
{
  "type": "Feature",
  "geometry": {
     "type": "Point",
     "coordinates":  [ 13.3887,52.51657 ]
  },
  "properties": {
  "recId":8,
  "club_ranking_ra_closed":492,
  "club_name_closed":"Cookies",
  "houseNumber":"158",
  "street":"Friedrichstra√üe",
  "district":"Mitte",
  "city":"Berlin",
  "postalCode":10117,
  "county":"Berlin",
  "state":"Berlin",
  "country":"DEU",
  "club_description_closed":"This venue has closed. Cookies is a club in East Berlin located on the corner of Friedrichstrasse and Unter Den Linden.
  }
}
```

The data can then be used further with different visualization tools:
+ There is a dedicated GeoJSON viewer (http://geojson.tools) by HERE which can be used to further work with the data.
+ [XYZ Studio](https://xyz.here.com/studio) by HERE is another tool for creating custom maps from data sets.

Example visualization in XYZ Studio
![XYZ Studio Visualization](https://share.icloud.com/photos/0WYXGTsUkFbKWkl4sU6Xr1f3A#Haar)

The actual interactive map can be found here: https://xyz.here.com/viewer/?project_id=6be065fd-d4e5-4abc-bcce-59baaa76db61

# Next Steps

+ Fix geocoding  errors & outliers
+ Fix special characters (new GeoJSON from master file)
+ Get a clean result list with club names, location & additional information (closing date, wikipedia entry, soundcloud & spotify results, size, residents)
+ Convert dataset into suitable database format
+ Automate the conversion chain (scraping, formatting, geocoding, conversion,...)
+ Get a similar data set for club openings over time
+ Write a look-up for suitable spotify & soundcloud playlists for the clubs
+ Create a mobile app for location-based music discovery
+ Playlists are triggered based on proximity, and seamlessly weave into each other as you discover the city
+ Discovery similar to a radio frequency knob, discover clubs by turning around and picking up signals (volume/clarity depends on distance)

# Tools & Useful links
+ [XYZ Studio](https://xyz.here.com/studio) HERE tool for creating custom maps from data sets.
+ [GeoJSON viewer](http://geojson.tools) HERE tool specicially designed to work with GeoJSON data sets
+ [MyGeoData](https://mygeodata.cloud/converter/csv-to-geojson) conversion tool from CSV to GeoJSON
+ [HERE Batch Geocoder](https://developer.here.com/documentation/batch-geocoder/topics/quick-start.html) HERE REST API for Geocoding
+ [Postman](https://www.getpostman.com) API development & testing environment
+ [Codebeautify](https://codebeautify.org) Conversion tool for many different languages
+ [JSON Formatter](https://jsonformatter.curiousconcept.com) little tool for formatting JSON data
+ [Grepsr](https://app.grepsr.com/app/concierge/welcome) Scraper for web content
+ [closed/former club names & locations](https://www.residentadvisor.net/clubs.aspx?ai=34&status=closed) Data set from resident advisor
