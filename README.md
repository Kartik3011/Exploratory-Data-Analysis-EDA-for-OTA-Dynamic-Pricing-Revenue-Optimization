# Exploratory Data Analysis (EDA) for OTA Dynamic Pricing Analytics

Step by step EDA on hotel bookings & search behaviour for an Online Travel Agency (MakeMyTrip / Goibibo-scale)

## Business Scenario
A MakeMyTrip/Goibibo scale Online Travel Agency (OTA) platform identified a critical revenue leak: hotel pricing across Tier-2/3 Indian cities (e.g., Jaipur, Udaipur, Rishikesh) was static and manually adjusted. This resulted in missed revenue during festival demand spikes and severe over pricing.

This project audits historical platform data, clean raw engineering extracts, engineer demand sensitive features, and lay the groundwork for a dynamic pricing and search ranking machine learning model. 

## Data Architecture & Handoff
The data simulates a anonymous 3 year historical extract sourced from the client's Snowflake/BigQuery warehouse. The data contract consists of a star schema with four tables:

* **`dim_hotels`**: Property master containing attributes (star rating, base price, distance from airport).
* **`dim_customers`**: Customer demographics and acquisition channels.
* **`fact_search_pricing`**: Demand signal table consisting of individual search events, competitor prices, and click/conversion flags.
* **`fact_bookings`**: Transaction table logging confirmed stays, net amount paid, and cancellations.

## Methodology

### 1. Business Logic Data Cleaning
A strict null handling hierarchy was applied to prevent data distortion before analysis:
* **<5% Missing:** Imputed using statistical median for skewed numeric data (e.g., base_price_inr) and mode for categoricals.
* **5–25% Missing:** Leaned heavily toward median fills to respect real-world revenue skew.
* **>25% Missing (Business Critical):** Never fake-filled. Created explicit "Missing" categorical buckets (e.g., competitor_avg_price, review_count) or utilized flag columns coupled with median fills to preserve signal.
* **Structural Nulls:** Columns like "post_stay_rating" were left null for canceled bookings to prevent faking customer satisfaction metrics.

### 2. Exploratory Data Analysis (EDA)
Following standardization to snake_case and datetime resolution, the exploratory phase focused on baseline metrics:
* Demand distributions across Tier 1, 2, and 3 cities.
* Seasonality spikes mapped to regional events.
* Booking channel mix and device preference impact on conversion.

### 3. Feature Engineering
Raw columns rarely explain cancellation risk or dynamic pricing efficiently. Features were engineered based on domain specific revenue management principles:

| Engineered Feature | Logic | Business Value |
| :--- | :--- | :--- |
| `is_festival_stay` | 1 if check-in falls near Diwali/Holi/New Year, else 0 | Captures discrete demand surges. |
| `is_weekend_stay` | 1 if check-in is Friday/Saturday | Calculates weekend premium behavior. |
| `price_gap_vs_competitor` | (price_shown - competitor_price) / competitor_price | Calculates if a listing is priced above or below market average. |
| `days_to_checkin_bucket` | Groups lead_time_days into Last-minute / Advance / Early | Simplified pricing models. |
| `revenue_per_room_night` | net_amount_paid / (room_nights * num_rooms) | Revenue across varying stay lengths. |
| `customer_tenure_days` | booking_date - signup_date | Displays customer tenure. |

### 4. Target Definition for Modeling
The engineered dataset pipelines into two distinct predictive models representing the OTA business:
1. **Price Prediction (Regression):** Predicting the continuous variable "net_amount_paid_inr" to optimize yield.
2. **Cancellation Risk & Search Conversion (Classification):** Predicting the binary "cancellation_flag" and "conversion_flag" to manage inventory.

## Technologies Used
* **Python** 
* **Pandas / NumPy** (Data manipulation and cleaning)
* **Seaborn / Matplotlib** (Visualizing demand distributions and price gaps)

## How to Run
1. Clone the repository:
   ```bash
   git clone [https://github.com/kartik3011/ota-dynamic-pricing-analytics.git](https://github.com/kartik3011/ota-dynamic-pricing-analytics.git)
2. Change the location of the datasets inside pd.readcsv as per your dataset location.
3. Run EDA project OTA clients.ipynb step by step.
