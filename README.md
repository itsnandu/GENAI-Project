#  Food Delivery Dataset

##  Overview

This guide walks through combining three different data sources into a unified dataset for analysis.

---

##  Input Files

| File | Format | Purpose | Key Column |
|------|--------|---------|------------|
| `orders.csv` | CSV | Transactional order data | `order_id` |
| `users.json` | JSON | User master data | `user_id` |
| `restaurants.sql` | SQL | Restaurant master data | `restaurant_id` |

---

##  Step-by-Step Process

### **Step 1: Load CSV Data** 

```python
import pandas as pd

# Load orders
orders_df = pd.read_csv('orders.csv')

# Convert dates
orders_df['order_date'] = pd.to_datetime(orders_df['order_date'], format='%d-%m-%Y')
```

**What happens:** Reads the CSV file into a pandas DataFrame and converts order dates to proper datetime format for time-based analysis.

**Columns in orders.csv:**
- `order_id` - Unique order identifier
- `user_id` - Links to users
- `restaurant_id` - Links to restaurants
- `order_date` - When the order was placed
- `total_amount` - Order value
- `restaurant_name` - Name from order record

---

### **Step 2: Load JSON Data** 

```python
import json

# Load users
with open('users.json', 'r') as f:
    users_data = json.load(f)

# Convert to DataFrame
users_df = pd.DataFrame(users_data)
```

**What happens:** Reads the JSON file and converts the list of user dictionaries into a pandas DataFrame.

**Columns in users.json:**
- `user_id` - Unique user identifier (JOIN KEY)
- `name` - User name
- `city` - User's city
- `membership` - Gold or Regular

---

### **Step 3: Load SQL Data** 

```python
import re

# Read SQL file
with open('restaurants.sql', 'r') as f:
    sql_content = f.read()

# Extract INSERT statements using regex
pattern = r"INSERT INTO restaurants VALUES \((\d+), '([^']+)', '([^']+)', ([\d.]+)\);"
matches = re.findall(pattern, sql_content)

# Create DataFrame
restaurants_df = pd.DataFrame(matches, 
    columns=['restaurant_id', 'restaurant_name', 'cuisine', 'rating'])

# Convert types
restaurants_df['restaurant_id'] = restaurants_df['restaurant_id'].astype(int)
restaurants_df['rating'] = restaurants_df['rating'].astype(float)
```

**What happens:** Uses regex to extract data from SQL INSERT statements and creates a DataFrame.

**Columns in restaurants.sql:**
- `restaurant_id` - Unique restaurant identifier (JOIN KEY)
- `restaurant_name` - Restaurant name (master)
- `cuisine` - Type of cuisine
- `rating` - Restaurant rating

---

### **Step 4: Merge the Datasets** 

#### **First Merge: Orders + Users**

```python
merged_df = orders_df.merge(
    users_df,
    on='user_id',
    how='left',
    suffixes=('_order', '_user')
)
```

**Join Type:** LEFT JOIN  
**Join Key:** `user_id`  
**Result:** All orders retained, with user info added where available

#### **Second Merge: (Orders + Users) + Restaurants**

```python
final_df = merged_df.merge(
    restaurants_df,
    on='restaurant_id',
    how='left',
    suffixes=('', '_restaurant')
)
```

**Join Type:** LEFT JOIN  
**Join Key:** `restaurant_id`  
**Result:** Complete dataset with order, user, and restaurant information

---

### **Step 5: Finalize Dataset** 

```python
# Add derived columns for analysis
final_df['order_month'] = final_df['order_date'].dt.month
final_df['order_year'] = final_df['order_date'].dt.year
final_df['order_quarter'] = final_df['order_date'].dt.quarter

# Save to CSV
final_df.to_csv('final_food_delivery_dataset.csv', index=False)
```

**What happens:** Adds time-based columns for easier analysis and saves the result.

---

##  Final Dataset Structure

The final dataset contains **15+ columns:**

| Column | Source | Description |
|--------|--------|-------------|
| `order_id` | Orders | Unique order ID |
| `order_date` | Orders | Order timestamp |
| `user_id` | Orders → Users | User identifier |
| `name` | Users | User name |
| `city` | Users | User's city |
| `membership` | Users | Gold/Regular |
| `restaurant_id` | Orders → Restaurants | Restaurant ID |
| `restaurant_name_from_order` | Orders | Restaurant name (transactional) |
| `restaurant_name_master` | Restaurants | Restaurant name (master) |
| `cuisine` | Restaurants | Cuisine type |
| `rating` | Restaurants | Restaurant rating |
| `total_amount` | Orders | Order value |
| `order_month` | Derived | Month (1-12) |
| `order_year` | Derived | Year |
| `order_quarter` | Derived | Quarter (Q1-Q4) |

---

##  Key Analysis Questions Students Should Answer

### 1. **Order Trends Over Time**
- Which months had the highest order volume?
- Is there seasonality in orders?
- How did orders change quarter-over-quarter?

```python
monthly_trend = final_df.groupby(['order_year', 'order_month'])['order_id'].count()
```

### 2. **User Behavior Patterns**
- Do Gold members order more frequently?
- What's the average order value by membership type?
- Which cities have the most active users?

```python
membership_analysis = final_df.groupby('membership').agg({
    'total_amount': ['mean', 'sum', 'count']
})
```

### 3. **City-wise Performance**
- Which cities generate the most revenue?
- What's the average order value per city?
- Which cuisines are popular in each city?

```python
city_performance = final_df.groupby('city').agg({
    'total_amount': 'sum',
    'order_id': 'count'
}).sort_values('total_amount', ascending=False)
```

### 4. **Cuisine Analysis**
- Which cuisines are most popular?
- Which cuisines generate the highest revenue?
- Is there a relationship between cuisine type and order value?

```python
cuisine_stats = final_df.groupby('cuisine').agg({
    'total_amount': ['sum', 'mean'],
    'order_id': 'count',
    'rating': 'mean'
})
```

### 5. **Revenue Distribution**
- What's the revenue split by membership type?
- Which restaurants generate the most revenue?
- How does rating correlate with order volume?

```python
import matplotlib.pyplot as plt

# Revenue by membership
final_df.groupby('membership')['total_amount'].sum().plot(kind='bar')
```

---

##  Best Practices Demonstrated

###  **Data Loading**
- Handle different file formats appropriately
- Convert data types for proper analysis
- Parse dates correctly

###  **Data Integration**
- Use LEFT JOIN to preserve all orders
- Add suffixes to handle duplicate column names
- Verify row counts after each merge

###  **Data Quality**
- Check for missing values
- Validate join keys
- Add derived columns for easier analysis

###  **Code Organization**
- Clear step-by-step structure
- Informative print statements
- Comments explaining each section

---

##  Running the Code

1. Place all three files in the same directory
2. Run the Python script:
   ```bash
   python merge_food_delivery_data.py
   ```
3. Output will be saved as `final_food_delivery_dataset.csv`

---

##  Next Steps for Analysis

After running the script, students can:

1. **Load the final dataset** for analysis
2. **Create visualizations** (bar charts, line graphs, heatmaps)
3. **Build dashboards** using tools like Tableau or Power BI
4. **Perform statistical analysis** (correlations, trends)
5. **Generate insights** for business decisions

---

##  Learning Outcomes

By completing this exercise, students will understand:

-  Working with multiple data formats (CSV, JSON, SQL)
-  Performing data integration using pandas
-  Using LEFT JOIN to preserve data integrity
-  Adding derived columns for analysis
-  Conducting exploratory data analysis
-  Writing clean, documented code

---

