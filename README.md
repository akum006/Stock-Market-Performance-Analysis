!pip install yfinance

import pandas as pd
import yfinance as yf
from datetime import datetime

# Stock market performance analysis can be used to inform investment decisions and help investors make informed decisions about buying or selling stocks. For this task, Yahoo finance API (yfinance) is used to collect real-time stock market data for the past three months.

# Now below is how we can collect real-time stock market data using the yfinance API:

start_date = datetime.now()-pd.DateOffset(months=3)
end_date = datetime.now()
tickers = ['AAPL', 'MSFT', 'NFLX', 'GOOG']
df_list = []
for ticker in tickers:
    data = yf.download(ticker, start=start_date, end=end_date)
    df_list.append(data)

df = pd.concat(df_list, keys=tickers, names=['Ticker', 'Date'])
print(df.head())

df = df.reset_index()

df.drop('level_0',axis = 1,inplace = True)

df.drop('index',axis = 1,inplace = True)

# Now let’s have a look at the performance in the stock market of all the companies:

import plotly.express as px
fig = px.line(df, x='Date',
              y='Close',
              color='Ticker',
              title="Stock Market Performance for the Last 3 Months")
fig.show()

# Now let’s look at the faceted area chart, which makes it easy to compare the performance of different companies and identify similarities or differences in their stock price movements:

px.area(df,x='Date',y='Close',color='Ticker',facet_col='Ticker',labels={'Date':'Date', 'Close':'Closing Price', 'Ticker':'Company'},
              title='Stock Prices for Apple, Microsoft, Netflix, and Google')

# Now let’s analyze moving averages, which provide a useful way to identify trends and patterns in each company’s stock price movements over a period of time:

df['MA10'] = df.groupby('Ticker')['Close'].rolling(window=10).mean().reset_index(0, drop=True)
df['MA20'] = df.groupby('Ticker')['Close'].rolling(window=20).mean().reset_index(0, drop=True)

for ticker, group in df.groupby('Ticker'):
  print(f'Moving Averages for {ticker}')
  print(group[['MA10', 'MA20']])

for ticker, group in df.groupby('Ticker'):
  fig = px.line(group, x='Date',y=['Close', 'MA10', 'MA20'],title=f"{ticker} Moving Averages")
  fig.show()
  
# The output shows four separate graphs for each company. When the MA10 crosses above the MA20, it is considered a bullish signal indicating that the stock price will continue to rise. Conversely, when the MA10 crosses below the MA20, it is a bearish signal that the stock price will continue falling.

# Let us now analyze the volatility of all companies. Volatility is a measure of how much and how often the stock price or market fluctuates over a given period of time. Here’s how to visualize the volatility of all companies:

df['Volatility'] = df.groupby('Ticker')['Close'].pct_change().rolling(window=10).std().reset_index(0,drop=True)
fig = px.line(df,x='Date',y='Volatility',color='Ticker',title='Volatility of All Companies')
fig.show()

# High volatility indicates that the stock or market experiences large and frequent price movements, while low volatility indicates that the market experiences smaller or less frequent price movements.

# Now let’s analyze the correlation between the stock prices of Apple and Microsoft:

apple = df.loc[df['Ticker']== 'AAPL',['Date','Close']].rename(columns={'Close': 'AAPL'})
microsoft = df.loc[df['Ticker']== 'MSFT',['Date','Close']].rename(columns={'Close': 'MSFT'})

df_corr = pd.merge(apple, microsoft, on = 'Date')

fig = px.scatter(df_corr,x='AAPL',y='MSFT',trendline= 'ols', title='Correlation between Apple and Microsoft')
fig.show()

# There is a strong linear relationship between the stock prices of Apple and Microsoft, which means that when the stock price of Apple increases, the stock price of Microsoft also tends to increase. It is a sign of a strong correlation or similarity between the two companies, which can be due to factors such as industry trends, market conditions, or common business partners or customers. For investors, this positive correlation may indicate an opportunity to diversify their portfolio by investing in both companies, as both stocks may offer similar potential returns and risks.

# Stock Market Performance Analysis involves calculating moving averages, measuring volatility, conducting correlation analysis and analyzing various aspects of the stock market to gain a deeper understanding of the factors that affect stock prices and the relationships between the stock prices of different companies.
