## Project 007 Write-Up
## Data Collection
```
By: Andy Snitgen
For: Professor Karen Jin
Class: comp574 - Applied Computing II
Date: October 31, 2021
```

### API Collection
```
The data used in the 007 project primarily comes from Internet Movie Database and is accessed through their API. IMDb requires an email to get the necessary key.  The program uses a key that is linked to my student email.  While IMDb allows for 100 free API calls per day, the key in project007.ipynb will provide 5,000 calls per day until November 23rd, 2021.  If the project requires more than cursory usage after that date, I will likely purchase another month.  Their documentation warns against using multiple emails to obtain more keys, and thus free calls.  In fact, it suggests that the practice could result in an IP ban, and I would like to avoid that.  It is an important fact to consider because the code within project007.ipynb requires 54 to 58 API calls per successful run.  That is the equivalent of only a single use per day. 

The small variance in number of API calls is due to an algorithm that detects if the API contains box office data for No Time to Die. That is the newest James Bond movie, and it is currently in theaters. It opened in the UK on September 30th and in the USA on October 8th, 2021.  This is interesting to note, because the API contains data for the international market, but returns only a blank string for the US.  While I could have scrapped the data from another website, I noticed IMDb's own webpage has the data.  So, I wrote a function called checker that ascertains whether that information has been placed in the API.  If the checker function returns a false value, the program instead inserts the accurate data that I hardcoded into the program in place of the empty string.  I chose to hardcode it instead of pulling it from a different source to maintain consistent methodology.  In my experience, different sources can provide different information; I trust IMDb's data to be authentic and stuck with them throughout.  Ultimately I placed priority on providing an accurate and useful database for comparisons. 

Project 007 uses API calls in two places, both functions.  One of them is the aforementioned checker function, but usage primarily comes from the data function.  In fact, the checker just uses a shorter, more concise version of data's code.  In essence, checker pulls data only for No Time to Die's missing values to confirm that are, indeed, still missing. Below I will provide a brief description of how the data function works.  
```
* The data function requires input parameter: `movie_title`.
    * This variable is a string representation of the film title in question.  In project 007, the titles are pulled from a hardcoded list of all James Bond films.
* It then uses IMDb API's search function for the title and return the corresponding id number for the movie and save it as `id_num`.
* The ID number is then used in a second API call to produce a comprehensive list of information on the title.
* The relevant data is then pulled and saved to variables corresponding to columns in the project's pandas dataframe.
* There is significant curating of the information that takes place at this point.
    * International Box Office data represents a cumulative worldwide total in IMDb's database. However on many of the older films, the data is missing.
        * In this instance, IMDb uses the US Box office data as the international data, as it is the only country whose data available.
            * This leads to a significant statistical anomaly when compared with films that do a more comprehensive worldwide gross.
        * To compensate for this, project 007 filters out any international data that is within 1% of the US box office totals.
        * It then replaces the filtered values with a 'No Data' string.
        * The checker function, which has its own API calls, is used in this section of the code. 
    * The Metacritic scores are concatenated with a % symbol to accurately convey their meaning.
    * The IMDb user ratings are also concatenated, in their case with the string ' / 10' to denote they are a score out of ten.
    * In newer movies, such as No Time to Die, IMDb doesn't have enough user scores to publish a rating.  
        * In this case, the API returns a null value which the program changes to the Metacritic score divided by 10.
            * It will also concatenate a '(est.)' string to the end to show the data has been interpolated and is only an estimation.
* Once all the data has been curated, formatted and saved to corresponding variables, these variables are saved to a dictionary: `new_row`.
    * The keys in this dictionary are the column names in the dataframe, and the values are the variables derived from the API call.
* The dictionary, `new_row` is then returned to end the data function.

### Interpolated Data
```
Some values used in the dataframe are not drawn directly from the API, instead they entirely interpolated.  They are the best estimation I could produce given the concrete data the IMDb API provided.  I chose to use approximations because it allows for more useful comparisons than would otherwise be viable.  One example that has already been discussed is the case of a null value for IMDb user ratings.  In that case, the only other relevant data that is accessible is the Metacritic aggregate score.  With no other data points, the best approximation of a user rating is the critics.  So, the Metacritic score was simply converted to the IMDb rating format and appended with an estimation tag. While this only affects the recent No Time to Die, I thought it prudent to develop an algorithm capable of handling null edge cases in case other movies are added in the future. 

That single instance is not the only approximation in the program.  There is another, much more common case of interpolated data found in the international revenue column.  As discussed, when foreign box office data is unavailable or limited in scope, the international revenue will be equal or very close the US box office.  Specifically, if the international revenue is within 1% variance of the US box office I deemed the data to be missing and replaced the value with the string 'No Data'.  The missing data makes any comparison of total revenue between films invalid, so to compensate I devised an algorithm remedy the problem.  It can be found in the tenth code block of the project007 Jupyter notebook.  It contains to sections, headed by comments that are described with an abstract style of psuedocode below.
```
* The first segment of code calculates the variable `change`.  This variable is the coefficient derived from ratios of US box office to international revenue.
    * First, it creates a for loop that iterates through each movie in the dataframe.
        * For each film, it determines if data exists for both the US box office and international revenue.
            * If it does, the code uses the floater function to change the dollar value strings into float data types.
            * It then takes the ratio of international revenue to US box office and saves the result in running tally under the variable `total`.
    * When the loop is done, the program uses a value_counts() boolean to determine the number of movies that have data for both columns.  This result is saved as `counter` and is equal to the number of ratios the `total` variable is comprised of.
    * The segment ends by creating the `change` variable and setting it equal to `total` divided by `counter`.  This makes `change` the average of all the valid ratios, essentially the coefficient by which US box office must be multiplied by to determine the average international revenue.
