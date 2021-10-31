## Project 007 Write-Up
```
By: Andy Snitgen
For: Professor Karen Jin
Class: comp574 - Applied Computing II
Date: October 31, 2021
```

### API Collection
```
The data used in the 007 project primarily comes from Internet Movie Database, and is accessed through their API. IMDb requires an email to get the necessary key.  The program uses a key that is linked to my student email.  While IMDb allows for 100 free API calls per day, the key in project007.ipynb will provide 5,000 calls per day until November 23rd, 2021.  If the project requires more then cursory usage after that date, I will likely purchase another month.  Their documentation warns against using multiple emails to obtain more keys, and thus free calls.  In fact, it suggests that the practice could result in an IP ban and I would like to avoid that.  It is an imporant fact to consider, because the code within project007.ipynb requires 54 to 58 API calls per successful run.  That is the equivalent of only a single use per day. 

The small variance in number of API calls is due to an algorythm that detects if the API contains box office data for No Time to Die. That is the newest James Bond movie and it is currently in theaters. It opening in the UK on Semptember 30th and in the USA on October 8th, 2021.  This is interesting to note, because the API contains data for the international market, but returns only a blank string for the US.  While I could have scrapped the data from another website, I noticed IMDb's own webpage has the data.  So, I wrote an function called checker that assertains whether that information has been placed in the API.  If the checker function returns a false value, the program instead inserts the accurate data that I hardcoded into the program in place of the empty string.  I chose to hardcode it instead of pull it from a different source to maintain consistent methodology.  In my experience, different sources can provide different information; I trust IMDb's data to be authentic, and stuck with them throughout.  Ultimately I placed priority on providing an accurate and useful database for comparisons. 

Project 007 uses API calls in two places, both functions.  One of them is the aforementioned checker function, but usage primarily comes from the data function.  In fact, the checker just uses a shorter, more concise version of data's code.  In essence, checker pulls data only for No Time to Die's missing values to confirm that are, indeed, still missing. Below I will provide a brief description of how the data function works.  
```
* The data function requires input parameter: `movie_title`.
    * This variable is a string representation of the film title in question.  In project 007, the titles are pulled from a hardcoded list of all James Bond films.
* It then uses IMDb API's search function for the title and return the corresponding id number for the movie and save it as `id_num`.
* The ID number is then used in a second API call to produce a comprehensive list of information on the title.
* The relevant data is then pulled and saved to variables corresponding to columns in the projects pandas's dataframe.
* There is significant currating of the information that takes place at this point.
    * International Box Office data represents a cumulative worldwide total in IMDb's database. However on many of the older films, the data is missing.
        * In this instance, IMDb uses the US Box office data as the international data, as it is the only country whose data available.
            * This leads to a significant statistical anomaly when compared with films that do a more comprehensive worldwide gross.
        * To compensate for this, project 007 filters out any international data that is within 1% of the US box office totals.
        * It then replaces the filtered values with a 'No Data' string.
        * The checker function, which has its own API calls, is used in this section of the code. 
    * The Metacritic scores are concatenated with a % symbol to accurately convey their meaning.
    * The IMDb user ratings are also concatenated, in their case with the string ' / 10' to denote they are a score out of ten.
    * In newer movies, such as No Time to Die, IMDb doesn't have enough user scores to publish a rating.  
        * In this case, the API returns a null value which the program changes to the metacritic score divided by 10.
            * It will also concatenate a '(est.)' string to the end to show the data has been interpolated and is only an estimation.
* Once all the data has been curated, formatted and saved to corresponding variables, these variables are saved to a dictionary: `new_row`.
    * The keys in this dictionary are the column names in the dataframe, and the values are the variables derived from the API call.
            