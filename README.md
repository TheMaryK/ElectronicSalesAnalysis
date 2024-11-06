# Consumer Purchase Behaviour and Purchase Drivers Analysis in Electronic Sales

![electonic analysis word cloud](https://github.com/user-attachments/assets/1430557e-fe7f-4346-92d0-8fab1a9de33b)

## Table of Content
- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Tools](#tools)
- [Data Cleaning and Preprocesses](#data-cleaning-and-preprocesses)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Analysis](#data-analysis)
- [Findings](#findings)
- [Insights](#insights)
- [Recommendations](#recommendations)

## Project Overview

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

## Exploratory Data Analysis
Key business questions will be explored to uncover insights:
1. Which product types are the top performers in terms of revenue and units sold?
2. What are the average customer ratings for each product type?
3. To what extent do loyalty members provide higher average ratings compared to non-members?
4. What percentage of customers use each available payment method?
    Are there any significant differences in payment method usage across customer age groups?
5. What are the peak purchasing months, and do they align with customer satisfaction levels?
6. How does shipping type affect customer satisfaction (inferred from completed and cancelled orders)?
7. Which add-ons are most frequently purchased?

## Data Analysis
### Question 1

```python
#Step 1: Filter completed orders to focus on actual purchases
completed_orders = df[df['Order Status'] == 'Completed']

#Step 2: Calculate total revenue and units sold for each product type
revenue_by_product = completed_orders.groupby('Product Type')['Total Price'].sum()
units_sold_by_product = completed_orders.groupby('Product Type')['Quantity'].sum()

#Step 3: Combine both metrics into a single DataFrame for easy comparison
product_performance = pd.DataFrame({
    'Total Revenue': revenue_by_product,
    'Units Sold': units_sold_by_product
})

#Step 4: Sort by Total Revenue and Units Sold in descending order to identify top performers
product_performance_sorted = product_performance.sort_values(by=['Total Revenue', 'Units Sold'], ascending=False)

#Step 5: Display the top performers
print(product_performance_sorted)

```
### Question 2

```python
average_rating_by_product_type = df.groupby('Product Type').Rating.mean().round(2)
average_rating_by_product_type = average_rating_by_product_type.sort_values(ascending=False)
print(average_rating_by_product_type)
```
### Question 3

```python
loyalty_member_ratings = df[df['Loyalty Member'] == 'Yes']['Rating']
non_loyalty_member_ratings = df[df['Loyalty Member'] == 'No']['Rating']
avg_loyalty_member_rating = round(loyalty_member_ratings.mean(), 2)
avg_non_loyalty_member_rating = round(non_loyalty_member_ratings.mean(), 2)
print("Average Rating for Loyalty Members:", avg_loyalty_member_rating)
print("Average Rating for Non-Loyalty Members:", avg_non_loyalty_member_rating)
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
### Question 6

```python
# Group by 'Shipping Type' and 'Order Status' to get counts for each status per shipping type
order_status_by_shipping = df.groupby(['Shipping Type', 'Order Status']).size().unstack(fill_value=0)

# Calculate the total orders for each shipping type
order_status_by_shipping['Total Orders'] = order_status_by_shipping.sum(axis=1)

# Calculate the percentage of completed and cancelled orders for each shipping type
order_status_by_shipping['Completed %'] = (order_status_by_shipping.get('Completed', 0) / order_status_by_shipping['Total Orders']) * 100
order_status_by_shipping['Cancelled %'] = (order_status_by_shipping.get('Cancelled', 0) / order_status_by_shipping['Total Orders']) * 100

# Sort by Completed % in descending order and display the table
order_status_by_shipping = order_status_by_shipping[['Completed %', 'Cancelled %']].sort_values(by='Completed %', ascending=False)

# Round the results to two decimal places
order_status_by_shipping = order_status_by_shipping.round(2)


print("Order Completion and Cancellation Rates by Shipping Type:")
print(order_status_by_shipping)
```
### Question 7

```python
adds_on_purchased_counts = df['Add-ons Purchased'].value_counts()

# Display the most frequently purchased add-ons
print("Most Frequently Purchased Add-ons:")
print(adds_on_purchased_counts)
```

## Findings
1. Which product types are the top performers in terms of revenue and units sold?
   
|Rank|Product Type|Total Revenue($)|Units Sold| 
|----|------------|----------------|----------|
|1|Smartphone|14407835.84|21947|
|2|Smartwatch|9398591.23|14474|
|3|Laptop|8365905.25|14623|
|4|Tablet|7722632.25|15041
|5|Headphone|2734651.00|7565|             

2. What are the average customer ratings for each product type?
   
|Rank|Product Type|Average Rating|
|----|------------|--------------|
|1|Smartphone|3.32|
|2|Tablet|3.02|
|3|Headphone|2.99|
|4|Smartwatch|2.99|
|5|Laptop|2.98| 

3. To what extent do loyalty members provide higher average ratings compared to non-members?
   
|Rank|Loyalty Membership|Average Rating|
|----|------------------|--------------|
|1|Loyalty Members|3.1|
|2|Non-Loyalty Members|3.09| 

4. What percentage of customers use each available payment method?
   
|Rank|Payment Method|Percentage of customers|
|----|--------------|-----------------------|
|1|Credit Card|29.340%|
|2|Paypal|28.990%|
|3|Bank Transfer|16.855%|
|4|Cash|12.460%|
|5|Debit Card|12.355%|

Are there any significant differences in payment method usage across customer age groups?
From the analysis, credit cards and PayPal are the most widely used payment methods across all age groups while cash usage is consistently lower across all age groups. Overall, the data shows a clear shift toward digital payments, with subtle differences in preferences across age groups.

5. What are the peak purchasing months, and do they align with customer satisfaction levels?
January and May have the highest number of purchases, reaching the lowest counts in October, November, and December. 

6. How does shipping type affect customer satisfaction (inferred from completed and cancelled orders)?

|Rank|Shipping Type|Completed order%|                 
|----|-------------|----------------|
|1|Overnight|66.93|    
|2|Standard|67.82|   
|3|Same Day|66.67|   
|4|Expedited|67.54|    
|5|Express|66.16|   

|Rank|Shipping Type|Cancelled order%|
|----|-------------|----------------|
|1|Express|33.83|
|1|Expedited|32.45|    
|2|Same Day|33.32|   
|3|Standard|32.18|   
|4|Overnight|33.07|

7. Which add-ons are most frequently purchased?
The most frequently purchased  invidual Add-ons are Extended Warranty, Accessory and Impulse Item, while Extended Warranty + Impulse Item and Accessory + Impulse Item are the most common combinations                                      

## Insights
The following insights are gotten from the findings:
- Smartphones and smartwatches are the top revenue drivers, with strong sales and higher unit prices, while headphones have lower revenue per unit due to their lower price point. Laptops outpace tablets in total revenue despite having slightly lower unit prices.
  
- While most electronic products tend to hover around a 3-star average rating, smartphones stand out by consistently receiving higher ratings, suggesting they're a strong point in the market with greater customer satisfaction.

- Given the minimal difference in average ratings, the data suggests that loyalty membership does not significantly impact customer ratings.

-  Credit cards and PayPal is the most prefered payment method across age groups, while cash and Debit card has lower usage. This indicate that there is preference for digital payment methods, likely driven by convenience, ease, and perceived security.

-  Interestingly, the peak purchasing months (January–August) coincide with relatively stable but lower customer satisfaction scores while during the months with the lowest purchase volumes (October–December), customer satisfaction ratings are at their highest. This suggests that while fewer purchases occur later in the year, those who do purchase report higher satisfaction, possibly due to less demand on service or seasonal factors enhancing customer experience.

-  Shipping speed does not appear to significantly affect the overall cancellation rate, with a slight preference for standard shipping in terms of completed orders. The consistent cancellation rate across all shipping types suggests that cancellations may be influenced by factors such as customer preferences or product availability which is beyond just the shipping method.

-  Extended Warranty, Accessory, and Impulse Item are the most frequently purchased individual add-ons. This suggests that customers tend to purchase these add-ons separately rather than in combination.
Extended Warranty + Impulse Item and Accessory + Impulse Item are the most common combinations. This suggests that customers who buy impulse items are also likely to purchase either an extended warranty or an accessory.
Impulse items are often bought alongside extended warranties and accessories, suggesting that customers value both added protection and customization for their purchases.

## Recommendations
- To optimize revenue and enhance customer satisfaction, the following general recommendations are provided based on these findings.
1. The low purchase volume in the last quarter, despite higher satisfaction, could indicate an opportunity to boost marketing efforts during these months to convert the high satisfaction levels into increased sales, while customer satisfaction should be enhanced during the peak purchasing months.
2. Promotion of smartphones and smartwatches should be priortized to maximize revenue, bundle accessories like headphones with higher-priced items, and explore strategies to boost tablet and laptop sales through premium offerings or targeted marketing.
3. Since the cancellation rate is quite high across all shipping types, investigating reasons behind cancellations (e.g., unexpected delays, pricing, or customer experience issues) could help reduce this percentage.
