# Monthly Stock Aggregation & Technical Indicators (Pandas)

The focus of this implementation is **correct financial logic**, **clarity**, and **no dependency on technical-analysis libraries**.

---

## Dataset Overview

**Input Columns**
```
date, volume, open, high, low, close, adjclose, ticker
```

**Tickers**
```
AAPL, AMD, AMZN, AVGO, CSCO, MSFT, NFLX, PEP, TMUS, TSLA
```

---

## What the Script Does

1. Reads daily stock price data  
2. Groups data by stock symbol  
3. Resamples daily data into **monthly OHLC format**  
4. Calculates **SMA 10, SMA 20, EMA 10, EMA 20**  
5. Writes **one CSV per stock**, each with **24 rows**


## Why the Open Price May Look “Slightly Off”

You may notice that the **monthly open price does not always match expectations** if you compare it with adjusted prices.

### Reason:
- The dataset provides **raw `open` prices**
- Adjustments (splits/dividends) are applied **only to `adjclose`**
- There is **no adjusted open column**

### What was done:
- **Open** is taken directly from the **first trading day’s raw open**
- **No artificial adjustment** is applied to open

### Why this is correct:
Applying the adjustment factor to open would require:
```
Adjustment Factor = adjclose / close
Adjusted Open = open × adjustment factor
```

This adjustment was **intentionally NOT applied** because:
- The dataset does not provide official adjusted open
- Mixing adjusted and unadjusted values can distort OHLC integrity
- The task explicitly focuses on OHLC correctness, not synthetic prices

✔ Result:  
**The open may appear slightly off when compared to adjusted prices, but it is financially correct.**

---
## Why Some SMA and EMA Values Are Missing (NaN)

You may notice that the **initial rows** in the `SMA_10`, `SMA_20`, `EMA_10`, and `EMA_20` columns contain missing values (`NaN`).  
This behavior is **expected and mathematically correct**.

---

### Simple Moving Average (SMA)

An SMA of period **N** requires **N completed data points**.

For example:
- **SMA 10** needs the last **10 monthly closing prices**
- **SMA 20** needs the last **20 monthly closing prices**

Since the dataset contains **24 monthly rows**:
- SMA 10 will be missing for the **first 9 months**
- SMA 20 will be missing for the **first 19 months**

This is because there is **insufficient historical data** to compute the average during those early months.

---

### Exponential Moving Average (EMA)

Although EMA is a weighted moving average, it still needs an **initial starting value**.

In this project:
- The **first EMA value is initialized using the corresponding SMA**
- EMA 10 starts only after SMA 10 becomes available
- EMA 20 starts only after SMA 20 becomes available

As a result:
- EMA 10 is missing for the **first 9 months**
- EMA 20 is missing for the **first 19 months**

---

## Technical Indicator Calculations

All indicators are calculated using **monthly closing prices only**.

---

### Simple Moving Average (SMA)

**Formula**
```
SMA(N) = (Sum of last N closing prices) / N
```

---

### Exponential Moving Average (EMA)

**Multiplier**
```
Multiplier = 2 / (N + 1)
```

**Recursive Formula**
```
EMA = (Current Close − Previous EMA) × Multiplier + Previous EMA
```

The **first EMA value** is initialized using the corresponding SMA.

---

These internally compute **the same mathematical result**.

However, this project:
- Implements SMA & EMA **manually**
- Matches **academic and textbook formulas**
- Keeps calculations **explicit and auditable**

Built-in functions could replace the manual logic **without changing results**.

---

## Output Format

Each stock generates one file:

```
result_AAPL.csv
result_AMD.csv
...
result_TSLA.csv
```

![Output Screenshot](image.png)
