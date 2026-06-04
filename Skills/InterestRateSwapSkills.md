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

### Step 3: Calculate Swap Cash Flows
Determine the fixed and floating payment schedules based on the swap specifications.

### Step 4: Discount Cash Flows
Apply the discount curve to present-value the future cash flows.

### Step 5: Calculate Swap Value
Compute the net present value of the swap by comparing fixed and floating leg values.

## Example: Value a 5-Year Interest Rate Swap

Below is a comprehensive example of valuing a 5-year interest rate swap with a 3% fixed rate and 3M-LIBOR floating rate.

```matlab
% Step 1: Create a discount curve from market spot rates
% Define spot rates (annual, e.g., 2%, 2.5%, 3%, etc.)
spotRates = [0.02; 0.025; 0.03; 0.035; 0.04];  % 2%, 2.5%, 3%, 3.5%, 4%
maturityTerms = [1; 2; 3; 4; 5];  % Years

% Create a rate curve object
curve = ratecurve("zero", today(), maturityTerms, spotRates);

% Step 2: Define the interest rate swap specifications
swapStartDate = today();
swapMaturityDate = swapStartDate + years(5);  % 5-year swap
fixedRate = 0.03;  % 3% fixed rate
notionalAmount = 1e6;  % $1 million notional
dayCount = "Actual/360";  % Day count convention
fixedPaymentFrequency = 2;  % Semi-annual (2 times per year)
floatingPaymentFrequency = 4;  % Quarterly (4 times per year) for 3M-LIBOR

% Step 3: Create the interest rate swap instrument
% Create a fixed leg
fixedLeg = "Fixed";
fixedSchedule = [swapStartDate, swapMaturityDate];

% Create a floating leg
floatingLeg = "Floating";
floatingLegRates = [0.025; 0.03; 0.035; 0.04];  % Forward rates for floating leg

% Step 4: Generate payment schedules
% Fixed leg payment dates (semi-annual)
fixedPaymentDates = swapStartDate:calmonths(6):swapMaturityDate;
fixedPaymentDates = fixedPaymentDates(fixedPaymentDates <= swapMaturityDate)';

% Floating leg payment dates (quarterly)
floatingPaymentDates = swapStartDate:calmonths(3):swapMaturityDate;
floatingPaymentDates = floatingPaymentDates(floatingPaymentDates <= swapMaturityDate)';

% Step 5: Calculate discounting factors
% Calculate discount factors for all payment dates
allPaymentDates = unique([fixedPaymentDates; floatingPaymentDates]);
discountFactors = discountfactor(curve, allPaymentDates);

% Step 6: Calculate swap cash flows
% Fixed leg cash flows
fixedCashFlows = zeros(length(fixedPaymentDates), 1);
for i = 1:length(fixedPaymentDates)
    if i == 1
        daysBetween = yearfrac(swapStartDate, fixedPaymentDates(i), dayCount);
    else
        daysBetween = yearfrac(fixedPaymentDates(i-1), fixedPaymentDates(i), dayCount);
    end
    fixedCashFlows(i) = notionalAmount * fixedRate * daysBetween;
end

% Floating leg cash flows (using forward rates)
% For simplicity, using pre-determined rates; in practice, use market forward rates
floatingCashFlows = zeros(length(floatingPaymentDates), 1);
floatingRates = [0.025; 0.026; 0.027; 0.028; 0.029; 0.03; ...
                 0.031; 0.032; 0.033; 0.034; 0.035; 0.036; ...
                 0.037; 0.038; 0.039; 0.04; 0.041; 0.042; ...
                 0.043; 0.044];  % Example forward rates for each quarter

for i = 1:min(length(floatingPaymentDates), length(floatingRates))
    if i == 1
        daysBetween = yearfrac(swapStartDate, floatingPaymentDates(i), dayCount);
    else
        daysBetween = yearfrac(floatingPaymentDates(i-1), floatingPaymentDates(i), dayCount);
    end
    floatingCashFlows(i) = notionalAmount * floatingRates(i) * daysBetween;
end

% Step 7: Calculate present values
% PV of fixed leg (receiver pays fixed)
pv_fixed = 0;
for i = 1:length(fixedPaymentDates)
    idx = find(allPaymentDates == fixedPaymentDates(i));
    pv_fixed = pv_fixed + fixedCashFlows(i) * discountFactors(idx);
end

% PV of floating leg (receiver receives floating)
pv_floating = 0;
for i = 1:length(floatingPaymentDates)
    if i <= length(floatingCashFlows)
        idx = find(allPaymentDates == floatingPaymentDates(i));
        if ~isempty(idx)
            pv_floating = pv_floating + floatingCashFlows(i) * discountFactors(idx);
        end
    end
end

% Step 8: Calculate swap value
% From the perspective of the fixed-rate payer
swapValue = pv_floating - pv_fixed;

% Display results
fprintf('Interest Rate Swap Valuation Results:\n');
fprintf('======================================\n');
fprintf('Swap Start Date: %s\n', datestr(swapStartDate));
fprintf('Swap Maturity Date: %s\n', datestr(swapMaturityDate));
fprintf('Fixed Rate: %.2f%%\n', fixedRate * 100);
fprintf('Notional Amount: $%.0f\n', notionalAmount);
fprintf('---\n');
fprintf('PV of Fixed Leg: $%.2f\n', pv_fixed);
fprintf('PV of Floating Leg: $%.2f\n', pv_floating);
fprintf('Swap Value (receiver of fixed): $%.2f\n', swapValue);
fprintf('Swap Value (payer of fixed): $%.2f\n', -swapValue);
```

## Alternative: Using Financial Instrument Toolbox Objects

For a more advanced and production-ready approach, use Financial Instrument Toolbox objects:

```matlab
% Create instruments using Financial Instrument Toolbox
assetIns = fininstrument("Swap", ...
    "Leg1Type", "Fixed", ...
    "Leg1Rate", 0.03, ...
    "Leg2Type", "Float", ...
    "Leg2Spread", 0.01, ...
    "Leg2ResetTimes", [0.25, 0.5, 0.75, 1], ...
    "StartDate", today(), ...
    "Maturity", years(5), ...
    "Principal", 1e6, ...
    "DayCount", "Actual/360", ...
    "PaymentFrequencies", [2, 4]);  % Fixed: semi-annual, Floating: quarterly

% Price the swap
priceSwap = price(pricer, assetIns, "DiscountCurve", curve);
```

## Important Notes
- **Day Count Conventions**: Common conventions include Actual/360, Actual/Actual (ISDA), and 30/360. Ensure consistency with market conventions.
- **Reset Dates**: Floating rate legs reset at specific dates. Use `ratebetweendate` or market-based forward rate methods to determine floating rates.
- **Curve Bootstrapping**: For real-world applications, bootstrap the discount curve from market instruments (Treasuries, LIBOR futures, swap rates).
- **SOFR Transition**: As of 2021, LIBOR is being phased out. Use SOFR (Secured Overnight Financing Rate) instead of LIBOR for U.S. dollar swaps.
- **Basis Spreads**: Account for basis spreads between different floating rate indices.
- **Valuation Date vs. Trade Date**: Always specify the valuation date; swap values change continuously as rates move.

## References
- [Financial Instrument Toolbox Documentation](https://www.mathworks.com/help/fininst/)
- [Financial Toolbox Documentation](https://www.mathworks.com/help/finance/)
- [Interest Rate Swap Pricing Models](https://www.mathworks.com/help/fininst/interest-rate-swap-models.html)
- [ISDA Standard Definitions](https://www.isda.org/standards/)
