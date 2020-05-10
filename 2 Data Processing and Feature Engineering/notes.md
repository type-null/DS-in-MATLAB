# Data Processing and Feature Engineering

Before you start, download [the code files and readings](https://www.mathworks.com/supportfiles/practicaldsmatlab/flights/Data%20Processing%20and%20Feature%20Engineering.zip) and [data files](https://www.mathworks.com/supportfiles/practicaldsmatlab/flights/Flights.zip).

Open MATLAB, navigate the Current Folder. Right click on the folder and select `Add to Path` -> `Selected Folders and Subfolders`.

__Important!__ In the MATLAB Command Window type 
```matlab
savepath
```
This will ensure the data files are always accessible every time you start MATLAB.

## 1 Survey Data

### Flights Dataset

- 2015 data from the US Department of Transportation's Airline On-time Performance Data Set, specifically the [Bureau of Transportation Statistics](https://www.transtats.bts.gov/homepage.asp).

- To import the data for January 
```matlab
flights = importFlightsData("flightsJan.csv")
```

- The departure time is in the time zone for the origin airport, whereas the arrival time is in the time zone for the destination airport.

![variables relationship](https://d3c33hcgiwev3.cloudfront.net/imageAssetProxy.v1/LA4Gfp92SkCOBn6fdspArg_aa226070cad5140f55bd2102b551dbdc_Flight-time-diagram.png?expiry=1589068800000&hmac=9-AH07HuRJBHHqx5lk9MRuZOqt19mvvRGx2QZC5C3JI)


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

  - Parallel coordinate plot: `parallelplot`

    - Single plot with multiple-axes to show the path of observations


## 2 Organize Data

## 3 Clean Data

## 4 Find Features that Matter

## 5 Domain Specific Feature Engineering

