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

## 2 Organize Data

## 3 Clean Data

## 4 Find Features that Matter

## 5 Domain Specific Feature Engineering

