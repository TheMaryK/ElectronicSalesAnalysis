# Customer Insights and Payment Trends Analysis in Electronic Sales

##Table of Content
- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Tools](#tools)
- [Data Cleaning and Preprocesses](#data-cleaning-and-preprocesses)
- [Exploratory Data Analysis (EDA)](#exploratory-data-analysis-(EDA))
- 

## Project Overview

This project focuses on uncovering key trends in customer purchasing behavior, payment preferences, and satisfaction levels for an electronic company from September 2023 to September 2024. This analysis aims to provide data-driven insights to help enhance customer satisfaction, optimize marketing strategies, and inform business growth initiatives.
This project leverages electronic sales data from September 2023 to September 2024 to reveal **customer satisfaction trends**, **payment preferences**, and **product performance insights**, helping to **optimize customer experience** and **drive strategic business growth** of an electronic company.

## Data Source

The data for this project, **"Electronic_sales.csv"**, was sourced from **Kaggle** [see it here](https://www.kaggle.com/datasets/cameronseamons/electronic-sales-sep2023-sep2024). 
This dataset contains comprehensive customer attributes, product, order and transaction details.

## Tools
- Python: For data analysis and visualization.
      Key libraries used include:
    - Pandas - For data cleaning, manipulation and analysis.
    - Numpy -  For numerical operations and calculations.
    - Matplotlib - For creating static visualizations.
    - Seaborn - For enhanced visualizations and statistical graphics.
- Google Slides -  For creating presentations to share findings.
- Google Colab - For executing Python code and sharing notebooks collaboratively.
- GitHub - For hosting the project documentation and version control

## Data Cleaning and Preprocesses 
In this phase, the following task was performed: [check it out](https://colab.research.google.com/drive/1Rk3fFlJWbC0tKCgyFpSJwqOTqGoGVxXs?usp=sharing)
1. Data Loading and Inspection
   ```python
   df = pd.read_csv('Electronic_sales.csv')
   df
   ```
2. Handling of  Missing Values
   ```python
   #Step 1: Identify missing values.
   df.isnull().sum()

   # Step 2: Fill missing values
   df['Add-ons Purchased'].fillna('None', inplace=True)
   df['Gender'] = df['Gender'].fillna(df['Gender'].mode()[0])
   ```
3. Regularizing of Inconsistent Naming Conventions in the 'Payment Method' Column
   ```python
   df['Payment Method'] = df['Payment Method'].str.title()
   ```
   
4. Create 'Age Group' column based on different life stages for better analysis
   ```python
   age_bins = [18, 25, 35, 45, 55, 65, 81]
   age_labels = ['18-24', '25-34', '35-44', '45-54', '55-64', '65+']
   df['Age Group'] = pd.cut(df['Age'], bins=age_bins, labels=age_labels, right=False)
   ```
   
5. Converting date entries to a standard datetime format for uniform analysis
   ```python
   df['Purchase Date'] = pd.to_datetime(df['Purchase Date'])
   ```

## Exploratory Data Analysis (EDA)
Key business questions will be explored to uncover insights:
1. Which product types generate the highest average annual revenue, and which have the lowest?
2. What are the average customer ratings for each product type?, and how does this impact customer satisfaction?
3. To what extent do loyalty members provide higher average ratings compared to non-members, and are these differences statistically significant?
4. What percentage of customers use each available payment method? Are there any significant differences in payment method usage across customer age groups?
5. What are the peak purchasing months, and do they align with customer satisfaction levels?

## Data Analysis
### Question 1

```python
completed_orders = df[df['Order Status'] == 'Completed']
average_price_by_product = completed_orders.groupby('Product Type')['Total Price'].mean()
print(average_price_by_product)
```
### Question 2

```python
average_rating_by_product_type = df.groupby('Product Type').Rating.mean()
print(average_rating_by_product_type)
```
### Question 3

```python
loyalty_member_ratings = df[df['Loyalty Member'] == 'Yes']['Rating']
non_loyalty_member_ratings = df[df['Loyalty Member'] == 'No']['Rating']
avg_loyalty_member_rating = loyalty_member_ratings.mean()
avg_non_loyalty_member_rating = non_loyalty_member_ratings.mean()
print("Average Rating for Loyalty Members:", avg_loyalty_member_rating)
print("Average Rating for Non-Loyalty Members:", avg_non_loyalty_member_rating)

t_stat, p_value = ttest_ind(loyalty_member_ratings, non_loyalty_member_ratings, equal_var=False)
print("T-Statistic:", t_stat)
print("P-Value:", p_value)
if p_value < 0.05:
    print("There is a statistically significant difference in average ratings between loyalty members and non-members.")
else:
    print("There is no statistically significant difference in average ratings between loyalty members and non-members.")
```
### Question 4

```python
payment_method_counts = df['Payment Method'].value_counts()
payment_method_percentages = (payment_method_counts / len(df)) * 100

print("Percentage of Customers by Payment Method:")
print(payment_method_percentages)
age_bins = [18, 25, 35, 45, 55, 65, 81]
age_labels = ['18-24', '25-34', '35-44', '45-54', '55-64', '65+']
df['Age Group'] = pd.cut(df['Age'], bins=age_bins, labels=age_labels, right=False)
payment_method_by_age_group = df.groupby(['Age Group', 'Payment Method']).size().unstack().fillna(0)
payment_method_by_age_group_percent = payment_method_by_age_group.div(payment_method_by_age_group.sum(axis=1), axis=0) * 100

print("\nPayment Method Usage Across Age Groups (Percentage):")
print(payment_method_by_age_group_percent)
```
### Question 5

```python
completed_orders = df[df['Order Status'] == 'Completed']

monthly_purchase_volume = completed_orders.groupby('Month').size()
monthly_avg_satisfaction = completed_orders.groupby('Month')['Rating'].mean()

print("Monthly Purchase Volume (Completed Orders):")
print(monthly_purchase_volume)

print("\nMonthly Average Customer Satisfaction:")
print(monthly_avg_satisfaction)
```
## Findings
1.
|Product Type|Average Annual Revenue ($)|
---
|Headphones|2009.295371
---
|Laptop|3114.633377|
---
|Smartphone|3598.360599|
---
|Smartwatch|3565.474670|
---
|Tablet|2813.345082|
---
