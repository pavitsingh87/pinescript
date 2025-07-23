# Boolinger + RSI 
trades give 5 to 7 profit factor 
//@version=6
strategy("Bollinger + RSI, Double Strategy India", overlay=true)

// === INPUTS ===
RSIlength     = input.int(6, title="RSI Period Length")
RSIoverSold   = 50
RSIoverBought = 50
maxLossPct    = input.float(20.0, title="Max Loss %", minval=0.1)

BBlength = input.int(200, minval=1, title="Bollinger Period Length")
BBmult   = input.float(2.0, minval=0.001, maxval=50.0, title="Bollinger StdDev Multiplier")

switch1 = input.bool(true, title="Enable Bar Color?")
switch2 = input.bool(true, title="Enable Background Color?")

// === PRICE SERIES ===
price  = close
source = close

// === RSI ===
vrsi = ta.rsi(price, RSIlength)

// === BOLLINGER BANDS ===
BBbasis = ta.sma(source, BBlength)
BBdev   = BBmult * ta.stdev(source, BBlength)
BBupper = BBbasis + BBdev
BBlower = BBbasis - BBdev

// === SESSION FILTER ===
sessStart = timestamp("America/New_York", year, month, dayofmonth, 9, 0)
sessEnd   = timestamp("America/New_York", year, month, dayofmonth, 16, 0)

//sessStart = timestamp("Asia/Kolkatta", year, month, dayofmonth, 9, 15)
//sessEnd   = timestamp("Asia/Kolkatta", year, month, dayofmonth, 15, 30)


isSession = (time >= sessStart and time <= sessEnd)
isWeekday = (dayofweek >= dayofweek.monday and dayofweek <= dayofweek.friday)
inSession = isSession and isWeekday

// === SIGNALS ===
buyEntry  = ta.crossover(source, BBlower)
sellEntry = ta.crossunder(source, BBupper)

// === PLOT BOLLINGER BANDS ===
plot(BBbasis, color=color.aqua, title="BB Basis")
p1 = plot(BBupper, color=color.silver, title="BB Upper")
p2 = plot(BBlower, color=color.silver, title="BB Lower")
fill(p1, p2, color=color.new(color.silver, 90))

// === COLOR LOGIC ===
TrendColor = (vrsi > RSIoverBought and price[1] > BBupper and price < BBupper and BBbasis < BBbasis[1]) ? color.red :
             (vrsi < RSIoverSold and price[1] < BBlower and price > BBlower and BBbasis > BBbasis[1]) ? color.green : na

barcolor(switch1 ? TrendColor : na)
bgcolor(switch2 ? color.new(TrendColor, 50) : na)

// === STOP LOSS CALC ===
longSL  = price * (1 - maxLossPct / 100)
shortSL = price * (1 + maxLossPct / 100)
// === STRATEGY LOGIC ===
if not na(vrsi)
    // Long Entry
    if ta.crossover(vrsi, RSIoverSold) and buyEntry
        strategy.entry("RSI_BB_L", strategy.long, comment="RSI_BB_L",qty=100)
        
    else
        strategy.cancel("RSI_BB_L")

    // Short Entry
    if ta.crossunder(vrsi, RSIoverBought) and sellEntry
        strategy.entry("RSI_BB_S", strategy.short, comment="RSI_BB_S",qty=100)
        
    else
        strategy.cancel("RSI_BB_S")
    
// === DYNAMIC STOP-LOSS BASED ON ENTRY PRICE ===
longStopPrice  = strategy.position_avg_price * (1 - maxLossPct / 100)
shortStopPrice = strategy.position_avg_price * (1 + maxLossPct / 100)

// Apply exit only when in position
if strategy.position_size > 0
    strategy.exit("Long SL", from_entry="RSI_BB_L", stop=longStopPrice)

if strategy.position_size < 0
    strategy.exit("Short SL", from_entry="RSI_BB_S", stop=shortStopPrice)


