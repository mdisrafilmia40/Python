from binance.client import Client
from ta.momentum import RSIIndicator
from ta.volatility import BollingerBands
import pandas as pd
import time

# 🔐 Binance API Key
api_key = 'YOUR_API_KEY'
api_secret = 'YOUR_API_SECRET'
client = Client(api_key, api_secret)

symbol = 'BANANAS31USDT'
interval = Client.KLINE_INTERVAL_1HOUR
limit = 100

def get_technical_data():
    klines = client.get_klines(symbol=symbol, interval=interval, limit=limit)
    df = pd.DataFrame(klines, columns=[
        'timestamp', 'open', 'high', 'low', 'close', 'volume',
        'close_time', 'quote_asset_volume', 'number_of_trades',
        'taker_buy_base', 'taker_buy_quote', 'ignore'
    ])
    df['close'] = pd.to_numeric(df['close'])
    df['rsi'] = RSIIndicator(close=df['close'], window=6).rsi()
    bb = BollingerBands(close=df['close'], window=20)
    df['bb_upper'] = bb.bollinger_hband()
    df['bb_lower'] = bb.bollinger_lband()
    df['bb_middle'] = bb.bollinger_mavg()
    return df

def check_signal():
    df = get_technical_data()
    current_price = df['close'].iloc[-1]
    rsi = df['rsi'].iloc[-1]
    upper = df['bb_upper'].iloc[-1]
    lower = df['bb_lower'].iloc[-1]

    print(f"Price: {current_price:.6f}, RSI: {rsi:.2f}")

    if rsi > 70 or current_price >= upper:
        print("🔴 SELL ALERT!")
    elif rsi < 30 or current_price <= lower:
        print("🟢 BUY ALERT!")
    else:
        print("⚪ HOLD - No strong signal.")

while True:
    try:
        check_signal()
        time.sleep(300)  # প্রতি ৫ মিনিট পরপর চেক করবে
    except Exception as e:
        print("Error:", e)
        time.sleep(60)
