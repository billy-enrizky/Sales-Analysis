# Sales Data Analysis Project

This project involves the analysis of sales data to gain insights into various aspects of the sales operation. The dataset used for this analysis includes information about sales orders, such as order details, products, quantities, prices, and more.

## Table of Contents
1. [Project Overview](#project-overview)
2. [Getting Started](#getting-started)
3. [Data Cleaning](#data-cleaning)
4. [Data Exploration](#data-exploration)
    - [Question 1: What was the best month for sales? How much was earned that month?](#question-1)
    - [Question 2: What city sold the most products?](#question-2)
    - [Question 3: What time should we display advertisements to maximize the likelihood of customers buying products?](#question-3)
    - [Question 4: What products are most often sold together?](#question-4)
    - [What product sold the most? Why do you think it sold the most?](#most-sold-product)

<a name="project-overview"></a>
## Project Overview

The goal of this project is to analyze sales data to gain insights into various aspects of the sales operation. This includes cleaning the data, performing data exploration, and answering specific questions related to sales performance.

<a name="getting-started"></a>
## Getting Started

### Import necessary libraries
```python
import os
import pandas as pd
```

### Merge data from each month into one CSV
```python
path = "./Sales_Data"
files = [file for file in os.listdir(path) if not file.startswith('.')]  # Ignore hidden files
all_months_data = pd.DataFrame()

for file in files:
    current_data = pd.read_csv(path + "/" + file)
    all_months_data = pd.concat([all_months_data, current_data])
    
all_months_data.to_csv("all_data_copy.csv", index=False)
```

### Read in the updated dataframe
```python
all_data = pd.read_csv("all_data.csv")
```

<a name="data-cleaning"></a>
## Data Cleaning

### Drop rows of NaN
```python
nan_df = all_data[all_data.isna().any(axis=1)]
all_data = all_data.dropna(how='all')
```

### Get rid of text in the 'Order Date' column
```python
all_data = all_data[all_data['Order Date'].str[0:2] != 'Or']
```

### Make columns the correct type
```python
all_data['Quantity Ordered'] = pd.to_numeric(all_data['Quantity Ordered'])
all_data['Price Each'] = pd.to_numeric(all_data['Price Each'])
```

### Augment data with additional columns

#### Add month column
```python
all_data['Month'] = all_data['Order Date'].str[0:2]
all_data['Month'] = all_data['Month'].astype('int32')
```

#### Add month column (alternative method)
```python
all_data['Month 2'] = pd.to_datetime(all_data['Order Date']).dt.month
```

#### Add city column
```python
def get_city(address):
    return address.split(",")[1].strip(" ")

def get_state(address):
    return address.split(",")[2].split(" ")[1]

all_data['City'] = all_data['Purchase Address'].apply(lambda x: f"{get_city(x)}  ({get_state(x)})")
```

<a name="data-exploration"></a>
## Data Exploration

<a name="question-1"></a>
### Question 1: What was the best month for sales? How much was earned that month?
```python
all_data['Sales'] = all_data['Quantity Ordered'].astype('int') * all_data['Price Each'].astype('float')
sales_by_month = all_data.groupby(['Month']).sum()

import matplotlib.pyplot as plt

months = range(1, 13)

plt.bar(months, sales_by_month['Sales'])
plt.xticks(months)
plt.ylabel('Sales in USD ($)')
plt.xlabel('Month number')
plt.show()
```

<a name="question-2"></a>
### Question 2: What city sold the most product?
```python
city_sales = all_data.groupby(['City']).sum()

keys = [city for city, df in all_data.groupby(['City'])]

plt.bar(keys, city_sales['Sales'])
plt.ylabel('Sales in USD ($)')
plt.xlabel('City')
plt.xticks(keys, rotation='vertical', size=8)
plt.show()
```

<a name="question-3"></a>
### Question 3: What time should we display advertisements to maximize the likelihood of customers buying a product?
```python
all_data['Hour'] = pd.to_datetime(all_data['Order Date']).dt.hour

keys = [pair for pair, df in all_data.groupby(['Hour'])]

plt.plot(keys, all_data.groupby(['Hour']).count()['Count'])
plt.xticks(keys)
plt.grid()
plt.show()
```

<a name="question-4"></a>
### Question 4: What products are most often sold together?
```python
# Find products that are often sold together
df = all_data[all_data['Order ID'].duplicated(keep=False)]
df['Grouped'] = df.groupby('Order ID')['Product'].transform(lambda x: ','.join(x))
df2 = df[['Order ID', 'Grouped']].drop duplicates()

# Count combinations
from itertools import combinations
from collections import Counter

count = Counter()
for row in df2['Grouped']:
    row_list = row.split(',')
    count.update(Counter(combinations(row_list, 2))

# Display the most common product combinations
for key, value in count.most_common(10):
    print(key, value)
```

<a name="most-sold-product"></a>
### What product sold the most? Why do you think it sold the most?
```python
product_group = all_data.groupby('Product')
quantity_ordered = product_group.sum()['Quantity Ordered']

keys = [pair for pair, df in product_group]

plt.bar(keys, quantity_ordered)
plt.xticks(keys, rotation='vertical', size=8)

prices = all_data.groupby('Product').mean()['Price Each']

fig, ax1 = plt.subplots()
ax2 = ax1.twinx()
ax1.bar(keys, quantity_ordered, color='g')
ax2.plot(keys, prices, color='b')
ax1.set_xlabel('Product Name')
ax1.set_ylabel('Quantity Ordered', color='g')
ax2.set_ylabel('Price ($)', color='b')
plt.show()
```

This README provides an overview of the Sales Data Analysis project, including the code for data cleaning, data exploration, and answers to specific questions related to the sales data. The project aims to provide insights into sales performance, best-selling products, and sales trends over time.
