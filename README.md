# 🛍️ Customer Shopping Behavior Analysis

> **End-to-end data analytics project** covering Python EDA, PostgreSQL querying, and Power BI dashboarding on real retail customer data.

---

## 📌 Overview

This project analyses the shopping behaviour of **3,900 retail customers** to uncover patterns in spending, product preferences, subscription impact, and discount effectiveness. The goal is to help business stakeholders make data-driven decisions on marketing, inventory, and customer retention.

The project follows a complete analytics workflow:

**Data Loading → Cleaning & EDA (Python) → SQL Analysis (PostgreSQL) → Dashboard (Power BI)**

---

## 📂 Dataset

| Field | Details |
|---|---|
| **File** | `customer_shopping_behavior.csv` |
| **Records** | 3,900 customers |
| **Columns** | 18 features |
| **Source** | Synthetic retail dataset |

### Key Columns

| Column | Description |
|---|---|
| `Customer ID` | Unique identifier per customer |
| `Age` | Customer age |
| `Gender` | Male / Female |
| `Item Purchased` | Product bought (e.g., Shirt, Jacket, Boots) |
| `Category` | Clothing, Footwear, Accessories, Outerwear |
| `Purchase Amount (USD)` | Transaction value |
| `Season` | Spring / Summer / Fall / Winter |
| `Review Rating` | Customer rating (1–5) |
| `Subscription Status` | Yes / No |
| `Discount Applied` | Whether a discount was used |
| `Previous Purchases` | Count of past transactions |
| `Shipping Type` | Standard, Express, Free Shipping, etc. |
| `Payment Method` | Credit Card, PayPal, Venmo, Cash, etc. |
| `Frequency of Purchases` | Weekly, Monthly, Annually, etc. |

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| **Python** (Pandas, Matplotlib, Seaborn) | Data loading, cleaning, and exploratory analysis |
| **Jupyter Notebook** | Interactive Python environment |
| **PostgreSQL** | Structured querying and business logic |
| **Power BI** | Interactive dashboard and visual storytelling |
| **GitHub** | Version control and project sharing |

---

## 🔄 Project Workflow

### Step 1 — Data Loading & Inspection (Python)

- Loaded the CSV using **Pandas**
- Inspected shape, data types, and null values
- Identified missing values in `Review Rating` column
- Checked for duplicate records

```python
import pandas as pd

df = pd.read_csv("customer_shopping_behavior.csv")
print(df.shape)        # (3900, 18)
print(df.info())
print(df.isnull().sum())
```

---

### Step 2 — Data Cleaning (Python)

- Filled missing `Review Rating` values with the column median
- Standardised column names (lowercase, underscores)
- Converted `Subscription Status` and `Discount Applied` to binary flags
- Created a new `Age Group` column for segmentation:
  - `18–25` → Young Adults
  - `26–40` → Adults
  - `41–55` → Middle-Aged
  - `56+` → Seniors

```python
df['review_rating'].fillna(df['review_rating'].median(), inplace=True)

bins = [17, 25, 40, 55, 100]
labels = ['Young Adults (18-25)', 'Adults (26-40)', 'Middle-Aged (41-55)', 'Seniors (56+)']
df['age_group'] = pd.cut(df['age'], bins=bins, labels=labels)
```

---

### Step 3 — Exploratory Data Analysis (Python)

Key analyses performed with visualisations:

- **Revenue by Gender** — Bar chart comparing male vs. female spending
- **Purchase Distribution** — Histogram of transaction amounts
- **Top Categories** — Bar chart by category revenue
- **Seasonal Trends** — Revenue breakdown across Spring, Summer, Fall, Winter
- **Rating Distribution** — Histogram of review ratings
- **Subscription vs. Spend** — Boxplot comparing subscriber and non-subscriber purchase amounts
- **Correlation Heatmap** — Relationships between numeric features

```python
import seaborn as sns
import matplotlib.pyplot as plt

# Revenue by category
df.groupby('category')['purchase_amount'].sum().sort_values().plot(kind='barh')
plt.title('Total Revenue by Category')
plt.show()
```

---

### Step 4 — SQL Analysis (PostgreSQL)

The cleaned dataset was loaded into a PostgreSQL database table named `customer`. Ten business questions were answered using SQL:

| # | Business Question | SQL Concept Used |
|---|---|---|
| Q1 | Total revenue by gender | `GROUP BY`, `SUM` |
| Q2 | Discount users spending above average | Subquery, `WHERE` filter |
| Q3 | Top 5 products by average review rating | `GROUP BY`, `AVG`, `ORDER BY`, `LIMIT` |
| Q4 | Avg purchase: Standard vs. Express shipping | `WHERE IN`, `AVG` |
| Q5 | Subscribers vs. non-subscribers — spend & revenue | `GROUP BY`, `COUNT`, `AVG`, `SUM` |
| Q6 | Top 5 products by discount usage rate | `CASE WHEN`, percentage calculation |
| Q7 | Customer segmentation: New / Returning / Loyal | `CTE`, `CASE WHEN` |
| Q8 | Top 3 products per category | `CTE`, `ROW_NUMBER()`, Window Function |
| Q9 | Repeat buyers (>5 purchases) and subscription rate | `WHERE`, `GROUP BY` |
| Q10 | Revenue contribution by age group | `GROUP BY`, `SUM`, `ORDER BY` |

