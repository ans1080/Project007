## Project 007 Write-Up
## Column Documentation
```
By: Andy Snitgen
For: Professor Karen Jin
Class: comp574 - Applied Computing II
Date: October 31, 2021
```

### Title
```
The data stored in the title column is the name of the film.
```
* Type: String
* Source: API Call

### Year
```
The data stored in the year column is the year in which the film released.
```
* Type: String
* Source: API Call

### Actor
```
The data stored in the actor column is the name of the person who portrayed the main character, James Bond, in a particular movie.  It represents the leading role of the film.
```
* Type: String
* Source: API Call

### US Box Office
```
The data stored in the US Box Office column is the amount, in United States Dollars (USD), that film grossed in its theatrical run in America.  It represents the value of all the tickets sold in the United States.
```
* Type: String
    * floater(input_str) converts to float data type
* Source: API Call

### International Revenue
```
The data stored in the International column is the sum, in USD of the foreign and domestic box markets for an individual movie.  Essentially, it is the worldwide gross revenue of the film.  It represents the total value of all theatrical tickets sold on Earth.
```
* Type: String
    * floater(input_str) converts to float data type
* Source: API Call / Interpolated

### Adjusted Revenue
```
The data stored in the Adjusted Revenue column is the international revenue data, converted into 2021 USD.  The coefficient of inflation is determined with the web-scraped CPI data and applied to the total worldwide revenue data to produce the correct, inflation-adjusted values.  This represents the amount of money the film would have made were it released this year.
```
* Type: String
    * floater(input_str) converts to float data type
* Source: Calculated / Web-Scraped

### Average Rating
```
The data stored in the Average Rating column is sum of a film's IMDb Rating and Metacritic values divided by two.  It reprsents the mean of the Metacritic score and IMDb user rating for each movie.
```
* Type: String
* Source: Calculated

### IMDb Rating
```
The data stored in the IMDb Rating column is each film's average user score on a 10-point scale.  It represents the mean value of all IMDb member's review scores for the film in question.  It is displayed in ## / 10 format, with 10 being the highest and 1 being the lowest.  
```
* Type: String
    * replace_rating(input_str) converts to percentage of 100% and returns a float data type
* Source: API Call / 1 Interpolated data element

### Metacritic
```
The data stored in the Metacritic column is the mean of the aggregate score of many published critical reviews.  It is returned as a percentage ranging from 0% to 100% and represents the average score of all professional reviewer's scores.  
```
* Type: String
    * replace_critic converts to float data type