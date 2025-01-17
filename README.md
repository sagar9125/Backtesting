# Backtesting
import pandas as pd

# Load data
data = pd.read_csv(r'C:\Users\Sagar Maurya\Downloads\SBIN.csv')

# Calculate additional indicators
# MACD
data['EMA12'] = data['Close'].ewm(span=12, adjust=False).mean()
data['EMA26'] = data['Close'].ewm(span=26, adjust=False).mean()
data['MACD'] = data['EMA12'] - data['EMA26']
data['Signal_Line'] = data['MACD'].ewm(span=9, adjust=False).mean()

# RSI
delta = data['Close'].diff(1)
gain = delta.where(delta > 0, 0)
loss = -delta.where(delta < 0, 0)
avg_gain = gain.rolling(window=14).mean()
avg_loss = loss.rolling(window=14).mean()
rs = avg_gain / avg_loss
data['RSI'] = 100 - (100 / (1 + rs))

# Heiken Ashi
data['HA_Close'] = (data['Open'] + data['High'] + data['Low'] + data['Close']) / 4
data['HA_Open'] = (data['Open'].shift(1) + data['Close'].shift(1)) / 2
data['HA_High'] = data[['High', 'HA_Open', 'HA_Close']].max(axis=1)
data['HA_Low'] = data[['Low', 'HA_Open', 'HA_Close']].min(axis=1)

# Initialize variables
buy, sell = 0, 0
buy_price, sell_price = 0, 0
target, stop_loss = 0.01, 0.01
profit, loss, profit_trade, sell_trade = 0, 0, 0, 0
qnt = 1000

# Trading logic
for i in range(1, len(data)):
    close = data['Close'][i]
    open_ = data['Open'][i]
    macd = data['MACD'][i]
    signal_line = data['Signal_Line'][i]
    rsi = data['RSI'][i]
    ha_open = data['HA_Open'][i]
    ha_close = data['HA_Close'][i]

    # Buy signal (MACD crossover, RSI oversold, or Heiken Ashi trend)
    if (macd > signal_line and buy == 0 and sell == 0) or (rsi < 30 and buy == 0 and sell == 0) or (ha_close > ha_open and buy == 0 and sell == 0):
        print("Buy signal", close)
        buy = 1
        buy_price = close

    # Buy target
    if buy == 1 and close > buy_price + (buy_price * target):
        print("Buy target", close)
        pnl = (close - buy_price) * qnt
        profit += pnl
        print("PnL", pnl)
        buy = 0
        profit_trade += 1

    # Buy stop loss
    if buy == 1 and close < buy_price - (buy_price * stop_loss):
        print("Buy stop loss", close)
        pnl = (close - buy_price) * qnt
        loss += pnl
        print("PnL", pnl)
        buy = 0

    # Sell signal (MACD crossover, RSI overbought, or Heiken Ashi trend)
    if (macd < signal_line and sell == 0 and buy == 0) or (rsi > 70 and sell == 0 and buy == 0) or (ha_close < ha_open and sell == 0 and buy == 0):
        print("Sell signal", close)
        sell = 1
        sell_price = close

    # Sell target
    if sell == 1 and close < sell_price - (sell_price * target):
        print("Sell target", close)
        pnl = (sell_price - close) * qnt
        profit += pnl
        print("PnL", pnl)
        sell = 0
        profit_trade += 1

    # Sell stop loss
    if sell == 1 and close > sell_price + (sell_price * stop_loss):
        print("Sell stop loss", close)
        pnl = (sell_price - close) * qnt
        loss += pnl
        print("PnL", pnl)
        sell = 0

# Summary
print("Net PnL==", profit + loss)
print("Profitable trades==", profit_trade)
