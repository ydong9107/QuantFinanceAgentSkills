# Retrieving Economic Data from FRED in MATLAB

This document provides guidance on how to retrieve economic data from the Federal Reserve Economic Data (FRED) database using MATLAB. FRED is a comprehensive source of economic data maintained by the Federal Reserve Bank of St. Louis.

## Prerequisites
1. **MATLAB Installed**: Ensure MATLAB and datafeed toolbox are installed in the system.
2. **Datafeed Toolbox**: The Datafeed Toolbox is required to access FRED data.
3. **FRED API Key**: Obtain an API key from the FRED website (https://fred.stlouisfed.org/).

## Steps to Retrieve Data
1. **Set Up the FRED Connection**:
    Use the `fredrs` function to create FRED REST connection object.

2. **Search for Series Name**:
    Find the series name from the Federal Economic Data System. 

3. **Retrieve Data**:
    Use the `series` function to download the data.

## Example: Retrieve GDP Data
Below is an example of retrieving U.S. GDP data from FRED.

```matlab
% Step 1: Create FRED REST connection object
c = fredrs(apikey);

% Step 2: Fetch GDP data
% Replace 'GDPC1' with the series ID of the desired GDP dataset
data = series(c, 'GDPC1', 'observations');
dinfo = series(c,'GDPC1');
values = data.observations{1,1}(:,3:4);

% Step 3 (Optional): Plot the data
figure;
plot(values.date, values.value,'-');
title(dinfo.seriess.title);
xlabel('Year');
ylabel(dinfo.seriess.units);
grid on;
```

## Notes
- Replace `'Your_API_Key_Here'` with your actual FRED API key.

## References
- [FRED API Documentation](https://fred.stlouisfed.org/docs/api/fred/)
- [MATLAB Datafeed Toolbox Documentation](https://www.mathworks.com/help/datafeed/index.html)
