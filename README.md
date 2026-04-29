# RedDoorz Performance Analytics Project
<img width="736" height="520" alt="WhatsApp Image 2026-04-24 at 10 32 17" src="https://github.com/user-attachments/assets/b9593280-7ce9-4952-a464-bb58db8a7267" />

---

# Table of Content

- [Business Understanding](#business-understanding)
- [Data Source](#data-source)
- [Stages of Analysis](#stages-of-analysis)
  - [Stage 1 - Data Preparation](#stage-1---data-preparation)
  - [Stage 2 - Feature Engineering](#stage-2---feature-engineering)
  - [Stage 3 - Exploratory Data Analysis](#stage-3---exploratory-data-analysis)
  - [Stage 4 - Revenue Driver Analysis (Decomposition Framework)](#stage-4---revenue-driver-analysis-decomposition-framework)
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
- Stage 2 - Feature Engineering
- Stage 3 - Exploratory Data Analysis
- Stage 4 - Revenue Driver Analysis
- Stage 5 - Property Benchmarking
- Stage 6 - Customer Prioritization
- Stage 7 - Dashboard Development
- Stage 8 - Analysis

---

# Stage 1 - Data Preparation
I implemented a multi-stage cleaning pipeline to eliminate noise and ensure data reliability:
1. **Data Consistency:** Validated date formats and handled missing or redundant records to maintain a clean baseline.
2. **Logic Auditing:** Removed operational anomalies and chronological inconsistencies, such as:
   - Stay durations with negative values (Check-out < Check-in).
   - Bookings recorded prior to a property’s official opening date (Cohort Date).
   - General data entry errors where booking dates followed check-in dates.

---

# Stage 2 - Feature Engineering
I engineered a dynamic analytical framework to convert raw transactions into actionable Business KPIs:
- **Occupancy Rate (OCC) Calculation:** Developed a sophisticated logic to calculate monthly occupancy by factoring in property-specific inventory and "Active Days" (adjusting for properties that opened mid-month).
- **Revenue Performance Metrics:** Automated the calculation of ***ADR*** (Average Daily Rate) and ***RevPAR*** (Revenue per Available Room) to benchmark pricing efficiency.
- **Financial Filtering:** Filtered out negative revenue outliers to ensure the financial trends reflect actual realized growth.

---

# Stage 3 - Exploratory Data Analysis
## 1. Revenue Trend (2020-2024)
<img width="1333" height="549" alt="Revenue Trend" src="https://github.com/user-attachments/assets/01246548-c8a4-4fe6-95d7-e60499ced9eb" />

**Brief Insight** : The chart illustrates a robust ***long-term bullish trend*** in monthly revenue, growing from approximately $2,500 in 2020 to over $33,000 by mid-2024. A key insight is the significant momentum gained after 2022, where revenue consistently outperformed the historical average. This steady progression indicates ***strong market demand*** and ***effective operational scaling*** over the four-year period.

---

## 2. Total Bookings by Stay Duration
<img width="639" height="476" alt="Stay Duration Distributions" src="https://github.com/user-attachments/assets/b0c23e44-6699-4e46-abb3-5981feee78d5" />

**Brief Insight** : The bookings are classified as ***short-term***, as the maximum stay duration is only ***3 days***. Furthermore, the volume is evenly distributed across all stay durations.

---

## 3. Total Bookings by Lead Time
<img width="872" height="553" alt="Lead Time Distributions" src="https://github.com/user-attachments/assets/502c19fc-9450-48d0-b2b2-d127593f2a5e" />

**Brief Insight** : The distribution reveals a significant ***right-skewed pattern***, where the vast majority of bookings are made with a ***0 to 2-day lead time***. With over 18,000 bookings occurring on day zero, the business exhibits a high reliance on ***spontaneous or last-minute travelers***. This insight is critical for operational readiness and suggests that ***real-time dynamic pricing strategies*** are essential to capture maximum value from this high-volume, short-notice segment.

---

## 4. Bookings Distribution by ADR
<img width="863" height="553" alt="ADR Distributions" src="https://github.com/user-attachments/assets/09176731-d027-4e13-a1ef-65e407eb8584" />

**Brief Insight** : The ADR distribution reveals a ***multi-modal pattern*** with a heavy concentration in the ***$2 – $5 range***, identifying it as ***the primary market volume driver***. The sharp decline in frequency beyond ***$15*** suggests a critical price ceiling for the majority of the customer base.

---

# Stage 4 - Revenue Driver Analysis (Decomposition Framework)
This stage involves a deep dive into the underlying mechanics of the revenue growth. By implementing a **Revenue Decomposition Framework**, i transition from simply tracking total revenue to understanding the specific "levers" driving the financial performance.

## Method & Analytical Focus:
The analysis utilizes a Month-over-Month (MoM) variance model to isolate two primary growth drivers:
- **The Volume Effect (Room Night Contribution):** Quantifies how much revenue change is driven by the increase or decrease in the number of rooms sold.
- **The Pricing Effect (ADR Contribution):** Isolates the impact of the pricing strategy by measuring how changes in the Average Daily Rate affect the bottom line.

## Multidimensional Granularity:
To provide actionable intelligence for different business units, the decomposition is executed across four key levels:
- **Overall Portfolio:** Evaluates the aggregate health and growth stability of the entire business.
- **City & Market Level:** Identifies which geographic regions are winning on volume versus those maximizing premium pricing.
- **Brand Segment:** Analyzes performance differences across various property categories (e.g., Budget vs. Premium).
- **Individual Property:** Provides specific data for property managers to optimize their local sales and pricing tactics.

## **Business Objective:**
The core goal of this stage is to answer a critical business question: **"Which operational levers drive revenue: room nights or ADR?"** By identifying whether revenue is coming from increased market share (Volume) or improved monetization (Price), management can determine if the current strategy requires more aggressive marketing to fill rooms or dynamic pricing adjustments to capture higher margins.

---

# Stage 5 - Property Benchmarking
To drive strategic asset management, this stage implements a **multi-criteria scoring model** that evaluates each property’s health relative to its peers. By shifting from raw metrics to a percentile-based ranking, i ensure a fair "apples-to-apples" comparison regardless of seasonal fluctuations.

## The Scoring Logic:
I utilize a weighted formula to calculate a **Final Performance Score:**

***Final Score = (0.4 * Occupancy) + (0.4 * ADR) + (0.4 * Growth)***

***Note:*** The weight distribution (40/40/20) is designed to prioritize a balanced Revenue Management strategy, optimizing for both market volume (Occupancy) and pricing power (ADR) while maintaining a baseline for growth momentum.

## Strategic Classification:
Properties are categorized into a four-tier performance matrix based on their distribution:
- **Star (Top 10%):** Elite performers excelling in both yield and volume.
- **Strong (Top 20% - 30%):** High-performing assets with consistent market presence.
- **Average (Middle 40%):** Stable properties maintaining standard market expectations.
- **Weak (Bottom 30%):** Underperforming assets requiring immediate operational intervention or strategic review.

## Business Objective:
This model serves as a **Portfolio Health Check**, allowing management to identify "Top Assets" for best-practice sharing and "Underperforming Clusters" that may require price adjustments, facility upgrades, or marketing support.

---

# Stage 6 - Customer Prioritization
To optimize marketing spend and operational focus, this stage implements a **3x3 Strategic Prioritization Framework**. By segmenting users based on demographics, behavior, and loyalty, the business can move away from a "one-size-fits-all" approach and instead deploy targeted strategies for different high-value customer groups.

## Multi-Dimensional Segmentation:
Users are categorized into 20 distinct segments using a combination of:
- **Travel Purpose:** Distinguishing between consistent Business needs and seasonal Leisure trends.
- **Life Stage (Age Groups):** Ranging from Gen Z (20–24) to Mature Travelers (55–60).
- **User Loyalty:** Differentiating between the cost of acquiring New Users versus the value of Repeat Users.

## The Prioritization Framework:
Each segment is evaluated through two critical lenses:
- **Current Value (50% Revenue | 50% ARPU):** Measures the immediate "cash engine" of the segment. It balances the total volume of money coming in with the individual spending power of each user.
- **Future Potential (40% Growth | 60% Retention):** Forecasts long-term sustainability. It prioritizes segments that are not only growing in size but, more importantly, are consistently returning to the platform.

## Strategic Outcomes:
The result is a Classification Matrix that guides commercial action:
- **Star Segments:** High value, high growth. These are the top priorities for investment.
- **Cash Cows:** High current value but low growth. These segments should be "milked" for efficiency to fund other growth bets.
- **Growth Bets:** High potential despite low current value; these are the future "Stars."

---

# Stage 7 - Dashboard Development

<video src="https://github.com/user-attachments/assets/0a42aa87-95dd-4b9f-81a7-6f344cb21dd5" controls="controls" style="max-width: 100%;">
  Your browser does not support the video tag.
</video>

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

- **Value Leaders:** Highest current value came from **'Business, 45–54, New User'**, driven by strong revenue and ARPU.
- **Future Winners:** Highest future potential came from **'Business, 55–60, Repeat User'**, supported by strong retention and growth.
- **Segment Pattern:** Most **Star, Cash Cow, and Mature Cash** segments were dominated by **Business-purpose users aged 25–60**.
- **Selective Leisure Opportunity:** Several leisure repeat-user segments showed strong upside, proving growth is not limited to business travelers alone.

### Strategic Mandate

Launch a **"Precision Customer Growth"** strategy. Prioritize retention and upsell programs for high-value business travelers, while building targeted conversion campaigns to turn promising leisure and new-user segments into repeat profitable customers.

---

# Final Management Priority Roadmap

## High Priority (Immediate Impact)
- **Yield Optimization:** Implement dynamic pricing to break the $15 ADR ceiling during peak last-minute demand (Day 0 lead time).
- **Yogyakarta Expansion:** Aggressively scale supply and marketing in Yogyakarta to capitalize on its high conversion rate.
- **Turnaround Plans:** Identify "Weak" properties with high potential and implement 90-day recovery plans.

## Medium Priority (Sustainable Growth)
- **Segment-Specific UI/UX:** Tailor the booking experience for mature business users (efficiency/invoice-focused) vs. leisure users (discovery/promo-focused).
- **Brand Re-invigoration:** Address the slowing growth of the core RedDoorz brand by diversifying the brand mix.

## Ongoing (Operational Excellence)
- **Decision Intelligence:** Transition from tracking "Total Revenue" to "Revenue Mix" (RN Growth vs. ADR Growth).
- **Median Health Monitoring:** Use Median Growth as the primary KPI for portfolio health to avoid being misled by a few top-performing assets.

---

# Executive Takeaway

Phase 1 was about **Demand Recovery**. Phase 2 must be about **Asset Productivity**. To sustain long-term growth, the business must pivot from 'Cheap Volume' to Strategic Monetization and Operational Equalization across the property network.

---
# Technical Resources
For a detailed step-by-step technical breakdown, including Python scripts for all 6 stages, please refer to:

**Source Code:** [Google Colab Notebook](https://colab.research.google.com/drive/1VhUIdYZ4fwIOt8FNcihVkQzjXW9d8NYQ?usp=sharing)
