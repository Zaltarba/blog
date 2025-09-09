---
layout: post
title: How to Fetch and Store Binance Data Efficiently Using HDF5 in Python
categories: [Quant, Algo Trading, Data Engineering] 
excerpt: Learn how to fetch Binance candlestick data and store it efficiently using HDF5 in Python. Discover the Binance API and how to keep your data process scalable.
image: /thumbnails/DataBaseCreation.jpeg
hidden: False
tags: [binance, trading, garch, volatility, quant, ewma, bitcoin, csv, hdf5, parquet]
---

## Why HDF5 is Ideal for Storing Cryptocurrency Data

When handling large datasets, especially in research, the choice of file format is crucial. [HDF5](https://fr.wikipedia.org/wiki/Hierarchical_Data_Format) (Hierarchical Data Format version 5) is a powerful data model that is ideal for managing and storing enormous amounts of data efficiently. It's designed to store data in a hierarchical structure, making it a perfect choice for large, complex datasets—like cryptocurrency trading data. Let's dive into how you can leverage HDF5 to store crypto data fetched from [Binance](https://www.binance.com/en).

Before jumping into the code, let's discuss why HDF5 is so beneficial:

- **Efficiency**: HDF5 is built for writing and reading speed. It can handle massive datasets much more efficiently than traditional file formats like CSV.
- **Scalability**: It’s designed to store and organize large amounts of data across multiple datasets.
- **Flexibility**: HDF5 files can be easily sliced and diced for different analyses without needing to load the entire dataset into memory.

These characteristics make HDF5 a valuable tool for research, especially when working with high-frequency trading data, which can accumulate quickly.   

Want some proof ? Here is an comparaison between .csv and .h5 files for a small dataset.  

For the **writing speed** :

```python
%time df.to_csv("data/speed_test.csv")
```
```bash
CPU times: total: 93.8 ms
Wall time: 123 ms
```
```python
%time df.to_hdf("data/speed_test.h5", key="data", mode="w")
```
```bash
CPU times: total: 0 ns
Wall time: 14.7 ms
```

For the **reading speed** : 

```python
%time a = pd.read_csv("data/speed_test.csv")
```
```bash
CPU times: total: 15.6 ms
Wall time: 32 ms
```
```python
%time b = pd.read_hdf("data/speed_test.h5", key="data")
```
```bash
CPU times: total: 15.6 ns
Wall time: 11.7 ms
```

As you can see, the csv format can't compete.

## Getting Started with Python Imports

First, let's set up our environment by importing the necessary libraries:

```python
import requests
import pandas as pd
from datetime import datetime, timedelta
import time
```
These libraries are fundamental for fetching, processing, and storing cryptocurrency data. requests will help us interact with Binance’s API, while pandas is crucial for data manipulation. datetime and time will manage our data fetching timeframes.

## Fetching and Storing Binance Data in HDF5

Next, we’ll create a function to fetch candlestick data from Binance and store it in an HDF5 file. The HDF5 file acts as a database, storing our data in a structured and efficient manner.

```python
def fetch_candlestick_data(symbol, interval, start_date, end_date, limit=10000, hdf5_file='data/crypto_database.h5', key='candlestick_data'):
    url = "https://api.binance.com/api/v3/klines"
    
    # Convert dates to timestamps in milliseconds
    start_ts = int(datetime.strptime(start_date, '%Y-%m-%d %H:%M:%S').timestamp() * 1000)
    end_ts = int(datetime.strptime(end_date, '%Y-%m-%d %H:%M:%S').timestamp() * 1000)

    # Calculate the total number of steps
    total_steps = (end_ts - start_ts) // (limit * 60 * 1000) + 1

    # Initialize step counter
    step_counter = 0
    
    while start_ts < end_ts:
        params = {
            'symbol': symbol,
            'interval': interval,
            'startTime': start_ts,
            'endTime': min(start_ts + limit * 60 * 1000, end_ts),
            'limit': limit
        }
        
        response = requests.get(url, params=params)
        if response.status_code == 200:
            data = response.json()
            if not data:
                break
            df = pd.DataFrame(data, columns=[
                'Open_Time', 
                'Open', 
                'High', 
                'Low', 
                'Close', 
                'Volume', 
                'Close_Time', 
                'Quote_Asset_Volume', 
                'Number_of_Trades', 
                'Taker_Buy_Base_Asset_Volume', 
                'Taker_Buy_Quote_Asset_Volume', 
                'Ignore'
            ])
            
            # Convert timestamps to datetime
            df['Open_Time'] = pd.to_datetime(df['Open_Time'], unit='ns')
            del df['Close_Time']
            # Convert numeric columns to appropriate types
            df['Open'] = pd.to_numeric(df['Open'])
            df['High'] = pd.to_numeric(df['High'])
            df['Low'] = pd.to_numeric(df['Low'])
            df['Close'] = pd.to_numeric(df['Close'])
            df['Volume'] = pd.to_numeric(df['Volume'])
            df['Quote_Asset_Volume'] = pd.to_numeric(df['Quote_Asset_Volume'])
            df['Taker_Buy_Base_Asset_Volume'] = pd.to_numeric(df['Taker_Buy_Base_Asset_Volume'])
            df['Taker_Buy_Quote_Asset_Volume'] = pd.to_numeric(df['Taker_Buy_Quote_Asset_Volume'])

            # Set 'Open Time' as index
            df.set_index('Open_Time', inplace=True)

            # Store DataFrame in HDF5 file
            df.to_hdf(
                hdf5_file, 
                key=key, 
                mode='a', 
                format='table', 
                append=True, 
                data_columns=True,
            )
            
            # Update step counter
            step_counter += 1
            
            # Print progress
            print(f"Step {step_counter}/{total_steps} completed.")
            
            start_ts = data[-1][0] + 1  # Move to the next timestamp
        else:
            print(f"Error: {response.status_code}")
            time.sleep(60)  # Wait a minute before retrying
            continue
        
        time.sleep(1)  # To avoid hitting rate limits
```

Breaking Down the Function : 

- API Request: The function sends a request to Binance's API for candlestick data. We loop through time intervals using the start_ts and end_ts timestamps to ensure we gather data over a specified period.

- Data Processing: The JSON response is converted into a pandas DataFrame, making it easier to work with. We format the data by converting timestamps and changing numeric columns to their appropriate types.

- HDF5 Storage: Finally, we store the DataFrame in an HDF5 file. This allows us to append new data over time, making it an efficient method for long-term storage.

## Running the Data Collection

Now, let's tie it all together with a main function that initiates the data fetching process:

```python
def main():
    symbol = 'BTCUSDT'
    interval = '1m'  # 1-minute granularity
    start_date = '2020-01-01 00:00:00'
    end_date = '2020-02-01 00:00:00'
    
    # Fetch the data and store it in HDF5
    fetch_candlestick_data(
        symbol, 
        interval, 
        start_date, end_date, 
        hdf5_file='data/crypto_database.h5',
        key=symbol)

if __name__ == "__main__":
    main()
```

## Final Thoughts

By using HDF5, you gain a powerful tool for managing large datasets, ensuring that your research is both scalable and efficient. Whether you're analyzing historical price data or developing trading strategies, this approach provides a robust foundation for your work.

Moreover, you can expand your database by storing additional cryptocurrency pairs in the same HDF5 file. Accessing them is straightforward—simply use pd.read_hdf() with the desired key, allowing you to seamlessly retrieve and analyze data across multiple assets.

## Additional Resources

- **Code Repository**: [GitHub Link](https://github.com/Zaltarba/BitcoinVolatilityEstimation/tree/main) 
- **Adviced Reading**: Yves Hilpish [*Python for Algorithmic Trading*](https://www.oreilly.com/library/view/python-for-algorithmic/9781492053347/)

I'd love to hear your thoughts! Share your feedback, questions, or suggestions for future topics in the comments below.

**Happy coding**, and may your data always be **manageable**!

## Frequently Asked Questions

**Q1: What is HDF5 and why is it useful for crypto data?**  
A: HDF5 is a high-performance file format that stores large datasets efficiently, ideal for high-frequency cryptocurrency trading data.  

**Q2: Can I store multiple cryptocurrency pairs in one HDF5 file?**  
A: Yes, HDF5 allows storing multiple datasets in a hierarchical structure, making it easy to organize data for multiple assets.  

**Q3: How does HDF5 improve Python data processing speed?**  
A: HDF5 provides fast reading and writing operations, especially for large datasets, outperforming traditional CSV files.

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What is HDF5 and why is it useful for crypto data?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "HDF5 is a high-performance file format that stores large datasets efficiently, ideal for high-frequency cryptocurrency trading data."
      }
    },
    {
      "@type": "Question",
      "name": "Can I store multiple cryptocurrency pairs in one HDF5 file?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Yes, HDF5 allows storing multiple datasets in a hierarchical structure, making it easy to organize data for multiple assets."
      }
    },
    {
      "@type": "Question",
      "name": "How does HDF5 improve Python data processing speed?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "HDF5 provides fast reading and writing operations, especially for large datasets, outperforming traditional CSV files."
      }
    }
  ]
}
</script>

