# RedDoorz Commercial Performance Analytics Project
<img width="736" height="520" alt="WhatsApp Image 2026-04-24 at 10 32 17" src="https://github.com/user-attachments/assets/b9593280-7ce9-4952-a464-bb58db8a7267" />

---

# Table of Content

- [Business Understanding](#business-understanding)
- [Data Source](#data-source)
- [Stages of Analysis](#stages-of-analysis)
  - [Stage 1 - Data Preparation](#stage-1---data-preparation)
  - [Stage 2 - Feature Engineering (Occupancy Rate)](#stage-2---feature-engineering-occupancy-rate)
  - [Stage 3 - Exploratory Data Analysis](#stage-3---exploratory-data-analysis)
  - [Stage 4 - Revenue Driver Analysis](#stage-4---revenue-driver-analysis)
  - [Stage 5 - Property Benchmarking](#stage-5---property-benchmarking)
  - [Stage 6 - Customer Prioritization](#stage-6---customer-prioritization)
  - [Stage 7 - Dashboard Development](#stage-7---dashboard-development)
  - [Stage 8 - Key Insights & Business Recommendations](#stage-8---key-insights--business-recommendations)
- [Final Management Priority Roadmap](#final-management-priority-roadmap)
- [Executive Takeaway](#executive-takeaway)

---

# Business Understanding
## Background
As a hospitality platform operating across multiple cities and property partners, RedDoorz needs data-driven visibility into business performance, portfolio health, and customer opportunities.

Management requires a centralized analytical framework to understand:
- How overall business performance has changed over time
- Which cities and properties are driving or dragging growth
- Whether revenue growth comes from demand expansion or pricing power
- Which customer segments should be prioritized for marketing investment
- What strategic actions can improve revenue and portfolio efficiency

---

## Business Problems
Despite revenue growth, management lacks clarity on:
1. Is business growth healthy and sustainable?
2. Which operational levers drive revenue: room nights or ADR?
3. Which properties are top performers vs underperformers?
4. Which cities have pricing upside or demand weakness?
5. Which customer segments offer highest current value and future potential?
6. Where should marketing and commercial resources be prioritized?

---

## Project Objectives
This project aims to build an end-to-end commercial analytics dashboard that helps decision-makers:
- Monitor company-wide performance through executive KPIs
- Diagnose revenue growth drivers using decomposition analysis
- Benchmark property portfolio performance
- Prioritize customer segments for marketing strategy
- Generate actionable business recommendations

---

## Key Business Questions
**Executive Performance**
- How did Revenue, Occupancy, ADR, and Room Nights perform YTD 2024 vs prior year?
- Which cities and brand types contribute most revenue?

**Revenue Drivers**
- Is revenue growth driven by higher bookings volume or better pricing?
- Which cities are volume-led vs pricing-led?

**Property Benchmarking**
- Which properties are Star, Strong, Average, or Weak?
- Is growth concentrated in a few properties or broad across portfolio?

**Customer Strategy**
- Which segments generate the most revenue?
- Which segments have strongest retention and growth?
- Which customer groups deserve acquisition vs loyalty budget?

---

# Data Source
This project uses three relational datasets representing hospitality business operations.
- **Transaction Data:** Booking-level transactional records used to measure business performance.
- **Customer Data:** Customer demographic and behavioral attributes used for segmentation.
- **Property Data:** Partner property information used for benchmarking.

---

# Stages of Analysis
- Stage 1 - Data Preparation
- Stage 2 - Feature Engineering (Occupancy Rate)
- Stage 3 - Exploratory Data Analysis
- Stage 4 - Revenue Driver Analysis
- Stage 5 - Property Benchmarking
- Stage 6 - Customer Prioritization
- Stage 7 - Dashboard Development
- Stage 8 - Analysis

---

# Stage 1 - Data Preparation
## Activities:
- Validate date formats
- Clean missing values
- Remove duplicates
- Remove inconsistency data
  - Remove inconsistency data where 'check_in_date > chech_out_date'
  - Remove inconsistency data where 'check_in_date > booking_date'
  - Remove inconsistency data where 'booking_date > cohort_date'

## Python Query:
```python
import pandas as pd

# Remove inconsistency data where 'check_in_date > chech_out_date'
bookings_df = bookings_df[bookings_df['CHECK_OUT_DATE'] >
                          bookings_df['CHECK_IN_DATE']].reset_index(drop=True)

# Remove inconsistency data where 'check_in_date > booking_date'
bookings_df = bookings_df[bookings_df['CHECK_IN_DATE'] >=
                          bookings_df['BOOKING_DATE']].reset_index(drop=True)

# Remove inconsistency data where 'booking_date > cohort_date'
bookings_df2 = pd.merge(
    bookings_df,
    property_df[['PROPERTY_CODE', 'COHORT_DATE']],
    on = 'PROPERTY_CODE',
    how = 'left'
)
bookings_df2 = bookings_df2[bookings_df2['BOOKING_DATE'] >
                            bookings_df2['COHORT_DATE']].reset_index(drop=True)
```

## Output:
Clean analytical base table for reporting.

---

# Stage 2 - Feature Engineering (Occupancy Rate)
## Python Query:
```python
import pandas as pd
import numpy as np

# Make monthly agregation
bookings_df2['MONTH'] = bookings_df2['CHECK_IN_DATE'].dt.to_period('M')

monthly_data = bookings_df2.groupby(['MONTH', 'PROPERTY_CODE']).agg({
    'ROOM_NIGHTS': 'sum',
    'REVENUE_DOLLAR': 'sum'
}).reset_index()

# Merge monthly data with property table (cohort and inventory)
monthly_data = pd.merge(
    monthly_data,
    property_df[['PROPERTY_CODE', 'CITY', 'BRAND_TYPE', 'COHORT_DATE', 'INVENTORY']],
    on='PROPERTY_CODE',
    how='left'
)

# Calculate ACTIVE_DAYS
def get_active_days(row):
    # Taking the first date and the last date from each month
    month_start = row['MONTH'].start_time
    month_end = row['MONTH'].end_time
    num_days_in_month = row['MONTH'].days_in_month
    cohort = row['COHORT_DATE']

    # Condition 1: if hotel open before this month, the active day will be full
    if cohort < month_start:
        return num_days_in_month

    # Condition 2: if hotel open in this month, the active day will be 'end day - cohort day'
    elif month_start <= cohort <= month_end:
        return (month_end.day - cohort.day) + 1

    # Condition 3: Hotel haven't opened
    else:
        return 0

monthly_data['ACTIVE_DAYS'] = monthly_data.apply(get_active_days, axis=1)

# Calculated Room Available, OCC, REVPAR, ADR
monthly_data['ROOMS_AVAILABLE'] = monthly_data['INVENTORY'] * monthly_data['ACTIVE_DAYS']
monthly_data['OCC'] = monthly_data['ROOM_NIGHTS'] / monthly_data['ROOMS_AVAILABLE'].replace(0, np.nan)
monthly_data['REVPAR'] = monthly_data['REVENUE_DOLLAR'] / monthly_data['ROOMS_AVAILABLE'].replace(0, np.nan)
monthly_data['ADR'] = monthly_data['REVENUE_DOLLAR'] / monthly_data['ROOM_NIGHTS'].replace(0, np.nan)

# Delete rows if the revenue < 0 (inconsistency)
monthly_data = monthly_data[monthly_data['REVENUE_DOLLAR'] > 0].copy()
```

## Output:
Engineered a dynamic Occupancy Rate feature by aggregating monthly property data. This metric serves as a core KPI for benchmarking property performance and driving data-driven strategic planning for future business growth.

---

# Stage 3 - Exploratory Data Analysis
## 1. Revenue Trend (2020-2024)
### Python Query:
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sna

df_temp = bookings_df2.set_index('BOOKING_DATE')
trend_flat = df_temp['REVENUE_DOLLAR'].resample('MS').sum().reset_index()

plt.figure(figsize=(16, 6))
ax = plt.gca()

plt.plot(trend_flat['BOOKING_DATE'], trend_flat['REVENUE_DOLLAR'],
         color='#E63946', marker='o', linewidth=2, label='Monthly Revenue')

mean_val = trend_flat['REVENUE_DOLLAR'].mean()
plt.axhline(mean_val, color='black', linestyle='--', alpha=0.5, label=f'Average: {mean_val:.2f}')

plt.title('Monthly Revenue Fluctuation (Discrete Monthly Sum)', fontsize=16)
plt.ylabel('Revenue Dollar')
plt.xlabel('Period')
plt.grid(True, axis='y', alpha=0.3)
plt.legend()
plt.show()
```
### Output:
<img width="1333" height="549" alt="Revenue Trend" src="https://github.com/user-attachments/assets/01246548-c8a4-4fe6-95d7-e60499ced9eb" />

**Brief Insight** : The chart illustrates a robust ***long-term bullish trend*** in monthly revenue, growing from approximately $2,500 in 2020 to over $33,000 by mid-2024. A key insight is the significant momentum gained after 2022, where revenue consistently outperformed the historical average. This steady progression indicates ***strong market demand*** and ***effective operational scaling*** over the four-year period.

---

## 2. Total Bookings by Stay Duration
### Python Query:
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sna

bookings_df2['STAY_DURATION'] = (bookings_df2['CHECK_OUT_DATE'] -
                                 bookings_df2['CHECK_IN_DATE']).dt.days
stay_duration_counts = bookings_df2['STAY_DURATION'].value_counts().sort_index()

plt.figure(figsize=(7, 5))
stay_duration_plot = sna.barplot(x=stay_duration_counts.index, y=stay_duration_counts.values,
                                 palette=['#C0392B'])
stay_duration_plot.set_title('Distribution of Stay Duration', fontsize=16)
stay_duration_plot.set_xlabel('Stay Duration', fontsize=12)
stay_duration_plot.set_ylabel('Number of Bookings', fontsize=12)
plt.show()
```
### Output:
<img width="639" height="476" alt="Stay Duration Distributions" src="https://github.com/user-attachments/assets/b0c23e44-6699-4e46-abb3-5981feee78d5" />

**Brief Insight** : The bookings are classified as ***short-term***, as the maximum stay duration is only ***3 days***. Furthermore, the volume is evenly distributed across all stay durations.

---

## 3. Total Bookings by Lead Time
### Python Query:
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sna

bookings_df2['LEAD_TIME'] = (bookings_df2['CHECK_IN_DATE'] -
                             bookings_df2['BOOKING_DATE']).dt.days
lead_time_counts = bookings_df2['LEAD_TIME'].value_counts().sort_index()

plt.figure(figsize=(10, 6))
lead_time_plot = sna.barplot(x=lead_time_counts.index, y=lead_time_counts.values,
                             palette=['#C0392B'])
lead_time_plot.set_title('Distribution of Lead Time', fontsize=16)
lead_time_plot.set_xlabel('Lead Time', fontsize=12)
lead_time_plot.set_ylabel('Number of Bookings', fontsize=12)
plt.show()
```
### Output:
<img width="872" height="553" alt="Lead Time Distributions" src="https://github.com/user-attachments/assets/502c19fc-9450-48d0-b2b2-d127593f2a5e" />

**Brief Insight** : The distribution reveals a significant ***right-skewed pattern***, where the vast majority of bookings are made with a ***0 to 2-day lead time***. With over 18,000 bookings occurring on day zero, the business exhibits a high reliance on ***spontaneous or last-minute travelers***. This insight is critical for operational readiness and suggests that ***real-time dynamic pricing strategies*** are essential to capture maximum value from this high-volume, short-notice segment.

---

## 4. Bookings Distribution by ADR
### Python Query:
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sna

bookings_df2['ADR'] = bookings_df2['REVENUE_DOLLAR'] / bookings_df2['ROOM_NIGHTS']

plt.figure(figsize=(10, 6))
plt.title('Ditribution of ADR', fontsize = 16)
sna.histplot(bookings_df2['ADR'], kde=True, color='#FF0000')
plt.xlabel('ADR (USD)', fontsize=12)
plt.ylabel('Frequency', fontsize=12)
plt.show()
```

### Output:
<img width="863" height="553" alt="ADR Distributions" src="https://github.com/user-attachments/assets/09176731-d027-4e13-a1ef-65e407eb8584" />

**Brief Insight** : The ADR distribution reveals a ***multi-modal pattern*** with a heavy concentration in the ***$2 – $5 range***, identifying it as ***the primary market volume driver***. The sharp decline in frequency beyond ***$15*** suggests a critical price ceiling for the majority of the customer base.

---

# Stage 4 - Revenue Driver Analysis
## Method:
Revenue decomposition framework used to split growth into:
- Room Night effect (volume)
- ADR effect (pricing)

## Objective:
Identify whether business growth depends on occupancy/volume or monetization.

## Python Query:
***1. Identified for Overall Level***
```python
import pandas as pd
import numpy as np

def calculate_decomposition(df, group_cols):
    # Sort data to ensure chronological order for MoM calculations
    sort_order = group_cols + ['MONTH'] if group_cols else ['MONTH']
    df = df.sort_values(sort_order)

    # Calculate ADR (Average Daily Rate), handling zero room nights to avoid errors
    df['ADR'] = df['REVENUE_DOLLAR'] / df['ROOM_NIGHTS'].replace(0, np.nan)

    # Shift data to get previous month (M-1) values
    if group_cols:
        grouped = df.groupby(group_cols)
        df['prev_ADR'] = grouped['ADR'].shift(1)
        df['prev_RNS'] = grouped['ROOM_NIGHTS'].shift(1)
        df['prev_REV'] = grouped['REVENUE_DOLLAR'].shift(1)
    else:
        df['prev_ADR'] = df['ADR'].shift(1)
        df['prev_RNS'] = df['ROOM_NIGHTS'].shift(1)
        df['prev_REV'] = df['REVENUE_DOLLAR'].shift(1)

    # Calculate the Total Revenue Delta
    df['delta_REV'] = df['REVENUE_DOLLAR'] - df['prev_REV']

    # Price Effect: (Current ADR - Previous ADR) * Current RNS
    df['ADR_Contribution'] = (df['ADR'] - df['prev_ADR']) * df['ROOM_NIGHTS']

    # Volume Effect: (Current RNS - Previous RNS) * Previous ADR
    df['RNS_Contribution'] = (df['ROOM_NIGHTS'] - df['prev_RNS']) * df['prev_ADR']

    return df

# --- A. OVERALL LEVEL ---
overall_df = monthly_data.groupby(['MONTH']).agg({
    'REVENUE_DOLLAR': 'sum',
    'ROOM_NIGHTS': 'sum'
}).reset_index()
decomp_overall = calculate_decomposition(overall_df, [])

# --- B. CITY LEVEL ---
city_df = monthly_data.groupby(['MONTH', 'CITY']).agg({
    'REVENUE_DOLLAR': 'sum',
    'ROOM_NIGHTS': 'sum'
}).reset_index()
decomp_city = calculate_decomposition(city_df, ['CITY'])

# --- C. BRAND TYPE LEVEL ---
brand_df = monthly_data.groupby(['MONTH', 'BRAND_TYPE']).agg({
    'REVENUE_DOLLAR': 'sum',
    'ROOM_NIGHTS': 'sum'
}).reset_index()
decomp_brand = calculate_decomposition(brand_df, ['BRAND_TYPE'])

# --- D. PROPERTY LEVEL ---
# Directly using monthly_data as it's already at the property level
decomp_property = calculate_decomposition(monthly_data.copy(), ['PROPERTY_CODE'])

# Display settings for the final table
display_cols = [
    'MONTH', 'REVENUE_DOLLAR', 'delta_REV',
    'ADR_Contribution', 'RNS_Contribution'
]

# Quick tip: Calculate contribution percentage for easier analysis
decomp_overall['ADR_Pct'] = (decomp_overall['ADR_Contribution'] / decomp_overall['delta_REV'].abs()) * 100
decomp_overall['RNS_Pct'] = (decomp_overall['RNS_Contribution'] / decomp_overall['delta_REV'].abs()) * 100
```

***2. Identified for City, Brand Type, & Property Level***
```python
# Core Decomposition Function (MoM)
def get_revenue_decomposition(df, dimensions):
    # 1. Aggregate data by selected dimensions + Month
    agg_df = df.groupby(dimensions + ['MONTH']).agg({
        'REVENUE_DOLLAR': 'sum',
        'ROOM_NIGHTS': 'sum'
    }).reset_index()

    # 2. Sort data to ensure correct MoM shift calculations
    agg_df = agg_df.sort_values(dimensions + ['MONTH'])

    # 3. Calculate ADR (Average Daily Rate)
    agg_df['ADR'] = agg_df['REVENUE_DOLLAR'] / agg_df['ROOM_NIGHTS'].replace(0, np.nan)

    # 4. Get previous month's data (M-1) using shift
    # Grouping by dimensions ensures we compare the same entity month-over-month
    grouped = agg_df.groupby(dimensions)
    agg_df['prev_ADR'] = grouped['ADR'].shift(1)
    agg_df['prev_RNS'] = grouped['ROOM_NIGHTS'].shift(1)
    agg_df['prev_REV'] = grouped['REVENUE_DOLLAR'].shift(1)

    # 5. Calculate Contributions (Decomposition Formulas)
    # Price Effect: Impact of ADR changes
    agg_df['ADR_Contribution'] = (agg_df['ADR'] - agg_df['prev_ADR']) * agg_df['ROOM_NIGHTS']

    # Volume Effect: Impact of Room Night changes
    agg_df['RNS_Contribution'] = (agg_df['ROOM_NIGHTS'] - agg_df['prev_RNS']) * agg_df['prev_ADR']

    # 6. Calculate total revenue variance
    agg_df['Total_Rev_Change'] = agg_df['REVENUE_DOLLAR'] - agg_df['prev_REV']

    # 7. Select final columns and drop NaN rows (first month of each group)
    final_cols = dimensions + [
        'MONTH', 'REVENUE_DOLLAR', 'Total_Rev_Change',
        'ADR_Contribution', 'RNS_Contribution'
    ]
    return agg_df[final_cols].dropna(subset=['Total_Rev_Change'])

# --- EXECUTION ---

# 1. City Level
report_city = get_revenue_decomposition(monthly_data, ['CITY'])

# 2. Brand Type Level
report_brand = get_revenue_decomposition(monthly_data, ['BRAND_TYPE'])

# 3. Property Level
report_property = get_revenue_decomposition(monthly_data, ['PROPERTY_CODE'])
```

---

# Stage 5 - Property Benchmarking
## Method:
**Scoring model using:**
- Occupancy
- ADR
- Growth

**Logic:**

- Score = (0.4 * Occupancy) + (0.4 * ADR) + (0.2 * Growth)

**Properties categorized into:**
- Star 
- Strong
- Average
- Weak

## Objective:
Identify top assets and underperforming portfolio.

## Python Query:
```python
import pandas as pd
import numpy as np

# 1. Quarterly Conversion and Aggregation
bench_df = monthly_data.copy()
bench_df['QUARTER'] = bench_df['MONTH'].dt.to_timestamp().dt.to_period('Q')

# Aggregate data by Property and Quarter
q_data = bench_df.groupby(['PROPERTY_CODE', 'QUARTER']).agg({
    'REVENUE_DOLLAR': 'sum',
    'ROOM_NIGHTS': 'sum',
    'ROOMS_AVAILABLE': 'sum'
}).reset_index()

# 2. Calculate Key Metrics (OCC & ADR)
q_data['OCC'] = q_data['ROOM_NIGHTS'] / q_data['ROOMS_AVAILABLE'].replace(0, np.nan)
q_data['ADR'] = q_data['REVENUE_DOLLAR'] / q_data['ROOM_NIGHTS'].replace(0, np.nan)

# 3. Calculate Revenue Growth (QoQ)
q_data = q_data.sort_values(['PROPERTY_CODE', 'QUARTER'])
q_data['PREV_REV'] = q_data.groupby('PROPERTY_CODE')['REVENUE_DOLLAR'].shift(1)
q_data['GROWTH'] = (q_data['REVENUE_DOLLAR'] - q_data['PREV_REV']) / q_data['PREV_REV'].replace(0, np.nan)

# Drop initial periods where growth comparison is unavailable
q_data = q_data.dropna(subset=['GROWTH'])

# 4. Percentile Weighted Scoring (Calculated per Quarter)
def calculate_star_rating(group):
    # Calculate Percentile Rank (Scale 0-100) for each metric
    group['SCORE_OCC'] = group['OCC'].rank(pct=True) * 100
    group['SCORE_ADR'] = group['ADR'].rank(pct=True) * 100
    group['SCORE_GROWTH'] = group['GROWTH'].rank(pct=True) * 100

    # Apply weighted scoring logic
    group['FINAL_SCORE'] = (
        (group['SCORE_OCC'] * 0.40) +
        (group['SCORE_ADR'] * 0.40) +
        (group['SCORE_GROWTH'] * 0.20)
    )
    return group

# Apply ranking per quarter to ensure a fair "apples-to-apples" comparison
benchmarking_final = q_data.groupby('QUARTER', group_keys=False).apply(calculate_star_rating)

# 5. Rating Classification
def classify_by_distribution(group):
    percentile_rank = group['FINAL_SCORE'].rank(pct=True)
    cond = [
        (percentile_rank > 0.90),
        (percentile_rank > 0.70),
        (percentile_rank > 0.30),
        (percentile_rank <= 0.30)
    ]
    choices = ['Star', 'Strong', 'Average', 'Weak']

    group['RATING'] = np.select(cond, choices, default='N/A')
    return group

# Aplied for each Quarters
benchmarking_final = benchmarking_final.groupby('QUARTER', group_keys=False).apply(classify_by_distribution)

# --- Display Final Results ---
target_quarter = pd.Period('2024Q3', freq='Q')
final_report = benchmarking_final[benchmarking_final['QUARTER'] == target_quarter]

print(f"=== Benchmarking Report (Distribution Based) Quarter: {target_quarter} ===")
print(final_report[['PROPERTY_CODE','ADR', 'OCC', 'GROWTH', 'FINAL_SCORE', 'RATING']].sort_values('FINAL_SCORE', ascending=False))
```

---

# Stage 6 - Customer Prioritization
## Method:
### 1. Customer Segmentation Logic

Customer segments were created using three dimensions:
- Travel Purpose (Business / Leisure)
- Age Group (20–24, 25–34, 35–44, 45–54, 55–60)
- User Type (New User / Repeat User)

This generated 20 strategic customer segments for commercial prioritization.

| No. | Travel Purpose | Age Group | User Type |
| ----------- | ----------- | ----------- | ----------- |
| 1 | Business | 20-24 | New User |
| 2 | Business | 25-34 | New User |
| 3 | Business | 35-44 | New User |
| 4 | Business | 45-54 | New User |
| 5 | Business | 55-60 | New User |
| 6 | Business | 20-24 | Repeat User |
| 7 | Business | 25-34 | Repeat User |
| 8 | Business | 35-44 | Repeat User |
| 9 | Business | 45-54 | Repeat User |
| 10 | Business | 55-60 | Repeat User |
| 11 | Leisure | 20-24 | New User |
| 12 | Leisure | 25-34 | New User |
| 13 | Leisure | 35-44 | New User |
| 14 | Leisure | 45-54 | New User |
| 15 | Leisure | 55-60 | New User |
| 16 | Leisure | 20-24 | Repeat User |
| 17 | Leisure | 25-34 | Repeat User |
| 18 | Leisure | 35-44 | Repeat User |
| 19 | Leisure | 45-54 | Repeat User |
| 20 | Leisure | 55-60 | Repeat User |

### 2. The 3x3 Prioritization Framework

Each segment was ranked using percentile distribution to ensure a balanced classification into Low, Medium, and High categories.
1. **Current Value Score**, Calculates the segment's immediate economic contribution.
   - ***Metrics:*** Revenue (Weighted 50%) + ARPU (Weighted 50%)
   - ***Logic:*** Score = (0.5 * Revenue) + (0.5 * ARPU)
2. **Future Potential Score**, Forecasts the segment's long-term scalability and loyalty.
   - ***Metrics:*** Revenue Growth (Weighted 40%) + Retention Rate (Weighted 60%)
   - ***Logic:*** Score = (0.4 * Growth) + (0.6 * Retention)
   
### 3. Strategic Classification Matrix

| Future Potential | Current Value | Priority Label |
| ---------------- | ------------- | -------------- |
| High | Low | Growth Bet |
| High | Medium | Emerging Core |
| High | High | Star Segment | 
| Medium | Low | Test & Nurture |
| Medium | Medium | Stable Segment |
| Medium | High | Cash Cow |
| Low | Low | Low Priority |
| Low | Medium |Defend Selectively |
| Low | High | Mature Cash |

## Python Query:
***1. Create Segments***
```python
import pandas as pd
import numpy as np

# Create a copy to avoid modifying the original dataframe
df_user_detail = user_df.copy()

# Age Binning Process
# Define age ranges and their corresponding labels
age_bins = [20, 25, 35, 45, 55, 61]
age_labels = ['20-24', '25-34', '35-44', '45-54', '55-60']

df_user_detail['AGE_GROUP'] = pd.cut(
    df_user_detail['USER_AGE'],
    bins=age_bins,
    labels=age_labels,
    right=False
).astype(str)

# Create 'USER_SEGMENT' Column
# Combine multiple dimensions into a single unique segment label
df_user_detail['USER_SEGMENT'] = (
    df_user_detail['TRAVEL_PURPOSE'] + " | " +
    df_user_detail['AGE_GROUP'] + " | " +
    df_user_detail['USER_TYPE']
)

# Select Columns for Final Output
df_final_detail = df_user_detail[[
    'USER_ID',
    'TRAVEL_PURPOSE',
    'USER_AGE',
    'AGE_GROUP',
    'USER_TYPE',
    'USER_SEGMENT'
]]
```

***2. Find Revenue, ARPU, Growth, and Retention Rate for each segments***
```python
import pandas as pd
import numpy as np

# 1. Join bookings with user segmentation (assuming df_final_detail is ready)
# Merging revenue data with segment labels based on USER_ID
full_data = pd.merge(bookings_df2, df_final_detail[['USER_ID', 'USER_SEGMENT']], on='USER_ID', how='left')

# 2. Date preprocessing
full_data['CHECK_IN_DATE'] = pd.to_datetime(full_data['CHECK_IN_DATE'])
full_data['YEAR'] = full_data['CHECK_IN_DATE'].dt.year
full_data['MONTH'] = full_data['CHECK_IN_DATE'].dt.month
full_data['QUARTER'] = full_data['CHECK_IN_DATE'].dt.to_period('Q')

# 3. Filter data for Year-Over-Year (YoY) Growth (Jan-Oct period for 2023 & 2024)
data_23 = full_data[(full_data['YEAR'] == 2023) & (full_data['MONTH'] <= 9)]
data_24 = full_data[(full_data['YEAR'] == 2024) & (full_data['MONTH'] <= 9)]

# 4. Calculate 2024 YTD Metrics
metrics_24 = data_24.groupby('USER_SEGMENT').agg({
    'REVENUE_DOLLAR': 'sum',
    'USER_ID': 'nunique'
}).rename(columns={'REVENUE_DOLLAR': 'REVENUE_2024', 'USER_ID': 'UNIQUE_USERS_2024'})

# ARPU (Average Revenue Per User) calculation
metrics_24['ARPU'] = metrics_24['REVENUE_2024'] / metrics_24['UNIQUE_USERS_2024']

# 5. Calculate 2023 YTD Metrics for Growth comparison
metrics_23 = data_23.groupby('USER_SEGMENT').agg({'REVENUE_DOLLAR': 'sum'}).rename(columns={'REVENUE_DOLLAR': 'REVENUE_2023'})

# 6. Merge and calculate Revenue Growth
final_metrics = pd.merge(metrics_24, metrics_23, on='USER_SEGMENT', how='left')
final_metrics['GROWTH_REVENUE'] = (final_metrics['REVENUE_2024'] - final_metrics['REVENUE_2023']) / final_metrics['REVENUE_2023'].replace(0, np.nan)

# 7. Retention Analysis Logic
def calculate_retention(df):
    # Identify unique users per quarter in 2024
    q1_users = set(df[df['QUARTER'] == '2024Q1']['USER_ID'].unique())
    q2_users = set(df[df['QUARTER'] == '2024Q2']['USER_ID'].unique())
    q3_users = set(df[df['QUARTER'] == '2024Q3']['USER_ID'].unique())

    # User base size per period
    a = len(q1_users)
    b = len(q2_users)

    # Calculate retention rates (overlapping users between consecutive quarters)
    # x_rate: Q1 users returning in Q2
    # y_rate: Q2 users returning in Q3
    x_rate = len(q1_users.intersection(q2_users)) / a if a > 0 else 0
    y_rate = len(q2_users.intersection(q3_users)) / b if b > 0 else 0

    # Weighted Average Retention to represent overall stability
    if (a + b) > 0:
        weighted_retention = (x_rate * a + y_rate * b) / (a + b)
    else:
        weighted_retention = 0

    return pd.Series({'RETENTION_RATE': weighted_retention})

# Apply retention logic to each segment
retention_df = data_24.groupby('USER_SEGMENT').apply(calculate_retention).reset_index()

# 8. Consolidate Final Segment Report
segment_final_report = pd.merge(
    final_metrics.reset_index(),
    retention_df,
    on='USER_SEGMENT',
    how='left'
)

# Column selection and final cleanup
output_columns = [
    'USER_SEGMENT',
    'REVENUE_2024',
    'ARPU',
    'GROWTH_REVENUE',
    'RETENTION_RATE'
]

segment_final_report = segment_final_report[output_columns].fillna(0)
segment_final_report = segment_final_report.sort_values('REVENUE_2024', ascending=False).reset_index(drop=True)
```

***3. Create Priority Labels***
```python
import pandas as pd
import numpy as np

# 1. Use the segment report from the previous step
df_matrix = segment_final_report.copy()

# 2. Calculate Scoring for Current and Future Value
# Current Value: Weighted blend of total revenue and individual ARPU
# Future Value: Weighted blend of growth momentum and customer retention
df_matrix['CURRENT_VALUE_SCORE'] = (0.5 * df_matrix['REVENUE_2024']) + (0.5 * df_matrix['ARPU'])
df_matrix['FUTURE_VALUE_SCORE'] = (0.4 * df_matrix['GROWTH_REVENUE']) + (0.6 * df_matrix['RETENTION_RATE'])

# 3. Categorize into Low, Medium, and High based on Percentiles (Terciles)
def get_3x3_category(series):
    # Using qcut to divide segments into three equal-sized groups (33rd & 66th percentiles)
    return pd.qcut(series, q=[0, 0.33, 0.66, 1.0], labels=['Low', 'Medium', 'High'])

df_matrix['CURRENT_CAT'] = get_3x3_category(df_matrix['CURRENT_VALUE_SCORE'])
df_matrix['FUTURE_CAT'] = get_3x3_category(df_matrix['FUTURE_VALUE_SCORE'])

# 4. 3x3 Matrix Label Mapping
# Dictionary mapping for strategic categorization based on (Future_Value, Current_Value)
matrix_labels = {
    ('High', 'Low'): 'Growth Bet',
    ('High', 'Medium'): 'Emerging Core',
    ('High', 'High'): 'Star Segment',
    ('Medium', 'Low'): 'Test & Nurture',
    ('Medium', 'Medium'): 'Stable Segment',
    ('Medium', 'High'): 'Cash Cow',
    ('Low', 'Low'): 'Low Priority',
    ('Low', 'Medium'): 'Defend Selectively',
    ('Low', 'High'): 'Mature Cash'
}

# Apply strategic labels based on the category combinations
df_matrix['PRIORITY_LABEL'] = df_matrix.apply(
    lambda x: matrix_labels.get((x['FUTURE_CAT'], x['CURRENT_CAT'])), axis=1
)

# 5. Finalize the Matrix Table
final_segment_matrix = df_matrix[[
    'USER_SEGMENT', 'CURRENT_VALUE_SCORE', 'FUTURE_VALUE_SCORE',
    'CURRENT_CAT', 'FUTURE_CAT', 'PRIORITY_LABEL'
]]
```

---

# Stage 7 - Dashboard Development


![](https://github.com/user-attachments/assets/0a42aa87-95dd-4b9f-81a7-6f344cb21dd5)



---

# Stage 8 - Key Insights & Business Recommendations

Based on the dashboard analysis, several strategic insights were identified across revenue performance, property portfolio, and customer segmentation.

---

## 1. Executive Overview - Key Insights

In 2024, the business demonstrated robust topline momentum with a **YTD Revenue of USD 285.1K (+27.2% YoY)**. However, a deep dive into the drivers reveals that growth is heavily **volume-led** (increased bookings) rather than **yield-driven** (ADR/Pricing). The strategic focus must now shift from pure demand acquisition to **Portfolio Optimization** and **Yield Management**.

---

## 2. Revenue Driver Analysis - Key Insights

**Current State:** Revenue growth is primarily fueled by a surge in **Room Nights (RNs)**, while ADR has remained stagnant or softened across all cities and brands.

- **Monetization Ceiling:** Most ADR values remain **under USD 15**. While this ensures **competitiveness** in the budget segment, it **limits the profit margin** per booking.
- **The Scalability Risk:** Relying solely on volume is risky. If market demand plateaus, revenue will stall because the pricing lever is currently underutilized.
- **City Spotlight:** Yogyakarta is the clear winner with **+38% YoY revenue growth**, driven by a massive **+37% surge in bookings**. It is the primary candidate for a **"Price Elasticity Test"** to improve margins.

### Strategic Mandate
Transition from **"Market Capture"** to **"Yield Optimization"**. Implement dynamic pricing models to capture upside during high-demand periods.

---

## 3. Property Performance Benchmarking – Key Insights

**Current State:** Portfolio health is uneven. While total revenue is up, more than **50% of properties** show **flat or negative growth**.

- **Performance Concentration:** Growth is carried by a small group of **Star Properties (approx. 10%)**.
- **The Median Reality:** The portfolio's **Median Growth is negative (-0.3% to -2.4%)**, indicating that the "Average" property is actually struggling to keep up with last year's performance.
- **Dynamic Leadership:** Leadership at the city level is **seasonal**. Malang overtook Yogyakarta in Q3, suggesting that "Best Practice" playbooks should be **localized and seasonal rather than static**.

### Strategic Mandate
Launch a **"Portfolio Remediation"** program. Instead of just adding new properties, focus on **moving the "Weak" and "Average" assets toward the "Strong" benchmark** through operational standardized playbooks.


---
## 4. Customer Segment Prioritization – Key Insights

**Current State:** Customer revenue is well diversified across **20 segments**, but long-term value is concentrated in a smaller group of high-performing business traveler segments.

- **Value Leaders:** Highest current value came from **Business | 45–54 | New User**, driven by strong revenue and ARPU.
- **Future Winners:** Highest future potential came from **Business | 55–60 | Repeat User**, supported by strong retention and growth.
- **Segment Pattern:** Most **Star, Cash Cow, and Mature Cash** segments were dominated by **Business-purpose users aged 25–60**.
- **Selective Leisure Opportunity:** Several leisure repeat-user segments showed strong upside, proving growth is not limited to business travelers alone.

### Strategic Mandate

Launch a **"Precision Customer Growth"** strategy. Prioritize retention and upsell programs for high-value business travelers, while building targeted conversion campaigns to turn promising leisure and new-user segments into repeat profitable customers.

---

# Final Management Priority Roadmap

## 🔴 High Priority (Immediate Impact)
- **Yield Optimization:** Implement dynamic pricing to break the $15 ADR ceiling during peak last-minute demand (Day 0 lead time).
- **Yogyakarta Expansion:** Aggressively scale supply and marketing in Yogyakarta to capitalize on its high conversion rate.
- **Turnaround Plans:** Identify "Weak" properties with high potential and implement 90-day recovery plans.

## 🟡 Medium Priority (Sustainable Growth)
- **Segment-Specific UI/UX:** Tailor the booking experience for mature business users (efficiency/invoice-focused) vs. leisure users (discovery/promo-focused).
- **Brand Re-invigoration:** Address the slowing growth of the core RedDoorz brand by diversifying the brand mix.

## 🟢 Ongoing (Operational Excellence)
- **Decision Intelligence:** Transition from tracking "Total Revenue" to "Revenue Mix" (RN Growth vs. ADR Growth).
- **Median Health Monitoring:** Use Median Growth as the primary KPI for portfolio health to avoid being misled by a few top-performing assets.

---

# Executive Takeaway

Phase 1 was about **Demand Recovery**. Phase 2 must be about **Asset Productivity**. To sustain long-term growth, the business must pivot from 'Cheap Volume' to Strategic Monetization and Operational Equalization across the property network.
