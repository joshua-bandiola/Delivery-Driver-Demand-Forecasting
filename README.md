# Delivery Driver Demand Forecasting 

## Description 
This project forecasts hourly delivery driver demand using historical delivery activity, operational features, and time-based patterns. 

The goal is to help a delivery business plan staffing levels more accurately, reduce driver shortages during peak hours, and avoid unnecessary labour costs during low-demand periods.

Several forecasting approaches were tested, including SARIMAX and XGBoost. The final SARIMAX model achieved the strongest performance, showing that the data had strong daily seasonality and that a statistical time-series model was well suited to the problem.

## Business Context
Delivery platforms need to match driver availability with customer demand. If there are too few drivers in an area, orders may take longer to be picked up and delivered. This can lead to colder food, longer waiting times, lower customer satisfaction, and potential loss of repeat customers. The platform may also need to offer higher driver incentives or surge pay to attract more drivers, increasing operational costs.

However, having too many drivers is also a problem. If driver supply is much higher than order demand, drivers may spend more time idle and earn less money per hour. Over time, this can reduce driver satisfaction and increase driver churn. From a business perspective, idle driver capacity also creates inefficiency because the available workforce is not being used effectively.

The aim is therefore not simply to have more drivers, but to synchronise driver supply with expected demand. A good forecasting model can help estimate how many drivers are likely to be needed during each hour, allowing the business to plan staffing levels, incentives, and operational decisions more effectively.

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

Closed business hours were treated as zero demand, meaning drivers_this_hour was set to 0. Open-hour missing values were handled carefully. They were not automatically converted to zero because missing data does not necessarily mean there was no demand. This helped preserve the true temporal structure of the dataset. 

Three model setups were tested:
- SARIMAX with missing values preserved.
- SARIMAX with time-interpolated values.
- XGBoost with time-interpolated values.

This allowed a fair comparison between keeping the original time-series structure and using interpolation for models that require complete data.

## Models Compared

The following models were tested and cross validated:

| Model | MAE | Notes |
|---|---:|---|
| SARIMAX with NaN's preserved | 4.20 | Used is_open as the external regressor while preserving missing target values |
| SARIMAX with transformed/interpolated data | 5.52 | Used time-interpolated values with is_open and driver_lag_24 |
| XGBoost with transformed/interpolated data | 5.19 | Used time-interpolated data with lag features and selected external variables |

MAE measures the average absolute forecast error. In this project, an MAE of 4.20 means that, on average, the model's hourly driver forecast was approximately 4 drivers away from the actual requirement.

SARIMAX outperformed XGBoost because the dataset showed strong time-series structure, especially daily seasonality. SARIMAX is designed to model autoregressive and seasonal patterns directly, which made it well suited to hourly driver demand forecasting.

XGBoost performed well, but it relied heavily on manually engineered lag features to understand time dependence. Because the dataset was relatively limited and had missing time periods, the tree-based model had less consistent temporal information to learn from.

Although SARIMAX performed best, XGBoost may still be useful in a larger dataset with more external predictors, such as promotions, regional demand, driver availability, pricing, or live traffic data.

## Interpretability vs Accuracy

SARIMAX was more interpretable because its structure captures time-series patterns directly, and its only external feature, is_open, was easy to explain from a business perspective.

XGBoost can capture more complex non-linear relationships, but it is less transparent. It also relied on several engineered lag and external features, making it harder to explain exactly how each prediction was produced. While feature importance can show which variables influenced the model, it does not provide the same direct time-series explanation as SARIMAX.

For a business setting, interpretability is important because staffing managers need to understand and trust the forecast before using it for scheduling decisions.

**General limitations:**
- The dataset covers a limited time period. 
- Some time periods contain missing observations.
- Additional external factors such as promotions, local events, driver availability, and pricing were not available.

## Real-World Business Impact
This project shows how forecasting can support practical operational decisions, not just produce a prediction. In a delivery business, knowing expected driver demand by hour can help teams prepare for busy periods and reduce avoidable service issues.

For example, if the model predicts high driver demand during a dinner peak, the business could prepare by encouraging more drivers to work in that area or offering targeted incentives before the shortage occurs. If the model predicts low demand, the business may avoid oversupplying drivers and reduce idle time.

The forecast could be used by an operations team to decide:
- When to increase driver incentives.
- Which hours require higher driver availability.
- When demand is likely to exceed normal capacity.
- Whether certain time periods are at risk of understaffing.
- How much staffing buffer is needed during peak periods.

The forecast will not remove uncertainty completely. Drivers still choose when and where to work, demand can change suddenly, and external events can disrupt normal patterns. However, even an imperfect forecast can improve planning by giving the business a data-driven estimate instead of relying only on manual judgement.

## Conclusion
The final SARIMAX model provided the strongest forecasting performance, achieving the lowest cross-validated MAE across the models tested. This suggests that SARIMAX was well suited to the time-series structure of the problem, particularly the recurring daily demand patterns.

Overall, this project shows how forecasting can support practical business decisions beyond model accuracy. By estimating hourly driver demand, the model can help operations teams improve workforce planning, reduce the risk of under- or overstaffing, and support more reliable delivery operations.