#### Sample Query — Customer Segmentation (Q7)

```sql
WITH customer_type AS (
  SELECT
    customer_id,
    previous_purchases,
    CASE
      WHEN previous_purchases = 1 THEN 'New'
      WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
      ELSE 'Loyal'
    END AS customer_segment
  FROM customer
)
SELECT customer_segment, COUNT(*) AS "Number of Customers"
FROM customer_type
GROUP BY customer_segment;
```

#### Sample Query — Top 3 Products per Category (Q8)

```sql
WITH item_counts AS (
  SELECT category, item_purchased,
         COUNT(customer_id) AS total_orders,
         ROW_NUMBER() OVER (PARTITION BY category ORDER BY COUNT(customer_id) DESC) AS item_rank
  FROM customer
  GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts
WHERE item_rank <= 3;
```

---

### Step 5 — Power BI Dashboard

An interactive Power BI dashboard was built using `customer_behavior_dashboard.pbix`.

#### Dashboard Pages

| Page | Content |
|---|---|
| **Overview** | Total revenue, total customers, avg purchase, avg rating KPI cards |
| **Customer Segmentation** | New / Returning / Loyal breakdown; subscriber vs. non-subscriber spend |
| **Product Analysis** | Top products by revenue and rating; category-wise performance |
| **Seasonal & Demographic** | Revenue by season, age group, and gender |
| **Discount & Shipping** | Discount impact on revenue; shipping type comparison |

#### Key DAX Measures Used

```dax
Total Revenue = SUM(customer[purchase_amount])

Avg Purchase Amount = AVERAGE(customer[purchase_amount])

Subscriber Revenue % =
  DIVIDE(
    CALCULATE([Total Revenue], customer[subscription_status] = "Yes"),
    [Total Revenue]
  )

Discount Revenue =
  CALCULATE([Total Revenue], customer[discount_applied] = "Yes")
```

---

## 📊 Key Findings & Insights

| # | Insight |
|---|---|
| 1 | **Male customers** generate slightly higher total revenue than female customers despite similar average spend |
| 2 | **Subscribed customers** show higher average spend and account for a disproportionate share of total revenue |
| 3 | **Loyal customers** (>10 previous purchases) are also more likely to hold active subscriptions |
| 4 | **Clothing** is the highest-revenue category; **Outerwear** has the fewest transactions but highest avg ticket |
| 5 | **Fall and Winter** seasons drive peak revenue — ideal windows for promotional campaigns |
| 6 | Customers who used **discounts** but still spent above average represent a high-value retention opportunity |
| 7 | **Standard shipping** and **Express shipping** show virtually equal average purchase amounts — shipping preference is not a spend predictor |
| 8 | **Seniors (56+)** contribute the highest revenue by age group, making them a priority retention segment |

---

## 📁 Project Files

```
customer-shopping-behavior-analysis/
│
├── customer_shopping_behavior.csv              # Raw dataset (3,900 rows, 18 columns)
├── Customer_Shopping_Behavior_Analysis.ipynb   # Python EDA and data cleaning notebook
├── customer_shopping_behavior_project.sql      # 10 PostgreSQL business queries
├── customer_behavior_dashboard.pbix            # Power BI interactive dashboard
└── README.md                                   # Project documentation (this file)
```

---

## ▶️ How to Run

### Python / Jupyter Notebook

1. Clone or download this repository
2. Install dependencies:
   ```bash
   pip install pandas matplotlib seaborn jupyter
   ```
3. Launch Jupyter:
   ```bash
   jupyter notebook Customer_Shopping_Behavior_Analysis.ipynb
   ```
4. Run all cells from top to bottom

---

### PostgreSQL

1. Open **pgAdmin** or any PostgreSQL client
2. Create a new database (e.g., `shopping_db`)
3. Import the CSV into a table named `customer`:
   ```sql
   CREATE TABLE customer (
     customer_id       INT,
     age               INT,
     gender            VARCHAR(10),
     item_purchased    VARCHAR(50),
     category          VARCHAR(30),
     purchase_amount   NUMERIC,
     location          VARCHAR(50),
     size              VARCHAR(5),
     color             VARCHAR(20),
     season            VARCHAR(10),
     review_rating     NUMERIC,
     subscription_status VARCHAR(5),
     shipping_type     VARCHAR(30),
     discount_applied  VARCHAR(5),
     promo_code_used   VARCHAR(5),
     previous_purchases INT,
     payment_method    VARCHAR(30),
     frequency_of_purchases VARCHAR(30)
   );

   COPY customer FROM '/path/to/customer_shopping_behavior.csv'
   DELIMITER ',' CSV HEADER;
   ```
4. Open `customer_shopping_behavior_project.sql` and run queries one by one

---

### Power BI Dashboard

1. Open **Power BI Desktop** (free download from Microsoft)
2. Open `customer_behavior_dashboard.pbix`
3. If prompted, update the data source path to point to your local CSV file
4. Click **Refresh** to reload data
5. Explore the dashboard using slicers for Gender, Season, Category, and Subscription Status

---

## 👤 Author

**Punith Raj**
Data Analyst | SQL · Python · Power BI
📧 punithraj0412@gmail.com
🔗 [GitHub](https://github.com/punith-da) · [LinkedIn](https://www.linkedin.com/in/punithraj)

---

## 📄 License

This project is for portfolio and learning purposes. Dataset is synthetic and does not represent real individuals.
