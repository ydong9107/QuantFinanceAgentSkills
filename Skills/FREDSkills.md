# Retrieving Economic Data from FRED in MATLAB

This document provides guidance on how to retrieve economic data from the Federal Reserve Economic Data (FRED) database using MATLAB. FRED is a comprehensive source of economic data maintained by the Federal Reserve Bank of St. Louis.

## Prerequisites
1. **MATLAB Installed**: Ensure MATLAB is installed on your system.
2. **Datafeed Toolbox**: The Datafeed Toolbox is required to access FRED data.
3. **FRED API Key**: Obtain an API key from the FRED website (https://fred.stlouisfed.org/).

## Steps to Retrieve Data
1. **Set Up the FRED Connection**:
    Use the `fred` function to establish a connection to the FRED database.

2. **Search for Data**:
    Use the `search` function to find datasets of interest.

3. **Retrieve Data**:
    Use the `fetch` function to download the data.

## Example: Retrieve GDP Data
Below is an example of retrieving U.S. GDP data from FRED.

```matlab
% Step 1: Establish a connection to FRED
c = fred('Your_API_Key_Here');

% Step 2: Search for GDP data
searchResults = search(c, 'GDP');

% Display the first few search results
disp(searchResults(1:5, :));

% Step 3: Fetch GDP data
% Replace 'GDPC1' with the series ID of the desired GDP dataset
data = fetch(c, 'GDPC1');

% Step 4: Plot the data
dates = data.Data(:, 1); % Extract dates
values = data.Data(:, 2); % Extract values

figure;
plot(dates, values);
datetick('x', 'yyyy');
title('Real Gross Domestic Product (GDP)');
xlabel('Year');
ylabel('Billions of Chained 2012 Dollars');
grid on;

% Step 5: Close the connection
close(c);
```

## Notes
- Replace `'Your_API_Key_Here'` with your actual FRED API key.
- Use the `search` function to explore other datasets available in FRED.

## References
- [FRED API Documentation](https://fred.stlouisfed.org/docs/api/fred/)
- MATLAB Datafeed Toolbox Documentation
