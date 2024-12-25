import streamlit as st
import yfinance as yf
import pandas as pd
import numpy as np
import requests
from textblob import TextBlob
import seaborn as sns
import matplotlib.pyplot as plt

# Set the Streamlit page layout
st.set_page_config(page_title="Dynamic Market Sentiment Analyzer", layout="wide")

# Header
st.title("Dynamic Market Sentiment Analyzer")

# Sidebar Configuration
st.sidebar.header("Configuration Panel")
st.sidebar.write("Customize your analysis settings below:")

# Input Tickers
tickers = st.sidebar.text_input("Enter stock tickers separated by commas (e.g., AAPL, TSLA, MSFT):", "AAPL, TSLA, MSFT")
days = st.sidebar.slider("Select the number of past days for analysis:", min_value=30, max_value=365, value=180)

# Split tickers and fetch data
def fetch_stock_data(tickers, days):
    tickers_list = tickers.split(",")
    data = {}
    for ticker in tickers_list:
        stock_data = yf.download(ticker.strip(), period=f"{days}d", progress=False)
        data[ticker.strip()] = stock_data['Close']
    return pd.DataFrame(data)

# Fetch stock data
data = fetch_stock_data(tickers, days)

# Stock Correlation Heatmap
st.header("Stock Correlation Heatmap")
if not data.empty:
    correlation = data.corr()
    fig, ax = plt.subplots(figsize=(10, 6))
    sns.heatmap(correlation, annot=True, cmap="coolwarm", ax=ax)
    st.pyplot(fig)
else:
    st.error("No data available for the selected tickers.")

# Real-Time Sentiment Analysis
def fetch_news_sentiment():
    url = "https://newsapi.org/v2/everything?q=stocks&language=en&sortBy=publishedAt&apiKey=YOUR_NEWS_API_KEY"
    response = requests.get(url)
    if response.status_code == 200:
        articles = response.json().get("articles", [])
        sentiments = []
        for article in articles[:10]:  # Limit to the latest 10 articles
            headline = article.get("title", "")
            analysis = TextBlob(headline)
            sentiments.append((headline, analysis.sentiment.polarity))
        return sentiments
    else:
        return []

st.header("Real-Time Market Sentiment")
try:
    news_sentiments = fetch_news_sentiment()
    if news_sentiments:
        for headline, sentiment in news_sentiments:
            st.write(f"Headline: {headline}")
            st.write(f"Sentiment Score: {sentiment}")
            st.write("---")
    else:
        st.warning("No news data available. Check your API key or the news source.")
except Exception as e:
    st.error(f"Error fetching news sentiment: {e}")

# Portfolio Risk Analysis
def calculate_var(portfolio, confidence_level=0.95):
    returns = portfolio.pct_change().dropna()
    var = np.percentile(returns, (1 - confidence_level) * 100)
    return var

st.header("Portfolio Risk Analysis")
portfolio = st.text_area("Enter your portfolio as stock tickers separated by commas (e.g., AAPL, TSLA):")
if portfolio:
    portfolio_data = fetch_stock_data(portfolio, days)
    if not portfolio_data.empty:
        risk_score = calculate_var(portfolio_data, 0.95)
        st.write(f"Portfolio Value at Risk (VaR) at 95% confidence: {risk_score * 100:.2f}%")
    else:
        st.error("No data available for the portfolio tickers.")
else:
    st.info("Please enter your portfolio tickers above.")

# Closing Remarks
st.sidebar.success("Configuration complete. Explore the insights!")
