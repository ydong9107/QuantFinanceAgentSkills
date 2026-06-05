# Interest Rate Swap Valuation in MATLAB

This document provides guidance on how to generate MATLAB code for valuing interest rate swaps using the Financial Instrument Toolbox. Interest rate swaps are fundamental derivatives used to exchange cash flows based on different interest rate bases.

## Prerequisites
1. **MATLAB Installed**: Ensure MATLAB is installed and properly configured on your system.
2. **Financial Instrument Toolbox**: Required to access swap valuation functions and market data utilities.
3. **Financial Toolbox**: Required for interest rate curve creation and financial calculations.
4. **Curve Fitting Toolbox** (Optional): Useful for constructing and interpolating discount curves.

## Understanding Interest Rate Swaps
An interest rate swap is a financial derivative where two parties exchange cash flows based on:
- **Fixed Leg**: One party pays/receives a fixed interest rate
- **Floating Leg**: The other party pays/receives a floating rate (e.g., LIBOR, SOFR) plus a spread

## Steps to Value an Interest Rate Swap

### Step 1: Set Up the Discount Curve and create ratecurve object
Create or retrieve a discount curve that represents the term structure of interest rates. This curve is essential for discounting future cash flows.

### Step 2: Create the Interest Rate Swap Instrument Object
Define the swap specifications including:
- Start date
- Maturity date
- Leg rate for each leg: Floating and Fixed
- Leg type
- Reset Frequency
- Basis
- Notional principal
- Business Day Convention
- Start Date
If there are any items missing, fill with the default values.

### Step 3: Create Pricer Object
Use finpricer() to create a discount pricer object.

### Step 4: Price Swap Instrument
Use price to compute the price and sensitivities for the vanilla swap instrument.

## Example: Value a 5-Year Interest Rate Swap

Below is a comprehensive example of valuing a 5-year interest rate swap with a 1.9% fixed rate and given floating rate. The maturity date of this swap is 2024/9/15.

```matlab
% Step 1: Create a discount curve from market spot rates
Settle = datetime(2019,9,15);
Type = 'zero';
ZeroTimes = [calmonths(6) calyears([1 2 3 4 5 7 10 20 30])]';
ZeroRates = [0.0052 0.0055 0.0061 0.0073 0.0094 0.0119 0.0168 0.0222 0.0293 0.0307]';
ZeroDates = Settle + ZeroTimes;
 
myRC = ratecurve('zero',Settle,ZeroDates,ZeroRates)

% Step 2: Use fininstrument to create a vanilla swap instrument object
Swap = fininstrument("Swap",'Maturity',datetime(2024,9,15),'LegRate',[0.022 0.019 ],'LegType',["float","fixed"],'ProjectionCurve',myRC,'Name',"swap_instrument")

% Step 3: Use finpricer to create a Discount pricer object and use the ratecurve object for the 'DiscountCurve' name-value pair argument.
outPricer = finpricer("Discount", 'DiscountCurve',myRC)

% Step 4: Price Swap Instrument
[Price, outPR] = price(outPricer, Swap,["all"])
% Display the results
outPR.Results
```

## Important Notes
- **Day Count Conventions**: Common conventions include Actual/360, Actual/Actual (ISDA), and 30/360. Ensure consistency with market conventions.
- **Reset Dates**: Floating rate legs reset at specific dates. Use `ratebetweendate` or market-based forward rate methods to determine floating rates.
- **Curve Bootstrapping**: For real-world applications, bootstrap the discount curve from market instruments (Treasuries, LIBOR futures, swap rates).
- **SOFR Transition**: As of 2021, LIBOR is being phased out. Use SOFR (Secured Overnight Financing Rate) instead of LIBOR for U.S. dollar swaps.
- **Basis Spreads**: Account for basis spreads between different floating rate indices.
- **Valuation Date vs. Trade Date**: Always specify the valuation date; swap values change continuously as rates move.
- **Data Inputs**: Load the swap settings and curve rate from Excel workbook. Assign these values into ratecurve and swap object.
- **Available Models**: ratecurve object, Hull White model, Black Karasinski model, Black Dermon Toy model, Cox Ingersoll Ross model and Linear Gaussian 2F model
- **Available Pricers**: Discount, IRTree for Hull White, Black Karasinski, Cox Ingersoll Ross, or Black Derman Toy models; IRMonteCarlo for Hull White, Black Karasinski, or Linear Gaussian 2F models



## References
- [Financial Instrument Toolbox Documentation](https://www.mathworks.com/help/fininst/)
- [Choose Instruments, Models, and Pricers](https://www.mathworks.com/help/fininst/choosing-instruments-models-and-pricers.html)
- [Financial Toolbox Documentation](https://www.mathworks.com/help/finance/)
- [Interest Rate Swap Pricing Models](https://www.mathworks.com/help/fininst/interest-rate-swap-models.html)
- [ISDA Standard Definitions](https://www.isda.org/standards/)
