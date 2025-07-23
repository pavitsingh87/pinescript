# Boolinger + RSI 
trades give 5 to 7 profit factor 
//@version=6
strategy("Bollinger + RSI Nasdaq accont", shorttitle="CA_-_RSI_Bol_Strat_1.1", overlay=true)

// === Inputs ===
RSIlength     = input.int(6, title="RSI Period Length")
RSIoverSold   = 50
RSIoverBought = 50
price         = close

BBlength = input.int(200, minval=1, title="Bollinger Period Length")
BBmult   = 2.0

// === RSI Calculation ===
vrsi = ta.rsi(price, RSIlength)

// === Bollinger Bands Calculation ===
BBbasis = ta.sma(price, BBlength)
BBdev   = BBmult * ta.stdev(price, BBlength)
BBupper = BBbasis + BBdev
BBlower = BBbasis - BBdev
source  = close

// === Entry signals ===
buyEntry  = ta.crossover(source, BBlower)
sellEntry = ta.crossunder(source, BBupper)

// === Session Filter: NY 09:30–15:00 (Mon–Fri) ===
sessStart = timestamp("America/New_York", year, month, dayofmonth, 9, 30)
sessEnd   = timestamp("America/New_York", year, month, dayofmonth, 15, 0)
isInTime  = (time >= sessStart and time <= sessEnd)
isWeekday = (dayofweek >= dayofweek.monday and dayofweek <= dayofweek.friday)
inSession = isInTime and isWeekday

// === Plot Bollinger Bands ===
//plot(BBbasis, color=color.aqua, title="Bollinger Bands SMA Basis Line")
p1 = plot(BBupper, color=color.silver, title="Bollinger Bands Upper Line")
p2 = plot(BBlower, color=color.silver, title="Bollinger Bands Lower Line")
//fill(p1, p2)

// === Colors ===
switch1 = input.bool(true, title="Enable Bar Color?")
switch2 = input.bool(true, title="Enable Background Color?")
TrendColor = (vrsi > RSIoverBought and price[1] > BBupper and price < BBupper and BBbasis < BBbasis[1]) ? color.red :
             (vrsi < RSIoverSold   and price[1] < BBlower and price > BBlower and BBbasis > BBbasis[1]) ? color.green : na

barcolor(switch1 ? TrendColor : na)
bgcolor(switch2 ? color.new(TrendColor, 85) : na)

// === Strategy logic (with session filter) ===
if not na(vrsi) and inSession
    if ta.crossover(vrsi, RSIoverSold) and buyEntry
        strategy.entry("RSI_BB_L", strategy.long, qty=100, stop=BBlower, oca_name="RSI_BB", comment="RSI_BB_L")
    else
        strategy.cancel(id="RSI_BB_L")

    if ta.crossunder(vrsi, RSIoverBought) and sellEntry
        strategy.entry("RSI_BB_S", strategy.short, qty=100, stop=BBupper, oca_name="RSI_BB", comment="RSI_BB_S")
    else
        strategy.cancel(id="RSI_BB_S")

