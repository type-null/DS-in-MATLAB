# Data Processing and Feature Engineering

| __Table of Contents__
| ----------------------
| 1. [Survey Data](#ch1)<br>- [Flights Dataset](#ch1.1)<br>- [Shape of Distribution](#ch1.2)<br>- [Visualizing Multi-Dimensional Data](#ch1.3)
| 2. [Organize Data](#ch2)<br>- [Strings](#ch2.1)<br>- [Dates and Times](#ch2.2)<br>- [Combine Data](#ch2.3)<br>- [Rearrange Data](#ch2.4)
| 3. [Clean Data](#ch3)
| 4. [Find Features](#ch4)
| 5. [Domain Specific Feature Engineering](#ch5)


Before you start, download [the code files and readings](https://www.mathworks.com/supportfiles/practicaldsmatlab/flights/Data%20Processing%20and%20Feature%20Engineering.zip) and [data files](https://www.mathworks.com/supportfiles/practicaldsmatlab/flights/Flights.zip).

Open MATLAB, navigate the Current Folder. Right click on the folder and select `Add to Path` -> `Selected Folders and Subfolders`.

__Important!__ In the MATLAB Command Window type 
```matlab
savepath
```
This will ensure the data files are always accessible every time you start MATLAB.

<a name="ch1"></a>
## 1 Survey Data

<a name="ch1.1"></a>
### Flights Dataset

- 2015 data from the US Department of Transportation's Airline On-time Performance Data Set, specifically the [Bureau of Transportation Statistics](https://www.transtats.bts.gov/homepage.asp).

- To import the data for January 
```matlab
flights = importFlightsData("flightsJan.csv")
```

- The departure time is in the time zone for the origin airport, whereas the arrival time is in the time zone for the destination airport.

![variables relationship](https://i.imgur.com/ivz2qti.png)

<a name="ch1.2"></a>
### Shape of Distribution

- `mean()`, `std()`

- __Skewness:__ Left skewed (< 0, most data above the mean, median close to Q3), Right skewed (> 0)

```matlab
sk = skewness(rmmissing(flights.DURATION_DIFF))
```
`sk = 1.8145`

  - kurtosis(Normal dstn) = 3. kt increases as the tail becomes heavier (more outlier-prone but no direction).

  ```matlab
  kt = kurtosis(rmmissing(flights.DURATION_DIFF))
  ```
  `kt = 14.3533`

- __Interquartile Range (IQR):__ Middle 50% of data = $$Q3 - Q1$$

  - Box plot: to see skewness

  ```matlab
  % group boxplot
  boxplot(flights.DURATION_DIFF, flights.ORIGIN)
  ```

<a name="ch1.3"></a>
### Visualizing Multi-Dimensional Data

- 2-D histogram `histogram2()`

- Multivariate Normal

```matlab
bins = 10 % # of bins
binscatter(X(:,1),X(:,2),bins);
```

Like 2-D histograms, but visualized using box color instead of bar height.

- `scatterhistogram()`

  - Adding a group variable to scatter histogram to see the potential relationships

- `heatmap()`

  - Use `'ColorVariable'` to visualize group statistics from continuous variable calculated using `groupsummary()`.

- High Dimensional Datasets

  - Scatter plot matrix: `gplotmatrix()`

  - Parallel coordinate plot: `parallelplot()`

    - Single plot with multiple-axes to show the path of observations


<a name="ch2"></a>
## 2 Organize Data

<a name="ch2.1"></a>
### Strings

1. Structured text

  - erase(_textVariable_, _textToErase_)
  
  ```matlab
  damageTxt = replace(Damage, ["K", "M", "B"], ["e3", "e6", "e9"]);
  damage = double(damageTxt)
  ```

2. Unstructured text

  - `contains()` -> _logical_

  - `extractFileText` reads text file to `str` (need [Text Analytics Toolbox](https://www.mathworks.com/help/textanalytics/index.html?s_tid=CRUX_lftnav))

  ```matlab
  sonnets = extractFileText("sonnets.txt");
  % Split the file as there are two newline characters after each sonnet
  sonnetsChunks = split(sonnets,[newline newline]);

  % The Roman numerals used to number the sonnets are individual elements in
  % the array - remove them by specifying a minimum length
  sonnetChunkLengths = strlength(sonnetsChunks);
  sonnets = sonnetsChunks(sonnetChunkLengths >= 300);
  ```

  - Counting the occurrences of words

  ```matlab
  deathCounts = count(sonnets, ["death", "kill", "die", "dead"]);
  sonnetsWithDeath = nnz(deathCounts)
  totalDeath = sum(deathCounts)
  [maxDeath, deathSonnet] = max(deathCounts)
  sonnets(deathSonnet)
  ```

<a name="ch2.2"></a>
### Dates and Times

- `datetime()` only takes variable of the same types (string/numeric ...)
  ```matlab
  datetime(date,"InputFormat","yyyy-MM-dd HH:mm","Format","yyyy-MM-dd HH:mm") % specify input and display format
  ```
- Calculate time
  1. fixed duration
      - `duratoin(3,20,0)` >> 03:20:00 (3 hour 20 minute)
        - Also `seconds`, `minutes`, `hours`, `days`, or `years`
      - `time = minutes(250)` >> 250 min
      - `time.Format = hh:mm` >> 04:10
  2. variable duration
      - `calmonths(2)` >> 2 _calendar months_ 
        - one month can be 28, 29, 30, or 31 days long
        - Also `caldays`, `calweeks`, `calquarters`, and `calyears`

- Sequence of Datetime or Duration using `:`
  - Defualt step size is one calendar day
    ```matlab
    t1 = datetime(2013,11,1,8,0,0);
    t2 = datetime(2013,11,5,8,0,0);
    t = t1:t2
    ```
  - Specify step size
    ```matlab
    t = t1:caldays(2):t2
    t = t1:hours(18):t2
    ```
  - Duration sequence
    ```matlab
    d = 0:seconds(30):minutes(3)
    ```

```matlab
doc dates and time
```


<a name="ch2.3"></a>
### Combine Data

- Import __multiple__ data files using `datastore`
  ```matlab
  flightsDatastore = datastore("flights*.csv") % use '*' to match any strings
  flightsAll = readall(flightsDatastore)
  ```
    - To specify import function
      ```matlab
      % use 'UniformRead' to import as one table
      flightsDatastore = fileDatastore("flights*.csv","ReadFcn",@importFlightsData,"UniformRead",true)
      flightsAll = readall(flightsDatastore)
      ```
      - Also `imageDatastore`, `spreadsheetDatastore`

- 5 Combine Methods
  ![Five combine methods](https://i.imgur.com/h1mlnYH.png)

- Join Tables

<a name="ch2.4"></a>
### Rearrange Data

- `sortrows` takes (_TABLE,SORT VARIABLE,ORDER_)

- `topkrows` when there are ties
  ```matlab
  % determine 20 least busy airport
  % use delay time as secondary sort variable
  flights.delayed = flights.departure_delay > 15
  delays = groupsummary(flights,"origin","sum","delayed")
  bottom20 = topkrows(delays,20,["GroupCount","sum_delayed"],["ascend","descend"])
  % use innerjoin to get more info
  bottom20 = innerjoin(flights,bottom20)
  ```


<a name="ch3"></a>
## 3 Clean Data



<a name="ch4"></a>
## 4 Find Features that Matter



<a name="ch5"></a>
## 5 Domain Specific Feature Engineering


