import ccxt
import pandas as pd
import talib
import matplotlib.pyplot as plt

# Параметри
symbol = 'BTC/USDT'  # Торгівельна пара
timeframe = '1h'  # Таймфрейм (годинний графік)
limit = 100  # Кількість свічок для аналізу
short_period = 20  # Період для короткострокової SMA
long_period = 50  # Період для довгострокової SMA
volatility_period = 14  # Період для обчислення волатильності

# Підключення до біржі (Binance як приклад)
exchange = ccxt.binance()

# Отримання історичних даних
def get_historical_data(symbol, timeframe, limit):
    ohlcv = exchange.fetch_ohlcv(symbol, timeframe, limit=limit)
    df = pd.DataFrame(ohlcv, columns=['timestamp', 'open', 'high', 'low', 'close', 'volume'])
    df['timestamp'] = pd.to_datetime(df['timestamp'], unit='ms')
    df.set_index('timestamp', inplace=True)
    return df

# Обчислення нового індикатора (VTI)
def calculate_vti(df, short_period, long_period, volatility_period):
    # Волатильність (стандартне відхилення)
    df['volatility'] = df['close'].rolling(window=volatility_period).std()
    
    # Тренд (різниця між короткостроковою і довгостроковою SMA)
    df['short_sma'] = talib.SMA(df['close'], timeperiod=short_period)
    df['long_sma'] = talib.SMA(df['close'], timeperiod=long_period)
    df['trend'] = df['short_sma'] - df['long_sma']
    
    # Індикатор VTI (волатильність * тренд)
    df['vti'] = df['volatility'] * df['trend']
    
    return df

# Отримуємо дані
df = get_historical_data(symbol, timeframe, limit)
df = calculate_vti(df, short_period, long_period, volatility_period)

# Побудова графіку
plt.figure(figsize=(12, 6))

# Підграфік 1: Графік VTI
plt.subplot(2, 1, 1)
plt.plot(df.index, df['vti'], label='VTI (Volatility Trend Indicator)', color='purple', linewidth=1)
plt.title('Індикатор VTI (Volatility Trend Indicator)')
plt.xlabel('Час')
plt.ylabel('VTI')
plt.legend(loc='upper left')

# Підграфок 2: Ціна з SMA
plt.subplot(2, 1, 2)
plt.plot(df.index, df['close'], label='Ціна BTC/USDT', color='blue', linewidth=1)
plt.plot(df.index, df['short_sma'], label=f'{short_period}-період SMA', color='orange', linestyle='--', linewidth=1)
plt.plot(df.index, df['long_sma'], label=f'{long_period}-період SMA', color='green', linestyle='--', linewidth=1)
plt.title('Графік ціни BTC/USDT з скользячими середніми')
plt.xlabel('Час')
plt.ylabel('Ціна (USDT)')
plt.legend(loc='upper left')

# Показуємо графіки
plt.tight_layout()
plt.show()
