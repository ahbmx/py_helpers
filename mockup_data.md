# Enhanced Sample Data Generator with Custom Date Ranges

Here's an improved Python function that creates various types of sample DataFrames with customizable date ranges and frequencies:

```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
import random

def create_sample_data(data_type='sales', start_date='2023-01-01', end_date='2023-12-31', freq='D', size=None):
    """
    Create sample DataFrames with mock data for testing and visualization.
    
    Parameters:
    data_type (str): Type of data to create. Options:
        - 'sales': Sales data with dates, products, and metrics
        - 'financial': Financial time series data
        - 'web_traffic': Website traffic and user engagement metrics
        - 'ecommerce': E-commerce transactions with user demographics
        - 'timeseries': Multiple time series with trend, seasonality, and noise
        - 'correlation': Data with correlated variables for relationship analysis
        - 'marketing': Marketing campaign performance metrics
    
    start_date (str): Start date for time series data (YYYY-MM-DD format)
    end_date (str): End date for time series data (YYYY-MM-DD format)
    freq (str): Frequency of time series ('D' for daily, 'H' for hourly, 'W' for weekly, etc.)
    size (int): Number of records for non-time series data
    
    Returns:
    pandas.DataFrame: Requested DataFrame
    """
    
    np.random.seed(42)  # For reproducible results
    
    # Generate date range based on parameters
    dates = pd.date_range(start=start_date, end=end_date, freq=freq)
    n_records = len(dates)
    
    if data_type == 'sales':
        products = ['Laptop', 'Smartphone', 'Tablet', 'Headphones', 'Monitor', 
                   'Keyboard', 'Mouse', 'Printer', 'Camera', 'Smartwatch']
        regions = ['North America', 'Europe', 'Asia', 'South America', 'Africa', 'Australia']
        
        data = []
        for i, date in enumerate(dates):
            for product in products[:random.randint(3, 8)]:  # Vary products per day
                region = random.choice(regions)
                units_sold = random.randint(1, 50)
                unit_price = random.uniform(50, 2000)
                discount = random.uniform(0, 0.3) if random.random() > 0.7 else 0
                
                data.append({
                    'date': date,
                    'product': product,
                    'region': region,
                    'units_sold': units_sold,
                    'unit_price': round(unit_price, 2),
                    'discount': round(discount, 3),
                    'revenue': round(units_sold * unit_price * (1 - discount), 2)
                })
        
        df = pd.DataFrame(data)
    
    elif data_type == 'financial':
        # Start with a base price
        base_price = 100
        
        # Generate random walk for stock price
        returns = np.random.normal(0.001, 0.02, n_records)
        price = base_price * np.cumprod(1 + returns)
        
        df = pd.DataFrame({
            'date': dates,
            'price': np.round(price, 2),
            'volume': np.random.lognormal(10, 1.2, n_records).astype(int),
            'rsi': np.random.uniform(30, 70, n_records),
            'macd': np.random.normal(0, 1.5, n_records),
            'volatility': np.random.exponential(0.5, n_records)
        })
        
        # Add some autocorrelation to make it more realistic
        df['price'] = df['price'].rolling(window=5, min_periods=1).mean()
    
    elif data_type == 'web_traffic':
        # Weekly seasonality (higher traffic on weekdays)
        day_of_week = dates.dayofweek
        seasonal_factor = np.where(day_of_week < 5, 1.5, 0.7)  # Higher on weekdays
        
        # Random growth trend
        trend = np.linspace(1, 1.5, n_records)
        
        # Generate traffic data
        visits = np.random.poisson(1000 * seasonal_factor * trend)
        
        df = pd.DataFrame({
            'date': dates,
            'visits': visits,
            'unique_visitors': (visits * np.random.uniform(0.6, 0.9, n_records)).astype(int),
            'pageviews': (visits * np.random.uniform(1.5, 3.5, n_records)).astype(int),
            'bounce_rate': np.random.uniform(0.3, 0.7, n_records),
            'avg_session_duration': np.random.uniform(60, 300, n_records)
        })
    
    elif data_type == 'ecommerce':
        if size is None:
            size = 1000
            
        first_names = ['Emma', 'Liam', 'Olivia', 'Noah', 'Ava', 'William', 'Sophia', 'James', 
                      'Isabella', 'Oliver', 'Charlotte', 'Elijah', 'Amelia', 'Benjamin', 'Mia']
        last_names = ['Smith', 'Johnson', 'Williams', 'Brown', 'Jones', 'Garcia', 'Miller', 'Davis', 
                     'Rodriguez', 'Martinez', 'Hernandez', 'Lopez', 'Gonzalez', 'Wilson', 'Anderson']
        products = ['Laptop', 'Smartphone', 'Tablet', 'Headphones', 'Monitor', 
                   'Keyboard', 'Mouse', 'Printer', 'Camera', 'Smartwatch']
        categories = ['Electronics', 'Computers', 'Accessories', 'Home Appliances', 'Gadgets']
        
        data = []
        for i in range(size):
            first_name = random.choice(first_names)
            last_name = random.choice(last_names)
            product = random.choice(products)
            category = random.choice(categories)
            quantity = random.randint(1, 3)
            unit_price = random.uniform(50, 2000)
            age = random.randint(18, 80)
            days_ago = random.randint(0, 90)
            
            data.append({
                'customer_id': f"C{10000 + i}",
                'first_name': first_name,
                'last_name': last_name,
                'email': f"{first_name.lower()}.{last_name.lower()}@example.com",
                'age': age,
                'product': product,
                'category': category,
                'quantity': quantity,
                'unit_price': round(unit_price, 2),
                'total_amount': round(quantity * unit_price, 2),
                'purchase_date': (datetime.now() - timedelta(days=days_ago)).strftime('%Y-%m-%d'),
                'country': random.choice(['USA', 'UK', 'Canada', 'Australia', 'Germany', 'France'])
            })
        
        df = pd.DataFrame(data)
    
    elif data_type == 'timeseries':
        # Create multiple time series with different patterns
        t = np.arange(n_records)
        
        # Linear trend
        trend = 0.05 * t
        
        # Seasonality (daily and weekly)
        daily_seasonality = 10 * np.sin(2 * np.pi * t / (24 if freq == 'H' else 1))
        weekly_seasonality = 15 * np.sin(2 * np.pi * t / (7 * (24 if freq == 'H' else 1)))
        
        # Noise
        noise = np.random.normal(0, 5, n_records)
        
        df = pd.DataFrame({
            'date': dates,
            'series_a': trend + daily_seasonality + noise,
            'series_b': 0.03 * t + 8 * np.sin(2 * np.pi * t / (30 * (24 if freq == 'H' else 1))) + np.random.normal(0, 3, n_records),
            'series_c': np.random.normal(50, 10, n_records),
            'series_d': np.log(t + 1) * 20 + 5 * np.sin(2 * np.pi * t / (14 * (24 if freq == 'H' else 1))) + np.random.normal(0, 2, n_records)
        })
    
    elif data_type == 'correlation':
        if size is None:
            size = 500
            
        # Create correlated variables
        base = np.random.normal(0, 1, size)
        
        df = pd.DataFrame({
            'variable_a': base + np.random.normal(0, 0.1, size),
            'variable_b': 2 * base + np.random.normal(0, 0.2, size),
            'variable_c': -base + np.random.normal(0, 0.3, size),
            'variable_d': np.random.normal(0, 1, size),  # Uncorrelated
            'variable_e': base**2 + np.random.normal(0, 0.2, size),  # Nonlinear relationship
            'category': np.random.choice(['Group 1', 'Group 2', 'Group 3'], size),
            'value': np.random.uniform(0, 100, size)
        })
    
    elif data_type == 'marketing':
        channels = ['Email', 'Social Media', 'Search Engine', 'Direct', 'Referral', 'Display Ads']
        campaigns = ['Summer Sale', 'Black Friday', 'Holiday Special', 'New Collection', 'Clearance']
        
        data = []
        for date in dates:
            for channel in channels:
                for campaign in random.sample(campaigns, k=random.randint(1, 3)):
                    impressions = random.randint(1000, 100000)
                    clicks = random.randint(50, 5000)
                    conversions = random.randint(5, 500)
                    spend = random.uniform(100, 10000)
                    
                    data.append({
                        'date': date,
                        'channel': channel,
                        'campaign': campaign,
                        'impressions': impressions,
                        'clicks': clicks,
                        'conversions': conversions,
                        'spend': round(spend, 2),
                        'cpc': round(spend / clicks, 2) if clicks > 0 else 0,
                        'conversion_rate': conversions / clicks if clicks > 0 else 0,
                        'roi': round(random.uniform(1, 10), 2)  # Random ROI for simplicity
                    })
        
        df = pd.DataFrame(data)
    
    else:
        raise ValueError(f"Unknown data_type: {data_type}. Choose from: 'sales', 'financial', 'web_traffic', 'ecommerce', 'timeseries', 'correlation', 'marketing'")
    
    return df

# Example usage and demonstration
if __name__ == "__main__":
    # Create sample data with different parameters
    sales_data = create_sample_data('sales', '2023-01-01', '2023-03-31', 'D')
    financial_data = create_sample_data('financial', '2022-01-01', '2022-12-31', 'D')
    web_data = create_sample_data('web_traffic', '2023-06-01', '2023-06-30', 'H')
    ecommerce_data = create_sample_data('ecommerce', size=500)
    
    print("Sales Data:")
    print(sales_data.head())
    print(f"\nShape: {sales_data.shape}")
    
    print("\nFinancial Data:")
    print(financial_data.head())
    print(f"\nShape: {financial_data.shape}")
    
    print("\nWeb Traffic Data (Hourly):")
    print(web_data.head())
    print(f"\nShape: {web_data.shape}")
    
    print("\nE-commerce Data:")
    print(ecommerce_data.head())
    print(f"\nShape: {ecommerce_data.shape}")
    
    # Show available data types
    print("\nAvailable data types:")
    print("['sales', 'financial', 'web_traffic', 'ecommerce', 'timeseries', 'correlation', 'marketing']")
```

## Key Features:

1. **Custom Date Ranges**: Specify start date, end date, and frequency (D, H, W, etc.)
2. **Multiple Data Types**: 
   - Sales data with products and regions
   - Financial time series with realistic patterns
   - Website traffic metrics
   - E-commerce transactions with customer demographics
   - Multiple time series with trend and seasonality
   - Data with correlation for relationship analysis
   - Marketing campaign performance metrics

3. **Realistic Patterns**: Includes trends, seasonality, and realistic distributions
4. **Flexible Size**: Control the number of records for non-time series data

## Usage Examples:

```python
# Create daily sales data for Q1 2023
sales_data = create_sample_data('sales', '2023-01-01', '2023-03-31', 'D')

# Create hourly web traffic data for a week
web_data = create_sample_data('web_traffic', '2023-06-01', '2023-06-07', 'H')

# Create financial data for a year
financial_data = create_sample_data('financial', '2022-01-01', '2022-12-31', 'D')

# Create e-commerce data with 1000 records
ecommerce_data = create_sample_data('ecommerce', size=1000)
```

This function will help you test various plotting functions with realistic and diverse data patterns.
