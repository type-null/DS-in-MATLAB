# Data Processing and Feature Engineering

| __Table of Contents__
| ----------------------
| 1. [Survey Data](#ch1)
|     - [Flights Dataset](#ch1.1)
|     - [Shape of Distribution](#ch1.2)
|     - [Visualizing Multi-Dimensional Data](#ch1.3)
| 2. [Organize Data](#ch2)
|     - [Strings](#ch2.1)
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

- Edit Text

  1. Structured text

  - erase(_textVariable_, _textToErase_)
  
  ```matlab
  damageTxt = replace(Damage, ["K", "M", "B"], ["e3", "e6", "e9"]);
  damage = double(damageTxt)
  ```

  2. Unstructured text

  - `contains()` -> _logical_




<a name="ch3"></a>
## 3 Clean Data



<a name="ch4"></a>
## 4 Find Features that Matter



<a name="ch5"></a>
## 5 Domain Specific Feature Engineering


