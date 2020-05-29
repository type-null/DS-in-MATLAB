# Data Processing and Feature Engineering

| __Table of Contents__
| ----------------------
| 1. [Survey Data](#ch1)<br>- [Flights Dataset](#ch1.1)<br>- [Shape of Distribution](#ch1.2)<br>- [Visualizing Multi-Dimensional Data](#ch1.3)
| 2. [Organize Data](#ch2)<br>- [Strings](#ch2.1)<br>- [Dates and Times](#ch2.2)<br>- [Combine Data](#ch2.3)<br>- [Rearrange Data](#ch2.4)
| 3. [Clean Data](#ch3)<br>- [Missing Data](#ch3.1)<br>- [Outliers](#ch3.2)<br>- [Normalize Data](#ch3.3)
| 4. [Find Features](#ch4)<br>- [Feature Engineering](#ch4.1)<br>- [Unsupervised Learning](#ch4.2)<br>- [Feature Selection](#ch4.3)
| 5. [Domain Specific Feature Engineering](#ch5)<br>- [Process Signals](#ch5.1)<br>- [Process Images](#ch5.2)<br>- [Process Text](#ch5.3)


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

<p align="center">
 <img src="https://i.imgur.com/ivz2qti.png" alt="variables relationship" width=500/>
</p>

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
  - `eraseBetween`, `insertBefore`, `extractAfter`, etc

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

- `dateshift()` shifts time to start or end of a time unit -> good for groupsummary

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
<p align="center">
  <img src="https://i.imgur.com/h1mlnYH.png" width="700" alt="Five combine methods"/>
</p>

- Join Tables
  - Use `Task` button

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

<a name="ch3.1"></a>
### Missing Data
- Types: MAR (relate to other variable), MACR, MNAR (strong bias)

- `ismissing()`
    - Identify missing value
    <p align="center">
      <img src="https://i.imgur.com/A25chjZ.png" width="450" alt="missing type"/>
    </p>

- `standardizeMissing()` to change abnormal data to missing taking (A,_indicator_,"DataVariables",_vars_)

- Handle missing data
    - Domain knowledge
    - Use `'omitnan'` option to include only numeric values
    - MAR, MCAR -> Remove `rmmissing`
    - MNAR -> Replace `fillmissing(A,method)`
    - Task `Clean Missing Data` button in live script


<a name="ch3.2"></a>
### Outliers
- Popular outlier limits
    - 3 std (μ±3σ) -> Normal dstn
    - 1.5 IQR (Q1-1.5IQR, Q3+1.5IQR)
    - Median Absolute Deviation (med±3cMAD)
      - MAD = median(|x - median(x)|)
    - Moving Mean or Median -> transient spikes
      - Moving _median_ is more robust than _mean_

- `isoutlier`, `rmoutlier`

- __Streamline:__
    - `histogram` to see distribution
    - `trimmed = rmoutliers(A,method)` then `histogram(trimmed)`, _method_ can be `mean`, `quartiles`, `median`
    - Adjust amout removed using `'ThresholdFactor'` (takes a double)
    - Gain flexibility with extremely skewed data using `"percentiles"` (takes [ lowPercent , highPercent ])
    ```matlab
    lowPercent = 5; 
    highPercent = 95; 

    % use the above choices with the percentile outlier classification method
    [~,pLow,pHigh,~] = isoutlier(flights.DEPARTURE_DELAY,...
    "percentiles",[ lowPercent , highPercent ]);  


    histogram(flights.DEPARTURE_DELAY);  
    xlabel("Delay (minutes)"); 
    xline(pLow,"m"); xline(pHigh,"m"); 

    xlim([pLow - 20 , pHigh + 20]) % optional zooom in to better see bounds 
    ```

- `movmean` and `movmedian` methods require __one__ data point at each index
    - We can calculate summary statistics at each index
    ```matlab
    velStats = groupsummary(flights,"DISTANCE",["max","min","mean","median"],"AVE_VELOCITY");

    % Find outliers
    moveMethod = "movmean"; 
    windowLength = 500; 
    tFactor = 4; 

    [outlierIdx,thresholdLow,thresholdHigh] =  isoutlier(velStats.max_AVE_VELOCITY, ...
        moveMethod,windowLength,"ThresholdFactor",tFactor,"SamplePoints",velStats.DISTANCE);
    ```
    <p align="center">
      <img src="https://i.imgur.com/h4RugpX.png" width="400" alt="output plot"/>
    </p>

<a name="ch3.3"></a>
### Normalize Data
| Distribution | Normalize Method | Code|
| --- | --- | --- |
| Uniform dstn | scale data between 0 and 1 | `normalize(A,'range')` |
| Normal dstn | z-score (z = (x-μ)/σ) | `normalize(A)` default method is 'zscore' |
| Non-normal dstn | robust zscore: use MAD instead of std to scale the data | `normalize(A,"zscore","robust")` |


- Smoothing Data
  - Only used for indexed data (by time, etc)
  - Methods: moving average, local regression, robust methods, advanced types of regression
  - `Smoothing data` live editor task button 
  ![smoothing data task](https://i.imgur.com/lQhv5kR.png)


<a name="ch4"></a>
## 4 Find Features that Matter
<a name="ch4.1"></a>
### Feature Engineering
- Feature generation approaches:
  1. Transforming variables -> Apply equations
    - BMI
    - Use the `azimuth` function to calculate this angle from the latitude and longitude data for the origin and destination airports
      ```matlab
      flights.AZIMUTH = azimuth(flights.LATITUDE_ORIGIN,flights.LONGITUDE_ORIGIN,...
        flights.LATITUDE_DESTINATION,flights.LONGITUDE_DESTINATION);
      % Take the sine of the Azimuthal angle. The function, sind, takes degrees as input.
      flights.SIN_AZIMUTH = sind(flights.AZIMUTH);
      ```


  2. Discretization -> Split data into groups (ranges) using numeric value
    - Age range -> life stage (child/adult/senior) -> heatmap
    - datetimes -> day of month
      ```matlab
      flights.DAY_OF_MONTH = day(flights.SCHEDULED_DEPARTURE_TIME);
      ```
    - Discrete numbers
      ```matlab
      % While flights that are > 15 minutes late are counted as "Delayed"
      flights.DELAYED = discretize(flights.ARRIVAL_DELAY,[-inf,15,inf],"categorical",["On Time", "Delayed"]);
      ```


  3. Summarizing groups
    - Mean `Weight` on each age category -> join the origin table


- Qualitative tests for useful features
  - Have variance
  - Are not random
  - Are unique
  - Make sense -> domain knowledge



<a name="ch4.2"></a>
### Unsupervised Learning
Identifying patterns or groupings in your data.

- Clustering Algorithms
    ![Clustering Algorithms](https://i.imgur.com/LIRkPga.png)
    1. k-means
    - algorithm starts with k random points as centroids to decide remaining points most similar to which groups -> update centroid location -> update groups -> until distance between all points and their centroids reaches minimum
    ```matlab
    [idx,C] = kmeans([X Y], 2, "Replicates", 5); % k=2
    ```
    - use `"Replicates"` to use multiple starting points
    - `"Distance"` option: 
      ```matlab
      'sqeuclidean'  - Squared Euclidean distance (the default).
      'cityblock'    - Sum of absolute differences, a.k.a. L1 distance
      'cosine'       - One minus the cosine of the included angle
                       between points (treated as vectors).
      'correlation'  - One minus the sample correlation between points
                       (treated as sequences of values).
      'hamming'      - Percentage of bits that differ (only suitable
                       for binary data). 
      ```


    2. k-medoids (_median_)
    - `kmedoids()`


    3. spectral clustering
    - Groups wrap around each other
    - Know # of clusters: `idx = spectralcluster(X,3)`
    - Not Know: `dbscan(X, 1, 5)`
        - 1 (_Epsilon_): radius
        - 5 (_MINPTS_): minimum number of points
        - useful for high-dimensional data
        - identify outliers:
            - __Core__: points having at least _MINPTS_ within _Epsilon_
            - __Border__: not enough _MINPTS_ within _Epsilon_ but within other __core__'s _Epsilon_
            - __Noise__: other -> outliers
           

    4. gaussian mixture models
    - Overlapping distribution
    - First determine k clusters
    ```matlab
    % k = 2
    GMModel = fitgmdist([x y],2);
    idx = cluster(GMModel,[x y]);
    gscatter(x,y,idx)
    ```

- Evaluate clusters 
  1. Silhouette Plot
    ```matlab
    [silh5,h] = silhouette(X,idx5,"sqEuclidean");
    ```
    - Points that are well matched to their cluster and poorly matched to other clusters have a value of 1.
    - In general, larger silhouette values mean better cluster quality.
    ```matlab
    s5 = mean(silh5)
    ```

  2. `evalclusters` (automately determine k)
    ```matlab
    clustev = evalclusters(X,"kmeans","silhouette","KList",2:6)
    ```
    - Optimal # of clusters stored in `k = clustev.OptimalK`


<a name="ch4.3"></a>
### Feature Selection

#### Evaluating Features

1. Wrapper
    train -> eval -> select features
    __output__: Best features, best model

2. Embedded
    selection as part of training
    __output__: trained model that emphasize useful features

Wrapper and Embedded rely on model performance to guide feature selection. ([Next course](../3%20Predictive%20Modeling%20and%20Machine%20Learning))

3. Filter
    select before train
    1. Variance Thresholding
    _Goal:_ eliminate low-variance features
    scale features -> compute var -> select threshold `t` -> select features `var>t`
    - Elbow
    - for continuous variable
    - convert scaled variance to proportion 
        - to see what proportion of variance you lose by discarding a given feature
    - Principle Component Analysis can be used to create a new feature set where variance is concentrated in a small subset of features, making variance thresholding more straightforward to apply.

    2. Covariance and Correlation
    _Goal:_ eliminate cov=0
    - standardized corr: Pearson Correlation Coefficient
    - Spearman and Kendall corr are better for non-linear data
        - Spearman is also robust to
            1. outliers
            2. non-normal
            3. heteroscedastic
            4. discrete ordinal (ratings)
        - Kendall is preferable to Spearman (and Pearson) in situations where the dataset is small and/or there are likely to be many ties in rank due to identical data values.
    - Monotonic only
    - Generally, |corr| is divided into 3 level by [0.4 0.7]
    - For weak correlation values, or small datasets, it may also be desirable to obtain a measure of confidence that a correlation value is significantly different than 0.
    ```matlab
    [c,p]= corr(tbl.Weight,tbl.Acceleration) 
    ```
    - heatmap
    ```matlab
    type = "Pearson";
    C = corr(ordinalX,'type',type);
    C(C==1) = nan;
    heatmap(ordinalvars,ordinalvars,C,'ColorLimits',[-1 1],'Title',type+" Correlation");
    ```

    - For non-ordinal discrete variables, it is possible to use Spearman or Kendall correlation by converting each categorical variable into two or more logical features - one new feature for each category - then computing the correlation between each new feature and the response. "[one hot encoding](https://en.wikipedia.org/wiki/One-hot)"
    ```matlab
    [c,p]= corr(tbl.Mfg == "saab",tbl.Weight,'type','Kendall')
    ```


    3. Statistical Tests
    choose test -> choose probability threshold (α) -> compute _p_-value


[Apply Filter Methods](Module%204/ApplyingFilterMethods.mlx)
- Variance Thresholding for Continuous Features
- Evaluating Continuous Predictor-Response Pairs
  - corr
- Evaluating discrete predictor-response pairs
  - contingency table (cross-tablation): `crosstab` to see how 2 variable codistributed
  - actual and expected counts
  - chi-squared test
- Evaluating continuous-discrete feature pairs
  - boxplot
  - The F-statistic 
  - ANOVA


#### Dimensionality Reduction and PCA
Cost of feature reduction: __Projection error__
 - -> reduce projection error by introducing new axes (Principle component axes)
   - min projection error
   - max variance
   - -> new feature _P_
   - __Principle component scores__: distance from origin to projection

- PCA steps:
  1. Scale and mean-center the original features
  2. Compute the components, scores and variances
      ```matlab
      [P,S,V] = pca(X)
      ```
  3. Apply variance threshold
  4. Scores of the retained components become the new feature set

[Apply PCA](Module%204/ApplyingPCA.mlx)


<a name="ch5"></a>
## 5 Domain Specific Feature Engineering
<a name="ch5.1"></a>
### Processing Signals

#### Synchronizing Data with Timetables
- Use `'TimeStep'` to build timetable
- Interpolation: `retime()` specifies interpolation method and new time step (sample rate)
  - then `join` tables
  - or you can combine these two steps using `synchronize()`

#### Summary Statistics as Features
- Windowing
  ```matlab
  windowLength = 2.5;
  summaryStat = "mean"; % min, max
  summaryStandWalk = retime(StandWalk,"regular",summaryStat,"TimeStep",seconds(windowLength));
  ```
  -  Standard deviation is not one of the default options for retime, but you can refer to any function with an "@" symbol. So instead of a string like "std", you write `@std` (without quotation marks). 

- Fourier Transform
  - Indexed by time -> indexed by frequency
    - Foundamental frequency (1st)
    - Harmonics (rest)
    - spectral summary statistics
    ![statistics](https://i.imgur.com/qaex9Zn.png)
    - spectrum plot
    ```matlab
    % Plot the periodogram. The empty inputs [] specify that the default values should be used.
    periodogram(walkSignal,[],[],sampleRate);
    ```

[Practice using summary statistics as features.](Module%204/Signals/SummaryStatisticsAsFeatures.mlx)


#### Finding Peaks
- load audio
    ```matlab
    [maleData,maleSampleRate] = audioread("speech.wav");
    maleSpeech = timetable(maleData,'SampleRate',maleSampleRate,'VariableNames','Data')
    ```
- play audio
    ```matlab
    sound(maleData,maleSampleRate)
    ```

- identify frequencies find location of peaks in the spectrum
    - live editor task `Find Local Extrema`

<a name="ch5.2"></a>
### Processing Images
![process](https://i.imgur.com/do40Xpn.png)
__Preprocessing__ Sefment Images
- APPS > `Image Segmenter`
  1. load image
  2. Adjust image? -> 'yes'
  3. _CREATE MASK_ Threshold
      - Global Threshold -> Adaptive Threshold: compensate for uneven lumination
      - adjust Sensitivity
      - Show Binary
      - Create Mask
  4. _REFINE MASK_ Fill holes, Clear borders
  5. _REFINE MASK_ Morphology
      - Select operation: Open Mask
      - increase Radius
  6. Generate Function, save to workspace

- APPS > `Image Batch Processer`
  1. load images -> select function -> Process all

__Feature Engineering__ Get region properties: area, diameter
- APPS > `Image Region Analyzer`
  1. Choose Properties
  2. Filter (__Cleaning Features__)
  3. Export function

__Evaluate Features__ ([this file](module%205/Images/clusteringImages.mlx))
  1. Apply generated functions to all images
  2. Create table of results
  3. Perform `anova1` test
  4. Cluster observations

[Practice](module%205/Images/Practice%20with%20Images/PracticeWithImages.mlx)
- load image
  ```matlab
  stop = imread(filename);
  ```

<a name="ch5.3"></a>
### Processing Text
- Extract text from website
  ```matlab
  url = "https://www.mathworks.com/help/textanalytics";
  code = webread(url);
  str = extractHTMLText(code)
  ```
- Word count
  ```matlab
  tokenizedSonnets = tokenizedDocument(sonnets);

  figure;
  wordcloud(tokenizedSonnets);
  ```
- Quantify wordcount
  - Create matrix of counts `bagOfWords`
  ```matlab
  wordBagSonnets = bagOfWords(tokenizedSonnets)
  full( wordBagSonnets.Counts )

  % visualize
  figure;
  spy(wordBagSonnets.Counts)
  axis normal
  xlabel('Unique Word #')
  ylabel('Sonnet')
  
  % or
  clf;
  plot(sum(wordBagSonnets.Counts))
  xlabel('Unique Word #')
  ylabel('Total Appearances')

  % top k words
  top10words = topkwords(wordBagSonnets,10)

  % word frequency
  clf;
  wordPos = wordBagSonnets.Vocabulary == "love";
  bar(wordBagSonnets.Counts(:,wordPos))

  % word appearance
  loveContext = context(tokenizedSonnets,"love");
  disp(loveContext.Context)
  ```

- Cleaning words
  ```matlab
  tokenizedSonnets = erasePunctuation(tokenizedSonnets); 
  tokenizedSonnets = removeStopWords(tokenizedSonnets); 
  tokenizedSonnets = lower(tokenizedSonnets); 
  ```
  - Reduce words to their root form using `normalizeWords`
  ```matlab
  tokenizedSonnets = addPartOfSpeechDetails(tokenizedSonnets);
  tokenizedSonnets = normalizeWords(tokenizedSonnets,'Style','lemma');
  ```


[Modeling Using Qualitative Description](module%205/Text/ExampleOfFeatureEngineeringUsingTextDescriptions.mlx)