* The second segment of code applies `change` to the US box office of films where there is no foreign box office data available.
    * It, again, begins with a for loop that iterates through all the movies.
        * And for each film it uses an if statement to isolate the films that have 'No Data' for international revenue.
            * It then uses the floater function to convert the US box office data into a data type suitable for mathematics.
            * Next, the program applies the coefficient to the US box office data and saves the result.
            * Finally, it formats the result into a string with a dollar sign, appropriate commas, and an estimation tag. Then it saves the result in the international revenue column for each indexed movie.

### Web-Scraping Data Collection
```
Even with all the missing data interpolated, comparisons between the financial data of different films is still of little worth.  This is because of inflation.  A dollar in 1962 was worth over 9 times the amount of a dollar in 2021.  It's clear that for a realistic analysis, inflation must be accounted for.  The easiest way to accomplish this task is with the consumer price index.  This is a value, known as cpi, is computed monthly and averaged yearly by the US Bureau of Labor Statistics.  It measures the average change overtime in the prices paid by consumers for goods and services.  Its extremely useful because it can be used to calculate the value of a dollar from one year, in any other year.  Specifically, the formula is the base price in the original year multiplied by the ratio of the new year's cpi divided by the original year's cpi: new amount = original amount * (new cpi/original cpi).  The application of this equation allows the program to easily convert the worldwide gross that is contained in the international revenue column into 2021 dollars.

Just because cpi is a useful statistic does not mean it is available on IMDb's API.  In this case, the data was found on the Federal Reserve Bank of Minneapolis web page.  The URL is as follows: https://www.minneapolisfed.org/about-us/monetary-policy/inflation-calculator/consumer-price-index-1913-.  A quick travel to that site will reveal a very simple table containing only the year, the annual average cpi for that year, and the rate of inflation for the same.  The python library beautiful soup, was used to return the html code of the document which was then scraped to produce a dictionary containing the year and it's corresponding cpi.  The code that does this is found in the 11th block of the project007.ipynb file.  The pseudocode for the segment can be found below.
```
* Use requests.get on the webpage.
* Use the html.parser function of beautiful soup to return the html code.
* Index into the `tbody` div in the html code.
* Within `tbody` use the find_all function to access each `td` entry and save the result in the variable `tables`.
* Create an empty list `info_list` to store the scraped data
* Create a for loop that iterates through each element, `elem` in `tables`.
    * For each element, use find_children and contents to return the values on the table and save them to variable `info`
    * append `info_list` with the `info` variable for each element
* Create two new empty lists `year` and `cpi`
* Create a for loop that iterates through the `info_list` by threes, starting at index 0 (This will iterate through all the year values)
    * Format the year data by replacing * characters with empty strings
    * Continue formatting by replacing unicode '\xa0' with ''.
    * Append the formatted year data to the `year` list.
* Create a for loop that iterates through the `info_list` by threes, starting at index 11 (This will iterate through all the cpi values)
    * Append the iterated data to the `cpi` list.

 ### Calculated Data
 ```
 Even with all the data derived from web-scraping and API calls, there was still some values that were obtained by other means.  In this final instance, data was calculated.  Specifically, the 'Adjusted Revenue' and 'Average Rating' columns.  The average rating is exactly what it sounds like; it is the sum of the IMDb user rating and the Metacritic score divided by 2.  However, both sets of data had to be converted from strings.  In addition, the user rating was multiplied by a factor of 10 to put both ratings on a uniform scale of 100.  More interesting is the adjusted revenue.  This value is derived from applying the formula described in the web-scraping section, (new amount = original amount * (new cpi/original cpi)), to the international revenue to produce an inflation adjusted total revenue for each movie.  With those final two calculated data columns, the project007 database can be considered complete.
 ```
 * A special note: many of the smaller for loops utilized would be excellent opportunities to make use of .apply function of pandas.  Unfortunately, I was unaware that apply could be used for anything except lambda functions until after I had already written the code.  If given the chance, it is possible I could update this for the second milestone.    

            