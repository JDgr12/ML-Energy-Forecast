# Nowcasting: Italian Fuel & Heating Inflation

This is a machine learning project that aims to implement / create a nowcasting system capable of estimating Italian energy inflation before Eurostat officially publishes the data.

## What is the actual problem?

The problem stems from a time lag:

- Market prices for oil, petrol, diesel and the EUR/USD exchange rate are observed **daily**.
- The Italian HICP for fuels is published **monthly** and with a delay.
- Therefore, for several weeks, we know what is happening in the markets, but we do not yet officially know by how much the prices paid by Italian consumers have changed.

Can we use energy market data available today to estimate how fuel inflation is behaving in Italy before Eurostat publishes the figures?

**Additional problem** If today were, for example, 15 August, we would only know approximately half of August’s market prices.
To estimate August’s inflation, we need data representative of the whole of August.

## Setup

1. Ensure Python 3.11 + is installed
2. Create and activate a virtual environment:
    - `python -m venv .venv`
    - `.venv\Scripts\activate`
3. Install dependencies:
    - `pip install -r requirements.txt`

## Project Structure

* `Data/`
* `Extra/` - Project description and dataset schema.
* `Mocks/` - output reports and documentation for tryouts and control.
* `Notebooks/` - Jupyter Notebooks for analysis, experimentation and final resutls.
* `.gitignore/` - Exlution of datasets and environment files.
* `requirements` - Project Dependencies.
* `README.md` - Principal documentation and description made on the project.

## Project objectives

- **Market Modelling (Part 1):** To analyse and model daily time series from international energy markets in order to assess their short-term mathematical predictability.
- **Macroeconomic Forecasting (Part 2):** Estimate (nowcast) monthly fuel inflation in Italy (HICP) using information from global financial markets.
- **Data Alignment:** Resolve discrepancies between the business day calendars of the United States and the European Central Bank.

## Methodology and Justification

- **Forward Fill:** Gaps caused by public holidays were filled by carrying forward the last known closing price, ensuring that there is no **data leakage** from the future.
- **Transformation to Growth Rates:** As the HICP is an index, the time series were transformed into monthly percentage changes to model real inflation and ensure stationarity.
- **Efficient Market Models:** A Naive Forecast was evaluated against Linear Regression and Random Forest, confirming that daily markets follow a **Random Walk** pattern.
- **Nowcasting with Ridge Regression:** **Ridge** was chosen to forecast inflation due to its ability to handle the high correlation between energy variables **(multicollinearity)** and avoid overfitting.
- **Robust Benchmarking:** It was analytically demonstrated that Random Forest algorithms struggle to extrapolate data and model macroeconomic noise in small samples, confirming **Ridge** as the best option.

## Key Findings

- **Limitations of daily forecasting:** The models demonstrated empirically that, at the daily level, energy prices are extremely efficient; beating the latest known price is practically impossible.
- **Explanatory power (Nowcast):** The monthly macroeconomic model achieved a test $R^2$ of over 0.22, a solid and highly valuable performance in anticipating Eurostat reports.
- **Economically Sensible Validation:** The model’s coefficients confirmed that US refined products are stronger predictors for Italian consumers than crude oil or exchange rate fluctuations.
