# Citi-Bike Trends

# Prompt
Using open data from Citi Bike ([Data Source](https://www.citibikenyc.com/system-data)) I wanted to explore any trends from that past few years. I was primarily insteresed in who was using the system, where they were going, and when they were going.

# Dependencies
- Tableau
- Jupyter Notebook
- Pandas
- Datetime
- Numpy
- Glob

# Process
1. Data Aquisiton and Processing

    The three things I was interested in examining were:
    - Ridership Trends over Time
    - Usage in 2020
    - Top Stations Used

    Each time a bike is rented it gets logged in citi bike database. On an approximate monthly cycle this data is published. To get a useable sample size I looked at the past four years. The data is held in a AWS S3 bucket so downloading it is easy enough. To make the data less resource hunger that monthly CSVs are combinted and the stats of interest are aggregated. 

    Since the files all follow the same naming convention glob can be used to combine then in the jupyter notebook. To ensure all the months get combined in the correct order the `glob()` function is passed through the `sorted()` function. 
    ```
    combined_csv = sorted(glob(f'data/{year}/*-citibike-tripdata.csv'))
    ```

    Once all the months are combined they get converted to a dataframe so that comlumns can be added or formatted as necessary. For this project I'll need columns such as the year, month, day, and gender. 

    To get a finalized dataframe to convert into a CSV a few other custom functions are needed. `countRiders(month, dataframe)` will use the numerical value associated with each month (i.e. January = 1) and the the dataframe for that year to count the riders along a few categories of interest. 
    ```
    df = dataframe.loc[dataframe['month'] == str(month)]
    d['Rider Total'] = len(df)
    d['Female Total'] = len(df.loc[df['gender'] == '2'])
    d['Male Total'] = len(df.loc[df['gender'] == '1'])
    d['Undisclosed Total'] = len(df.loc[df['gender'] == '0'])
    d['Annual Members'] = len(df.loc[df['usertype'] == 'Subscriber'])
    d['Non Members'] = len(df.loc[df['usertype'] == 'Customer'])
    ```
    In the end a diction in returned for use in the `riderPerMonth()` function. Which as the name indicates will aggregate the number of riders in each of the categories.

    The `riderPerMonth()` function works by using a while loop. The i variable act as the month number and the counts from the `countRiders()` function are all apeneded to a list. Once each month is counted 1 will be added to the i variable until all 12 months have been processed.

    One final function `getYearlyRiderCount(year)` ties everything together. Year and month are inserted into the dataframe so that we can check to see if the process worked.
    ```
    def getYearlyRiderCount(year):
        df = pd.DataFrame(ridersPerMonth(bikeDataToDF(f'{year}')))
        df.insert(0, 'Year', f'{year}')
        df.insert(1, 'Month', range(1, 1 + len(df)))
    
        return df
    ```
    <img src="images/ridercount-function.png" height="auto">
 
    The `to_csv()` function from pandas will allows convert the data frame into a file that can be used in Tableau.
    <img src="images/2020-riders.png" height="auto">

    Data Processing Summary for Ridership Over Time
      * Combine all the CSVs in a given year using `glob()`
      * Create a preliminary dataframe with rider counts for each month and time keeping columns
      * Create a final dataframe that get converted to a CSV for Tableau

    To get a more micro look a rider trends I took all the data for 2020 and preformed some further data engineering. Additional items is was interested in for 2020 was the most used stations and most used bikes by trip duration. Groupby was essential for both these categories. For the stations is allowed me to take all the trips that started and ended as a particular station and add that number to the dataframe as a column.
    ```
    start = data_2020.groupby('start station id')['start station id'].transform('count')
    stop = data_2020.groupby('end station id')['end station id'].transform('count')
    data_2020['total_usage'] = start + stop
    ```
    
    <img src="images/top-riders.png" height="auto">
    
    To avoid bogging down Tableau with a large file import, only the top 1000 most used bikes were used in the sample. The full set has 24,982 unique bike ids. The filtered CSV is around 89% smaller then the full dataset.
    
2. Tableau Charts

    
    
    
