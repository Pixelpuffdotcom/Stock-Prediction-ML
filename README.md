# Stock-Prediction-ML
Stock Market Prediction System Machine learning pipeline predicting daily price movements using Yahoo Finance data. Features:  4 ML models (XGBoost, Random Forest, SVM, Logistic Regression)  Automated OHLCV data fetching &amp; preprocessing  Visualizations (price trends, volume analysis)  Model accuracy benchmarking &amp; export
"""
Stock Market Prediction System (Verified Working Version)
"""

import yfinance as yf
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.svm import SVC
from xgboost import XGBClassifier

# Configuration
SYMBOLS = ['AAPL', 'MSFT']
START_DATE = '2023-01-01'
END_DATE = '2023-12-31'

def create_dataset(symbol):
    """Generate clean dataset for a single symbol"""
    try:
        print(f"\nDownloading {symbol}...")
        data = yf.download(
            symbol,
            start=START_DATE,
            end=END_DATE,
            progress=False,
            auto_adjust=True
        )
        
        if data.empty:
            print(f"⚠️ No data for {symbol}")
            return None
            
        # Create features and target
        df = data[['Open', 'High', 'Low', 'Close', 'Volume']].copy()
        df['Target'] = (df['Close'].shift(-1) > df['Close']).astype(int)
        df = df[:-1]  # Remove last row with NaN target
        
        # Validate data
        if df.empty:
            print(f"⚠️ Empty data after processing {symbol}")
            return None
            
        print(f"✅ {symbol} data processed | Shape: {df.shape}")
        return df.dropna()
    
    except Exception as e:
        print(f"⚠️ Error processing {symbol}: {str(e)}")
        return None

def create_visualizations(df, symbol):
    """Generate 4 visualizations for a symbol"""
    # 1. Price Trend
    plt.figure(figsize=(12,6))
    df['Close'].plot(title=f'{symbol} Closing Price Trend')
    plt.savefig(f'{symbol}_price_trend.png')
    plt.close()
    
    # 2. Volume Distribution
    plt.figure(figsize=(12,6))
    df['Volume'].hist(bins=50)
    plt.title(f'{symbol} Trading Volume Distribution')
    plt.savefig(f'{symbol}_volume_dist.png')
    plt.close()
    
    # 3. Daily Returns
    plt.figure(figsize=(12,6))
    df['Close'].pct_change(fill_method=None).plot(kind='hist', bins=100, alpha=0.7)
    plt.title(f'{symbol} Daily Returns Distribution')
    plt.savefig(f'{symbol}_returns_dist.png')
    plt.close()
    
    # 4. Target Distribution
    plt.figure(figsize=(12,6))
    df['Target'].value_counts().plot(kind='pie', autopct='%1.1f%%')
    plt.title(f'{symbol} Price Movement Distribution')
    plt.savefig(f'{symbol}_target_dist.png')
    plt.close()

def train_models(X, y, symbol):
    """Train and compare 4 ML models for a symbol"""
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42, shuffle=False
    )
    
    models = {
        'Logistic Regression': LogisticRegression(max_iter=1000),
        'Random Forest': RandomForestClassifier(n_estimators=100),
        'Support Vector Machine': SVC(),
        'XGBoost': XGBClassifier()
    }
    
    results = {}
    for name, model in models.items():
        try:
            model.fit(X_train, y_train)
            preds = model.predict(X_test)
            results[name] = accuracy_score(y_test, preds)
            print(f"{symbol} - {name} Accuracy: {results[name]:.2%}")
        except Exception as e:
            print(f"⚠️ {symbol} - Error in {name}: {str(e)}")
            results[name] = None
    
    return results

def main():
    for symbol in SYMBOLS:
        print(f"\n🔄 Processing {symbol}...")
        df = create_dataset(symbol)
        if df is None:
            continue
            
        # Data validation
        print(f"\n🔍 {symbol} Data Validation:")
        print(f"Samples: {len(df)}")
        print(f"NaN values: {df.isna().sum().sum()}")
        print(f"Target distribution:\n{df['Target'].value_counts()}")
        
        # Create visualizations
        print(f"\n🔄 Creating visualizations for {symbol}...")
        create_visualizations(df, symbol)
        
        # Prepare data for ML
        X = df.drop('Target', axis=1)
        y = df['Target']
        
        # Train models
        print(f"\n🔄 Training models for {symbol}...")
        results = train_models(X, y, symbol)
        
        # Save results
        pd.Series(results).to_csv(f'{symbol}_model_results.csv')
    
    print("\n✅ Completed! Check created files.")

if __name__ == "__main__":
    main()
