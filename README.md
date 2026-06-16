# Delivery Driver Demand Forecasting 

## Description 
This project forecasts hourly delivery driver demand using historical delivery activity, operational features, and time-based patterns. 
The goal is to help a delivery business plan staffing levels more accurately, reduce driver shortages during peak hours, and avoid unnecessary labour costs during low-demand periods.

Several forecasting approaches were tested, including SARIMAX and XGBoost. The final SARIMAX model achieved the strongest performance, showing that the data had strong daily seasonality and that a statistical time-series model was well suited to the problem.

## Business Context
Delivery platforms need to match driver availability with customer demand. If there are too few drivers in an area, orders may take longer to be picked up and delivered. This can lead to colder food, longer waiting times, lower customer satisfaction, and potential loss of repeat customers. The platform may also need to offer higher driver incentives or surge pay to attract more drivers, increasing operational costs.

However, having too many drivers is also a problem. If driver supply is much higher than order demand, drivers may spend more time idle and earn less money per hour. Over time, this can reduce driver satisfaction and increase driver churn. From a business perspective, idle driver capacity also creates inefficiency because the available workforce is not being used effectively.

The aim is therefore not simply to have more drivers, but to synchronise driver supply with expected demand. A good forecasting model can help estimate how many drivers are likely to be needed during each hour, allowing the business to plan staffing levels, incentives, and operational decisions more effectively.

## Objectives

- Forecast the number of drivers required per hour.
- Identify strong time-based demand patterns such as hour-of-day and daily seasonality.
- Compare statistical and machine learning forecasting models.
- Evaluate model performance using time-series cross-validation.
- Translate forecast accuracy into business impact for staffing decisions.

## Dataset Overview

The dataset was taken here: https://github.com/derak-isaack/UberEATSAnalytics/. The dataset contains delivery order records with timestamps, delivery times, traffic conditions, weather, festival indicators, and delivery batch information. The raw order-level data was aggregated into an hourly time series so that driver demand could be forecast at an operational planning level.

Key variables used:

- `drivers_this_hour`: target variable representing hourly driver demand.
- `deliveries_this_hour`: number of deliveries in each hour.
- `avg_time`: average delivery duration.
- `hour`: hour of the day.
- `day_of_week`: day of the week.
- `is_open`: indicator for business operating hours.
- Lag features such as `driver_lag_1`, `driver_lag_24`, and `driver_lag_168`.
- External variables such as weather, traffic, festival, and delivery batch size.

## Data Preparation

The raw delivery records were converted into an hourly time series. This allowed the project to focus on predicting operational demand at the level where staffing decisions are made.
Closed business hours were treated as zero demand because no drivers are required when the business is closed.

Open-hour missing values were handled carefully. They were not automatically converted to zero because missing data does not necessarily mean there was no demand. This helped preserve the true temporal structure of the dataset.

Different preprocessing approaches were used depending on the model. SARIMAX can handle missing target values within a state-space framework, while machine learning models such as XGBoost require a complete feature matrix.

## Exploratory Data Analysis

Exploratory analysis showed that driver demand followed clear time-based patterns.

The strongest pattern was hour-of-day demand, showing that certain hours consistently required more drivers than others. Autocorrelation analysis also showed daily seasonality around the 24-hour lag and some weekly structure around the 168-hour lag.

These findings confirmed that driver demand was not random and could be forecast using time-series methods.

## Feature Engineering

Several features were created to improve forecasting performance:
- Time-based features: hour, day of week, weekend indicator, month.
- Operational feature: business open/closed indicator.
- Lag features: previous hour, previous day, and previous week demand.
- Rolling statistics: rolling averages and rolling standard deviations.
- Encoded external variables: weather, traffic, and festival indicators.

Care was taken to avoid data leakage. Features that would not be known at forecast time were not used as direct predictors in the final model comparison.

## Models Compared

The following models were tested and cross validated:

| Model | MAE | Notes |
|---|---:|---|
| SARIMAX with NaN's preserved | 4.20 | Best performing model |
| SARIMAX with transformed/interpolated data | 5.52 | Weaker than final SARIMAX |
| XGBoost with transformed/interpolated data | 5.19 | Strong performance but required more feature engineering |

MAE measures the average absolute forecast error. In this project, an MAE of 3.11 means that, on average, the model's hourly driver forecast was approximately 3 drivers away from the actual requirement.

SARIMAX outperformed XGBoost because the dataset showed strong time-series structure, especially daily seasonality. SARIMAX is designed to model autoregressive and seasonal patterns directly, which made it well suited to hourly driver demand forecasting.

XGBoost performed well, but it relied heavily on manually engineered lag features to understand time dependence. Because the dataset was relatively limited and had missing time periods, the tree-based model had less consistent temporal information to learn from.

Although SARIMAX performed best, XGBoost may still be useful in a larger dataset with more external predictors, such as promotions, regional demand, driver availability, pricing, or live traffic data.

## Interpretability vs Accuracy

SARIMAX provides better interpretability because it directly models time-series components such as seasonality and autoregression. This makes it easier to explain why demand is expected to rise or fall at certain times.

XGBoost can capture more complex non-linear patterns, but it is less transparent. Feature importance can help explain which variables influence the model, but it does not provide the same direct explanation as a statistical time-series model.

For a business setting, interpretability is important because staffing managers need to understand and trust the forecast before using it for scheduling decisions.

**General limitations:**
- The dataset covers a limited time period. 
- Some time periods contain missing observations.
- Additional external factors such as promotions, local events, driver availability, and pricing were not available.

## Real-World Business Impact
This project is useful because it connects machine learning forecasting to operational decision-making. Instead of only predicting a number, the forecast can support real business actions.

For example, if the model predicts high driver demand during a dinner peak, the business could prepare by encouraging more drivers to work in that area or offering targeted incentives before the shortage occurs. If the model predicts low demand, the business may avoid oversupplying drivers and reduce idle time.

The forecast could be used by an operations team to decide:
- When to increase driver incentives.
- Which hours require higher driver availability.
- When demand is likely to exceed normal capacity.
- Whether certain time periods are at risk of understaffing.
- How much staffing buffer is needed during peak periods.

The forecast does not eliminate uncertainty completely. Drivers are independent decision-makers, demand can spike unexpectedly, and external events can disrupt normal patterns. However, even an imperfect forecast can improve planning by giving the business a data-driven estimate instead of relying only on manual judgement.

## Conclusion
The final SARIMAX model provided the best forecasting performance and was well suited to the time-series nature of the problem. The project demonstrates how forecasting can be used not only to improve model accuracy, but also to support practical business decisions such as workforce planning, cost control, and operational reliability.
