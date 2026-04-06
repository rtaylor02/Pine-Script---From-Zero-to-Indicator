# 📈 Pine Script: From Zero to Indicator — Complete Code Reference

> All scripts from the book, collected in one place for easy reference.
> Each script is numbered sequentially. Scripts are self-contained and can be
> copy-pasted directly into the **TradingView Pine Editor**.
>
> **TradingView Pine Editor:** [tradingview.com](https://www.tradingview.com) → Open any chart → `Pine Editor` tab at the bottom → Paste → `Add to chart`

---

## Table of Contents

- [Chapter 1 — Welcome to Pine Script](#chapter-1--welcome-to-pine-script)
- [Chapter 2 — Variables, Types, and Operators](#chapter-2--variables-types-and-operators)
- [Chapter 3 — The Series: Understanding Time in Pine Script](#chapter-3--the-series-understanding-time-in-pine-script)
- [Chapter 4 — Functions: Built-In and User-Defined](#chapter-4--functions-built-in-and-user-defined)
- [Chapter 5 — Conditional Logic and Loops](#chapter-5--conditional-logic-and-loops)
- [Chapter 6 — Drawing on the Chart](#chapter-6--drawing-on-the-chart)
- [Chapter 7 — Inputs: Making Your Scripts Interactive](#chapter-7--inputs-making-your-scripts-interactive)
- [Chapter 8 — Alerts](#chapter-8--alerts)
- [Chapter 9 — From Indicator to Strategy](#chapter-9--from-indicator-to-strategy)
- [Chapter 10 — Risk Management in Pine Strategies](#chapter-10--risk-management-in-pine-strategies)
- [Sample Indicators](#sample-indicators)
- [Sample Strategies](#sample-strategies)

---

## Chapter 1 — Welcome to Pine Script

### script1 — Anatomy of a Pine Script

> The minimal structure every Pine Script must have. Demonstrates `indicator()`, comments, and `plot()`.

```pine
//@version=5
indicator("My First Script", overlay=true)

// Your logic goes here
plot(close, title="Close Price", color=color.blue)
```

---

### script2 — Hello Chart: 20-Period SMA

> Your first real indicator. Calculates a 20-period Simple Moving Average and plots it as a blue line on the price chart.

```pine
//@version=5
indicator("Hello Chart - 20 SMA", overlay=true)

// Calculate a 20-period simple moving average of the close price
sma20 = ta.sma(close, 20)

// Plot it as a blue line
plot(sma20, title="SMA 20", color=color.blue, linewidth=2)
```

---

## Chapter 2 — Variables, Types, and Operators

### script3 — Variables Demo

> Demonstrates the six core data types: `int`, `float`, `bool`, `string`, `color`, and `series`.

```pine
//@version=5
indicator("Variables Demo", overlay=true)

myLength = 14          // integer variable
myFactor = 1.5         // float variable
myMessage = "Hello"   // string variable
isSignal  = true       // boolean variable

plot(ta.sma(close, myLength))
```

---

### script4 — The `var` Keyword

> Shows the difference between a normal variable (resets every bar) and a `var` variable (persists across bars). Uses `:=` for re-assignment.

```pine
//@version=5
indicator("var Demo", overlay=false)

// Without var: resets to 0 on every bar
normalCount = 0
normalCount := normalCount + 1   // This will always equal 1!

// With var: persists across bars
var persistCount = 0
persistCount := persistCount + 1  // This counts up correctly

plot(persistCount, title="Bar Count")
```

---

### script5 — Arithmetic Operators

> Calculates bar midpoint, range, and daily percentage change using arithmetic operators. Demonstrates working with OHLCV series.

```pine
//@version=5
indicator("Operators Demo", overlay=true)

midpoint  = (high + low) / 2        // Average of high and low
range_val = high - low               // Bar range
pctChange = (close - close[1]) / close[1] * 100  // Daily % change

plot(midpoint, title="Midpoint", color=color.orange)
plot(range_val, title="Range")
```

---

### script6 — Comparison and Logical Operators

> Combines `and` logic to highlight large bullish candles. Uses `bgcolor()` to shade those bars green.

```pine
//@version=5
indicator("Logic Demo", overlay=true)

isBullishCandle = close > open
isLargeBar      = (high - low) > ta.atr(14) * 1.5
isBullishLarge  = isBullishCandle and isLargeBar

bgcolor(isBullishLarge ? color.new(color.green, 85) : na)
```

---

### script7 — The Ternary Operator

> Uses the ternary operator `condition ? a : b` to color each bar green (bullish) or red (bearish).

```pine
//@version=5
indicator("Ternary Demo", overlay=true)

// Assign green if bullish, red if bearish
barColor = close >= open ? color.green : color.red
barcolor(barColor)
```

---

## Chapter 3 — The Series: Understanding Time in Pine Script

### script8 — The Series and History Operator

> Demonstrates accessing historical bar values with `[]`. Calculates 10-bar momentum.

```pine
//@version=5
indicator("Series Demo", overlay=false)

// close     = close price of the CURRENT bar  (same as close[0])
// close[1]  = close price of the PREVIOUS bar
// close[5]  = close price 5 bars ago

// Calculate momentum: difference between today's close and 10 bars ago
momentum = close - close[10]

plot(momentum, title="10-Bar Momentum")
```

---

### script9 — History Reference on a Calculated Series

> Applies `[]` to a user-calculated series (SMA) to detect whether the moving average is rising or falling. Colors the plot accordingly.

```pine
//@version=5
indicator("History Reference", overlay=true)

mySMA = ta.sma(close, 20)

// Was the SMA rising? Compare today's SMA to 3 bars ago
isRising = mySMA > mySMA[3]

// Plot the SMA, changing color based on direction
plot(mySMA, color = isRising ? color.lime : color.red, linewidth=2)
```

---

### script10 — Bar State Variables

> Shows how to use `barstate.isfirst` and `barstate.islast` to trigger actions on specific bars. Plots `bar_index` as a line.

```pine
//@version=5
indicator("Bar State Demo", overlay=false)

// Draw a label on the first bar
if barstate.isfirst
    label.new(bar_index, 0, "Start", style=label.style_label_right)

// Draw a label on the last bar
if barstate.islast
    label.new(bar_index, bar_index, str.format("End: {0}", bar_index), style=label.style_label_left)
    // ALTERNATIVE: use string concatenation: "End: " + str.tostring(bar_index)
    // label.new(bar_index, bar_index, "End: " + str.tostring(bar_index), style=label.style_label_left)

// Plot bar index as a line (useful for debugging)
plot(bar_index, title="Bar Index")
```

---

### script11 — Handling `na` Values

> Demonstrates how to detect and replace `na` values using `na()` and `nz()`. Applies to a 200-period SMA which cannot calculate until enough bars exist.

```pine
//@version=5
indicator("na Handling", overlay=false)

sma200 = ta.sma(close, 200)

// Safe: if sma200 is na, use 0 instead
safeSMA = nz(sma200, 0)

// Only plot when sma200 is actually valid
plot(na(sma200) ? na : sma200, title="SMA 200")
```

---

## Chapter 4 — Functions: Built-In and User-Defined

### script12 — EMA Crossover Signals with `ta.crossover`

> Uses `ta.crossover()` and `ta.crossunder()` to detect EMA crossings and plot buy/sell arrows on the chart.

```pine
//@version=5
indicator("Crossover Demo", overlay=true)

ema9  = ta.ema(close, 9)
ema21 = ta.ema(close, 21)

// Signal conditions
bullSignal = ta.crossover(ema9, ema21)   // 9 EMA crosses above 21 EMA
bearSignal = ta.crossunder(ema9, ema21)  // 9 EMA crosses below 21 EMA

plot(ema9,  "EMA 9",  color.blue)
plot(ema21, "EMA 21", color.orange)

plotshape(bullSignal, style=shape.triangleup,   location=location.belowbar, color=color.green, size=size.small)
plotshape(bearSignal, style=shape.triangledown, location=location.abovebar, color=color.red,   size=size.small)
```

---

### script13 — User-Defined Function (Single Return)

> Defines a reusable `percentChange(src, len)` function and calls it twice to plot 5-bar and 20-bar percentage changes.

```pine
//@version=5
indicator("Custom Functions", overlay=false)

// Function definition
percentChange(src, len) =>
    (src - src[len]) / src[len] * 100

// Function call
pct5  = percentChange(close, 5)
pct20 = percentChange(close, 20)

plot(pct5,  "5-bar % Change",  color.blue)
plot(pct20, "20-bar % Change", color.orange)
hline(0, "Zero Line", color.gray, linestyle=hline.style_dashed)
```

---

### script14 — User-Defined Function (Multiple Return Values)

> Demonstrates a function that returns a tuple of three values using `[upper, mid, lower]` — a Donchian Channel.

```pine
//@version=5
indicator("Multi-return Function", overlay=true)

// Returns both upper and lower channel
donchian(len) =>
    upper = ta.highest(high, len)
    lower = ta.lowest(low, len)
    mid   = (upper + lower) / 2
    [upper, mid, lower]   // return all three

[dUpper, dMid, dLower] = donchian(20)

plot(dUpper, "Upper", color.blue)
plot(dMid,   "Mid",   color.gray, style=plot.style_dashed)
plot(dLower, "Lower", color.red)
```

---

## Chapter 5 — Conditional Logic and Loops

### script15 — The `if / else if / else` Statement

> Shows a multi-branch `if` statement that sets signal values based on RSI zone (oversold / overbought / neutral).

```pine
//@version=5
indicator("If Demo", overlay=false)

rsi = ta.rsi(close, 14)

// Single condition
oversold  = 0.0
overbought = 0.0

if rsi < 30
    oversold := 1.0
else if rsi > 70
    overbought := 1.0
else
    oversold  := 0.0
    overbought := 0.0

plot(oversold,  "Oversold Signal",  color.green)
plot(overbought, "Overbought Signal", color.red)
hline(0.5, "Threshold", color.gray)
```

---

### script16 — The `switch` Statement

> Uses `switch` to classify the RSI into named zones and display the zone as a label on the last bar.

```pine
//@version=5
indicator("Switch Demo", overlay=false)

rsi = ta.rsi(close, 14)

zone = switch
    rsi >= 70   => "Overbought"
    rsi <= 30   => "Oversold"
    rsi >= 50   => "Bullish Zone"
    =>             "Bearish Zone"

// Display zone as a label on the last bar
if barstate.islast
    label.new(bar_index, rsi, zone, style=label.style_label_left)
```

---

### script17 — The `for` Loop

> Manually calculates a Simple Moving Average using a `for` loop, then plots it alongside the built-in `ta.sma()` to confirm they match.

```pine
//@version=5
indicator("For Loop Demo", overlay=false)

// Manually calculate a simple moving average using a loop
len = 10
total = 0.0
for i = 0 to len - 1
    total := total + close[i]

manualSMA = total / len
builtinSMA = ta.sma(close, len)

plot(manualSMA, "Manual SMA",   color.orange, linewidth=2)
plot(builtinSMA, "Built-in SMA", color.blue,   linewidth=1)
```

---

### script18 — The `while` Loop

> Uses a `while` loop to count how many bars back you need to go to find a lower closing price than the current bar.

```pine
//@version=5
indicator("While Loop Demo", overlay=false)

// Find how many bars back we need to go to find a lower close
len = 0
while close[len + 1] < close and len < 100
    len := len + 1

plot(len, "Bars Since Last Lower Close")
```

---

### script19 — Complex Multi-Condition Entry Logic

> Combines trend filter, RSI filter, and pullback detection using `and` logic to create a high-quality bull setup signal.

```pine
//@version=5
indicator("Complex Conditions", overlay=true)

ema50  = ta.ema(close, 50)
ema200 = ta.ema(close, 200)
rsi    = ta.rsi(close, 14)
atr    = ta.atr(14)

// Bull setup: price above both EMAs, RSI not overbought, recent pullback
aboveTrend   = close > ema50 and close > ema200
rsiNotHigh   = rsi < 65
hadPullback  = low[1] < ema50[1] or low[2] < ema50[2]

bullSetup = aboveTrend and rsiNotHigh and hadPullback

plotshape(bullSetup, style=shape.triangleup, location=location.belowbar, 
          color=color.lime, size=size.normal, title="Bull Setup")
```

---

## Chapter 6 — Drawing on the Chart

### script20 — Plot Styles and `fill()`

> Shows multiple `plot()` style options and how to use `fill()` to shade the area between two plot lines with dynamic color.

```pine
//@version=5
indicator("Plot Styles Demo", overlay=true)

sma20 = ta.sma(close, 20)
sma50 = ta.sma(close, 50)

// Different plot styles
plot(sma20, "SMA 20", color.blue,   linewidth=2, style=plot.style_line)
plot(sma50, "SMA 50", color.orange, linewidth=1, style=plot.style_line)

// Area fill between two plots
p1 = plot(sma20, display=display.none)
p2 = plot(sma50, display=display.none)
fill(p1, p2, color = sma20 > sma50 ? color.new(color.green, 85) : color.new(color.red, 85))
```

---

### script21 — `plotshape()` — Arrows and Text

> Uses `plotshape()` to draw buy/sell arrows with text labels at RSI crossover points.

```pine
//@version=5
indicator("plotshape Demo", overlay=true)

rsi = ta.rsi(close, 14)

buySignal  = ta.crossover(rsi, 30)
sellSignal = ta.crossunder(rsi, 70)

// Buy arrow below bar
plotshape(buySignal, title="Buy",  style=shape.arrowup,
          location=location.belowbar, color=color.green,
          size=size.normal, text="BUY")

// Sell arrow above bar
plotshape(sellSignal, title="Sell", style=shape.arrowdown,
          location=location.abovebar, color=color.red,
          size=size.normal, text="SELL")
```

---

### script22 — `label.new()` — Custom Text Labels

> Creates dynamic price labels at 20-bar high pivot points using `label.new()` with full style control.

```pine
//@version=5
indicator("Labels Demo", overlay=true)

atr  = ta.atr(14)
high20 = ta.highest(high, 20)

// Label at 20-bar high
if high == high20
    label.new(
        x     = bar_index,
        y     = high,
        text  = "20H: " + str.tostring(high, "#.####"),
        style = label.style_label_down,
        color = color.navy,
        textcolor = color.white,
        size  = size.small
    )
```

---

### script23 — `line.new()` — Dynamic Trend Lines

> Draws a persistent horizontal line extending right from each confirmed swing high, using `var` to maintain the line object across bars.

```pine
//@version=5
indicator("Trendlines Demo", overlay=true, max_lines_count=50)

// Draw a line from the last swing high to current bar
var line trendLine = na

swingHigh = high[2] > high[1] and high[2] > high[3] and 
            high[2] > high[0] and high[2] > high[4]

if swingHigh
    trendLine := line.new(
        x1    = bar_index[2],
        y1    = high[2],
        x2    = bar_index,
        y2    = high[2],
        color = color.blue,
        width = 2,
        extend = extend.right
    )
```

---

### script24 — `bgcolor()` — Session Highlighting

> Uses `bgcolor()` and `time()` to highlight the London trading session (08:00–17:00 UTC) with a subtle blue background.

```pine
//@version=5
indicator("Sessions Demo", overlay=true)

// Highlight the London session (08:00–17:00 UTC) in blue
isLondon = not na(time(timeframe.period, "0800-1700", "Europe/London"))
bgcolor(isLondon ? color.new(color.blue, 92) : na, title="London Session")
```

---

### script25 — `hline()` — Static Reference Levels

> Plots RSI with standard overbought (70), midline (50), and oversold (30) levels using `hline()`.

```pine
//@version=5
indicator("RSI with Levels", overlay=false)
rsi = ta.rsi(close, 14)
plot(rsi, "RSI", color.purple, linewidth=2)

hline(70, "Overbought", color.red,   linestyle=hline.style_dashed)
hline(50, "Midline",    color.gray,  linestyle=hline.style_dotted)
hline(30, "Oversold",   color.green, linestyle=hline.style_dashed)
```

---

## Chapter 7 — Inputs: Making Your Scripts Interactive

### script26 — `input.int()` and `input.float()`

> Exposes EMA lengths and ATR multiplier as user-adjustable settings using `input.int()` and `input.float()` with min/max constraints.

```pine
//@version=5
indicator("Inputs Demo", overlay=true)

// Integer input with a slider-like feel
fastLen = input.int(9,   "Fast EMA Length", minval=1, maxval=500, step=1)
slowLen = input.int(21,  "Slow EMA Length", minval=1, maxval=500, step=1)

// Float input
atrMult = input.float(1.5, "ATR Multiplier", minval=0.1, maxval=10.0, step=0.1)

ema_fast = ta.ema(close, fastLen)
ema_slow = ta.ema(close, slowLen)
atr      = ta.atr(14)

plot(ema_fast, "Fast EMA", color.blue)
plot(ema_slow, "Slow EMA", color.orange)
```

---

### script27 — `input.bool()` and `input.color()`

> Adds a toggle to show/hide the midline and a color picker for the fast EMA line.

```pine
//@version=5
indicator("Bool & Color Inputs", overlay=true)

showMidline  = input.bool(true,      "Show Midline")
lineColor    = input.color(color.blue, "Line Color")
fastLen      = input.int(9, "Fast Length")
slowLen      = input.int(21, "Slow Length")

ema_fast = ta.ema(close, fastLen)
ema_slow = ta.ema(close, slowLen)
midline  = (ema_fast + ema_slow) / 2

plot(ema_fast, "Fast", lineColor)
plot(ema_slow, "Slow", color.orange)
plot(showMidline ? midline : na, "Midline", color.gray, style=plot.style_dashed)
```

---

### script28 — `input.string()` with Dropdown Options

> Creates an MA type selector dropdown (SMA / EMA / WMA) using `input.string()` and routes the choice through a `switch` statement.

```pine
//@version=5
indicator("String Input Demo", overlay=true)

maType = input.string("EMA", "MA Type", options=["SMA", "EMA", "WMA"])
len    = input.int(20, "Length")

maValue = switch maType
    "SMA" => ta.sma(close, len)
    "EMA" => ta.ema(close, len)
    "WMA" => ta.wma(close, len)

plot(maValue, "MA", color.teal, linewidth=2)
```

---

### script29 — `input.source()`

> Allows the user to choose which price series (close, open, hl2, etc.) to feed into the moving average.

```pine
//@version=5
indicator("Source Input Demo", overlay=true)

src = input.source(close, "Source Price")
len = input.int(20, "Period")

plot(ta.sma(src, len), "MA", color.blue)
```

---

### script30 — Grouped Inputs with RSI Filter Toggle

> Demonstrates `group=` to organize inputs into collapsible sections. Combines EMA crossover signals with an optional RSI filter.

```pine
//@version=5
indicator("Grouped Inputs Demo", overlay=true)

// Group 1: Moving Averages
fastLen   = input.int(9,  "Fast Length",   group="Moving Averages")
slowLen   = input.int(21, "Slow Length",   group="Moving Averages")

// Group 2: RSI Filter
useRSI    = input.bool(true, "Enable RSI Filter", group="RSI Filter")
rsiLen    = input.int(14, "RSI Period",     group="RSI Filter")
rsiOB     = input.int(70, "Overbought",     group="RSI Filter")
rsiOS     = input.int(30, "Oversold",       group="RSI Filter")

ema_f = ta.ema(close, fastLen)
ema_s = ta.ema(close, slowLen)
rsi   = ta.rsi(close, rsiLen)

bull = ta.crossover(ema_f, ema_s) and (not useRSI or rsi < rsiOB)
bear = ta.crossunder(ema_f, ema_s) and (not useRSI or rsi > rsiOS)

plot(ema_f, "Fast", color.blue)
plot(ema_s, "Slow", color.orange)
plotshape(bull, style=shape.triangleup,   location=location.belowbar, color=color.green)
plotshape(bear, style=shape.triangledown, location=location.abovebar, color=color.red)
```

---

## Chapter 8 — Alerts

### script31 — `alertcondition()` — Registering Alert Conditions

> Registers two named alert conditions (bullish cross, bearish cross) using `alertcondition()`. Users activate them manually from TradingView's alert dialog.

```pine
//@version=5
indicator("Alerts Demo", overlay=true)

ema9  = ta.ema(close, 9)
ema21 = ta.ema(close, 21)

bullCross = ta.crossover(ema9, ema21)
bearCross = ta.crossunder(ema9, ema21)

plot(ema9,  "EMA 9",  color.blue)
plot(ema21, "EMA 21", color.orange)

plotshape(bullCross, style=shape.triangleup,   location=location.belowbar, color=color.green)
plotshape(bearCross, style=shape.triangledown, location=location.abovebar, color=color.red)

// Register alert conditions
alertcondition(bullCross, title="Bullish Cross", message="EMA 9 crossed above EMA 21 on {{ticker}}")
alertcondition(bearCross, title="Bearish Cross", message="EMA 9 crossed below EMA 21 on {{ticker}}")
```

---

### script32 — `alert()` — Programmatic Divergence Alert

> Fires an `alert()` directly from script logic when a bearish RSI divergence pattern is detected. Uses `alert.freq_once_per_bar`.

```pine
//@version=5
indicator("Programmatic Alert", overlay=true, alert_handler_enabled=true)

rsi = ta.rsi(close, 14)

// Divergence: price makes higher high but RSI makes lower high
priceHH = high > ta.highest(high, 10)[1]
rsiLH   = rsi  < ta.highest(rsi, 10)[1]

bearDivergence = priceHH and rsiLH

if bearDivergence
    alert("Bearish RSI Divergence on " + syminfo.ticker + 
          " | Price: " + str.tostring(close, "#.####"),
          alert.freq_once_per_bar)
```

---

### script33 — Webhook Alert for Automated Trading

> Demonstrates a JSON-formatted alert message designed for use with trading bots via TradingView's webhook system.

```pine
//@version=5
strategy("Webhook Example", overlay=true)

ema9  = ta.ema(close, 9)
ema21 = ta.ema(close, 21)

if ta.crossover(ema9, ema21)
    strategy.entry("Long", strategy.long)
    alert('{"action":"buy","symbol":"' + syminfo.ticker + 
          '","price":' + str.tostring(close) + '}',
          alert.freq_once_per_bar)
```

---

## Chapter 9 — From Indicator to Strategy

### script34 — The `strategy()` Declaration

> Shows the full `strategy()` declaration with all key parameters: capital, position sizing type, commission, and slippage.

```pine
//@version=5
strategy(
    title             = "My First Strategy",
    overlay           = true,
    initial_capital   = 10000,       // Starting capital in account currency
    default_qty_type  = strategy.percent_of_equity,
    default_qty_value = 10,          // Risk 10% of equity per trade
    commission_type   = strategy.commission.percent,
    commission_value  = 0.07,        // 0.07% per trade (typical Forex spread)
    slippage          = 2            // 2 ticks of slippage
)
```

---

### script35 — EMA Cross Strategy with `strategy.entry()` and `strategy.close()`

> A complete trend-following strategy using EMA crossovers. Enters long on bullish cross, closes on bearish cross. Foundation of most trend strategies.

```pine
//@version=5
strategy("EMA Cross Strategy", overlay=true,
         initial_capital=10000, default_qty_type=strategy.percent_of_equity,
         default_qty_value=10, commission_value=0.07)

fastLen = input.int(9,  "Fast EMA")
slowLen = input.int(21, "Slow EMA")

ema_f = ta.ema(close, fastLen)
ema_s = ta.ema(close, slowLen)

// Entry conditions
bullCross = ta.crossover(ema_f, ema_s)
bearCross = ta.crossunder(ema_f, ema_s)

// Enter long on bullish cross
if bullCross
    strategy.entry("Long", strategy.long)

// Close long (or enter short) on bearish cross
if bearCross
    strategy.close("Long")
    // strategy.entry("Short", strategy.short)  // Uncomment for short trades

plot(ema_f, "Fast EMA", color.blue)
plot(ema_s, "Slow EMA", color.orange)
```

---

### script36 — ATR-Based Stop Loss and Take Profit with `strategy.exit()`

> Adds dynamic ATR-based stop loss and take profit to a trend strategy. Uses `strategy.exit()` to define both exit levels in one call.

```pine
//@version=5
strategy("SL/TP Strategy", overlay=true,
         initial_capital=10000, default_qty_type=strategy.percent_of_equity,
         default_qty_value=5)

ema50  = ta.ema(close, 50)
atr    = ta.atr(14)
rsi    = ta.rsi(close, 14)

// ATR-based SL/TP
slMult = input.float(1.5, "SL ATR Multiplier")
tpMult = input.float(3.0, "TP ATR Multiplier")

longCondition = ta.crossover(close, ema50) and rsi > 50

if longCondition
    slPrice = close - atr * slMult
    tpPrice = close + atr * tpMult
    strategy.entry("Long", strategy.long)
    strategy.exit("Long Exit", "Long",
                  stop  = slPrice,
                  limit = tpPrice)

plot(ema50, "EMA 50", color.orange)
```

---

## Chapter 10 — Risk Management in Pine Strategies

### script37 — ATR-Based Dynamic Position Sizing

> Calculates position size dynamically so that the dollar risk per trade remains constant regardless of volatility. Uses `strategy.fixed` with a computed `qty`.

```pine
//@version=5
strategy("ATR Position Sizing", overlay=true,
         initial_capital = 10000,
         default_qty_type = strategy.fixed,
         default_qty_value = 1)

riskPerTrade = input.float(1.0, "Risk % Per Trade", minval=0.1, maxval=5.0)
atrMult      = input.float(1.5, "SL ATR Multiplier")
src          = input.source(close, "Source")
len          = input.int(20, "EMA Length")

ema  = ta.ema(src, len)
atr  = ta.atr(14)

// Calculate position size dynamically
riskAmount  = strategy.equity * (riskPerTrade / 100)
slDistance  = atr * atrMult
positionQty = math.floor(riskAmount / (slDistance / syminfo.mintick * syminfo.pointvalue))

longCond = ta.crossover(close, ema)

if longCond
    strategy.entry("Long", strategy.long, qty = positionQty)
    strategy.exit("Long Exit", "Long",
                  stop  = close - slDistance,
                  limit = close + slDistance * 2)

plot(ema, "EMA", color.teal, linewidth=2)
```

---

### script38 — Maximum Daily Loss Limit

> Tracks daily starting equity and halts all trading for the remainder of the day once the drawdown exceeds a configurable percentage threshold.

```pine
//@version=5
strategy("Daily Loss Limit", overlay=true,
         initial_capital=10000, default_qty_type=strategy.percent_of_equity,
         default_qty_value=5)

maxDailyLoss = input.float(2.0, "Max Daily Loss %", minval=0.5, maxval=10.0)

// Track daily starting equity
var float dayStartEquity = na
var bool  tradingHalted  = false

isNewDay = dayofweek != dayofweek[1]
if isNewDay
    dayStartEquity := strategy.equity
    tradingHalted  := false

if not na(dayStartEquity)
    dailyPnLPct = (strategy.equity - dayStartEquity) / dayStartEquity * 100
    if dailyPnLPct < -maxDailyLoss
        tradingHalted := true

ema9  = ta.ema(close, 9)
ema21 = ta.ema(close, 21)

bullCross = ta.crossover(ema9, ema21) and not tradingHalted

if bullCross
    strategy.entry("Long", strategy.long)

if ta.crossunder(ema9, ema21)
    strategy.close("Long")
```

---

## Sample Indicators

### script39 — Triple EMA Trend Dashboard

> Plots three EMAs with configurable periods, shades the chart background based on trend alignment, adds an ATR volatility band, and registers bullish/bearish alignment alerts.

```pine
//@version=5
indicator("Triple EMA Trend Dashboard", overlay=true)

// ── Inputs
fastLen  = input.int(9,   "Fast EMA",   minval=1, group="EMAs")
medLen   = input.int(21,  "Medium EMA", minval=1, group="EMAs")
slowLen  = input.int(55,  "Slow EMA",   minval=1, group="EMAs")
atrLen   = input.int(14,  "ATR Period", minval=1, group="Volatility")
atrBand  = input.float(1.0, "ATR Band Width", minval=0.1, step=0.1, group="Volatility")
showBand = input.bool(true, "Show ATR Band", group="Volatility")

// ── Calculations
ema_f = ta.ema(close, fastLen)
ema_m = ta.ema(close, medLen)
ema_s = ta.ema(close, slowLen)
atr   = ta.atr(atrLen)

bandUpper = ema_m + atr * atrBand
bandLower = ema_m - atr * atrBand

// ── Trend State
bullTrend = ema_f > ema_m and ema_m > ema_s
bearTrend = ema_f < ema_m and ema_m < ema_s

// ── Plots
plot(ema_f, "Fast EMA",   color.new(color.blue,   0), linewidth=1)
plot(ema_m, "Medium EMA", color.new(color.orange, 0), linewidth=2)
plot(ema_s, "Slow EMA",   color.new(color.red,    0), linewidth=3)

ubPlot = plot(showBand ? bandUpper : na, "Band Upper", color.new(color.gray, 70))
lbPlot = plot(showBand ? bandLower : na, "Band Lower", color.new(color.gray, 70))
fill(ubPlot, lbPlot, color.new(color.blue, 90))

bgcolor(bullTrend ? color.new(color.green, 90) :
        bearTrend ? color.new(color.red,   90) : na,
        title="Trend Background")

// ── Alerts
alertcondition(ta.crossover(ema_f, ema_m) and ema_m > ema_s,
    "Bull Alignment", "Triple EMA Bull Alignment on {{ticker}}")
alertcondition(ta.crossunder(ema_f, ema_m) and ema_m < ema_s,
    "Bear Alignment", "Triple EMA Bear Alignment on {{ticker}}")
```

---

### script40 — RSI Divergence Detector

> Identifies bullish and bearish RSI divergences using `ta.pivothigh()` / `ta.pivotlow()` and `ta.valuewhen()`. Labels divergences directly on the RSI pane.

```pine
//@version=5
indicator("RSI Divergence Detector", overlay=false)

// ── Inputs
rsiLen  = input.int(14, "RSI Length", minval=2)
obLevel = input.int(70, "Overbought", minval=50, maxval=95)
osLevel = input.int(30, "Oversold",   minval=5,  maxval=50)
lbBars  = input.int(5,  "Lookback for Divergence", minval=3, maxval=20)

// ── RSI
rsi = ta.rsi(close, rsiLen)

// ── Swing detection
pivotHigh_price = ta.pivothigh(high, lbBars, lbBars)
pivotLow_price  = ta.pivotlow(low,  lbBars, lbBars)
pivotHigh_rsi   = ta.pivothigh(rsi,  lbBars, lbBars)
pivotLow_rsi    = ta.pivotlow(rsi,   lbBars, lbBars)

// ── Divergence conditions
// Bearish: price makes higher high, RSI makes lower high
bearDiv = not na(pivotHigh_price) and not na(pivotHigh_rsi) and
          high[lbBars] > ta.valuewhen(not na(pivotHigh_price), high[lbBars], 1) and
          rsi[lbBars]  < ta.valuewhen(not na(pivotHigh_rsi),  rsi[lbBars],  1)

// Bullish: price makes lower low, RSI makes higher low
bullDiv = not na(pivotLow_price) and not na(pivotLow_rsi) and
          low[lbBars]  < ta.valuewhen(not na(pivotLow_price), low[lbBars],  1) and
          rsi[lbBars]  > ta.valuewhen(not na(pivotLow_rsi),  rsi[lbBars],   1)

// ── Plot
plot(rsi, "RSI", color.purple, linewidth=2)
hline(obLevel, "Overbought", color.red,   linestyle=hline.style_dashed)
hline(50,      "Midline",    color.gray,  linestyle=hline.style_dotted)
hline(osLevel, "Oversold",   color.green, linestyle=hline.style_dashed)

plotshape(bearDiv, "Bearish Div", shape.labeldown, location.absolute,
          color.red, text="Bear Div", textcolor=color.white,
          offset=-lbBars, size=size.small)

plotshape(bullDiv, "Bullish Div", shape.labelup, location.absolute,
          color.green, text="Bull Div", textcolor=color.white,
          offset=-lbBars, size=size.small)

bgcolor(rsi > obLevel ? color.new(color.red,   90) :
        rsi < osLevel ? color.new(color.green, 90) : na)
```

---

### script41 — Bollinger Bands Plus with Squeeze Detection

> Enhanced Bollinger Bands indicator displaying %B (price position within bands), detecting BB Squeeze (compression) and Squeeze Release breakout events.

```pine
//@version=5
indicator("BB Plus - Volatility Adapted", overlay=false)

// ── Inputs
src    = input.source(close, "Source")
bbLen  = input.int(20,   "BB Length",     minval=5)
bbMult = input.float(2.0,"BB Multiplier", minval=0.5, step=0.1)
sqzLen = input.int(20,   "Squeeze Lookback", minval=5)

// ── Bollinger Bands
[bbMid, bbUpper, bbLower] = ta.bb(src, bbLen, bbMult)
bbWidth  = (bbUpper - bbLower) / bbMid * 100   // % width relative to midline

// ── Squeeze Detection
minWidth  = ta.lowest(bbWidth, sqzLen)
maxWidth  = ta.highest(bbWidth, sqzLen)
isSqueezeOn  = bbWidth == minWidth
isSqueezeOff = bbWidth[1] == ta.lowest(bbWidth, sqzLen)[1] and not isSqueezeOn

// ── %B: position of price within the bands (0 = lower, 1 = upper, 0.5 = mid)
percentB = (src - bbLower) / (bbUpper - bbLower) * 100

// ── Plots
plot(percentB, "%B", color.blue, linewidth=2)
hline(100, "Upper Band",  color.red,   linestyle=hline.style_dashed)
hline(50,  "Midline",     color.gray,  linestyle=hline.style_dotted)
hline(0,   "Lower Band",  color.green, linestyle=hline.style_dashed)

// Highlight squeeze
bgcolor(isSqueezeOn ? color.new(color.orange, 80) : na, title="Squeeze Active")
plotshape(isSqueezeOff, "Squeeze Release", shape.diamond,
          location.absolute, color.orange, size=size.small,
          text="SQZ OFF")

alertcondition(isSqueezeOff, "BB Squeeze Release",
    "Bollinger Band Squeeze ending on {{ticker}} — potential breakout")
```

---

### script42 — Multi-Timeframe Trend Strength Meter

> Reads EMA slope on three user-selectable timeframes using `request.security()` and displays a color-coded trend score table in the chart corner.

```pine
//@version=5
indicator("MTF Trend Strength Meter", overlay=true)

// ── Inputs
tf1    = input.timeframe("D",  "Timeframe 1")
tf2    = input.timeframe("240","Timeframe 2")
tf3    = input.timeframe("60", "Timeframe 3")
emaLen = input.int(20, "EMA Length")

// ── Helper: get EMA on a given timeframe and check slope
trendScore(tf) =>
    e  = request.security(syminfo.tickerid, tf, ta.ema(close, emaLen))
    e1 = request.security(syminfo.tickerid, tf, ta.ema(close, emaLen)[1])
    score = e > e1 ? 1 : e < e1 ? -1 : 0
    [score, e]

[s1, e1Val] = trendScore(tf1)
[s2, e2Val] = trendScore(tf2)
[s3, e3Val] = trendScore(tf3)

totalScore = s1 + s2 + s3   // Range: -3 to +3

// ── Table
var table t = table.new(position.top_right, 4, 5, bgcolor=color.new(color.navy, 10), border_width=1)

scoreColor(s) => s > 0 ? color.lime : s < 0 ? color.red : color.gray

if barstate.islast
    table.cell(t, 0, 0, "Timeframe",    text_color=color.white, bgcolor=color.navy)
    table.cell(t, 1, 0, "EMA",          text_color=color.white, bgcolor=color.navy)
    table.cell(t, 2, 0, "Trend",        text_color=color.white, bgcolor=color.navy)

    table.cell(t, 0, 1, tf1,           text_color=color.white)
    table.cell(t, 1, 1, str.tostring(e1Val, "#.####"), text_color=color.white)
    table.cell(t, 2, 1, s1 > 0 ? "▲ Bull" : s1 < 0 ? "▼ Bear" : "◆ Flat",
               text_color=scoreColor(s1))

    table.cell(t, 0, 2, tf2,           text_color=color.white)
    table.cell(t, 1, 2, str.tostring(e2Val, "#.####"), text_color=color.white)
    table.cell(t, 2, 2, s2 > 0 ? "▲ Bull" : s2 < 0 ? "▼ Bear" : "◆ Flat",
               text_color=scoreColor(s2))

    table.cell(t, 0, 3, tf3,           text_color=color.white)
    table.cell(t, 1, 3, str.tostring(e3Val, "#.####"), text_color=color.white)
    table.cell(t, 2, 3, s3 > 0 ? "▲ Bull" : s3 < 0 ? "▼ Bear" : "◆ Flat",
               text_color=scoreColor(s3))

    totalColor = totalScore >= 2 ? color.lime : totalScore <= -2 ? color.red : color.orange
    table.cell(t, 0, 4, "TOTAL",        text_color=color.white, bgcolor=color.navy)
    table.cell(t, 2, 4, str.tostring(totalScore) + " / 3",
               text_color=totalColor, text_size=size.normal)

// Color current bar based on confluence
barcolor(totalScore >= 2 ? color.lime : totalScore <= -2 ? color.red : na)
```

---

### script43 — Market Structure: Swing Highs/Lows with Break of Structure

> Detects swing highs and lows using `ta.pivothigh()` / `ta.pivotlow()`, draws dashed levels at each, and signals Break of Structure (BOS) when price breaks through a prior swing.

```pine
//@version=5
indicator("Market Structure BOS", overlay=true, max_lines_count=100, max_labels_count=50)

// ── Inputs
swingLen  = input.int(5, "Swing Detection Length", minval=2, maxval=20)
showLines = input.bool(true, "Show Level Lines")

// ── Swing point detection
pivHigh = ta.pivothigh(high, swingLen, swingLen)
pivLow  = ta.pivotlow(low,  swingLen, swingLen)

// ── Store last swing levels
var float lastSwingHigh = na
var float lastSwingLow  = na
var line  shLine = na
var line  slLine = na

// When a new swing high is confirmed
if not na(pivHigh)
    lastSwingHigh := pivHigh
    if showLines
        if not na(shLine)
            line.set_x2(shLine, bar_index)
            line.set_extend(shLine, extend.none)
        shLine := line.new(bar_index[swingLen], pivHigh, bar_index, pivHigh,
                           color=color.red, width=1, style=line.style_dashed,
                           extend=extend.right)
        label.new(bar_index[swingLen], pivHigh, "SH", style=label.style_label_down,
                  color=color.red, textcolor=color.white, size=size.tiny)

// When a new swing low is confirmed
if not na(pivLow)
    lastSwingLow := pivLow
    if showLines
        if not na(slLine)
            line.set_x2(slLine, bar_index)
            line.set_extend(slLine, extend.none)
        slLine := line.new(bar_index[swingLen], pivLow, bar_index, pivLow,
                           color=color.green, width=1, style=line.style_dashed,
                           extend=extend.right)
        label.new(bar_index[swingLen], pivLow, "SL", style=label.style_label_up,
                  color=color.green, textcolor=color.white, size=size.tiny)

// ── Break of Structure Detection
bosBull = not na(lastSwingHigh) and close > lastSwingHigh and close[1] <= lastSwingHigh
bosBear = not na(lastSwingLow)  and close < lastSwingLow  and close[1] >= lastSwingLow

plotshape(bosBull, "BOS Bull", shape.arrowup,   location.belowbar, color.lime,
          size=size.normal, text="BOS ▲")
plotshape(bosBear, "BOS Bear", shape.arrowdown, location.abovebar, color.red,
          size=size.normal, text="BOS ▼")

alertcondition(bosBull, "Bullish BOS", "Bullish Break of Structure on {{ticker}}")
alertcondition(bosBear, "Bearish BOS", "Bearish Break of Structure on {{ticker}}")
```

---

## Sample Strategies

### script44 — EMA Crossover with RSI Filter Strategy

> Trend-following strategy entering on EMA crossovers, filtered by RSI zone to avoid chasing extreme momentum. ATR-based SL/TP. Best on 1H/4H Forex.

```pine
//@version=5
strategy("EMA Cross + RSI Filter", overlay=true,
         initial_capital=10000, default_qty_type=strategy.percent_of_equity,
         default_qty_value=5, commission_value=0.07, slippage=2)

// ── Inputs
fastLen  = input.int(9,   "Fast EMA",     group="Entry")
slowLen  = input.int(21,  "Slow EMA",     group="Entry")
rsiLen   = input.int(14,  "RSI Length",   group="Entry")
rsiLong  = input.int(65,  "RSI Max Long", group="Entry")
rsiShort = input.int(35,  "RSI Min Short",group="Entry")
slMult   = input.float(1.5,"SL ATR Mult", group="Exit")
tpMult   = input.float(3.0,"TP ATR Mult", group="Exit")
useShort = input.bool(false,"Enable Short Trades", group="Entry")

// ── Indicators
ema_f = ta.ema(close, fastLen)
ema_s = ta.ema(close, slowLen)
rsi   = ta.rsi(close, rsiLen)
atr   = ta.atr(14)

// ── Conditions
longCond  = ta.crossover(ema_f, ema_s)  and rsi > 45 and rsi < rsiLong
shortCond = ta.crossunder(ema_f, ema_s) and rsi < 55 and rsi > rsiShort and useShort

// ── Orders
if longCond
    strategy.entry("Long", strategy.long)
    strategy.exit("Long X", "Long", stop=close - atr*slMult, limit=close + atr*tpMult)

if shortCond
    strategy.entry("Short", strategy.short)
    strategy.exit("Short X", "Short", stop=close + atr*slMult, limit=close - atr*tpMult)

// ── Plots
plot(ema_f, "Fast EMA", color.blue,   linewidth=1)
plot(ema_s, "Slow EMA", color.orange, linewidth=2)
plotshape(longCond,  style=shape.triangleup,   location=location.belowbar, color=color.lime, size=size.small)
plotshape(shortCond, style=shape.triangledown, location=location.abovebar, color=color.red,  size=size.small)
```

---

### script45 — Bollinger Band Mean Reversion Strategy

> Counter-trend strategy that buys the lower BB touch and sells the upper BB touch when ADX confirms a ranging (non-trending) market. Exits at the midline.

```pine
//@version=5
strategy("BB Mean Reversion", overlay=true,
         initial_capital=10000, default_qty_type=strategy.percent_of_equity,
         default_qty_value=5, commission_value=0.07, slippage=2)

// ── Inputs
bbLen    = input.int(20,  "BB Length",      group="Bollinger Bands")
bbMult   = input.float(2.0,"BB Multiplier", group="Bollinger Bands")
rsiLen   = input.int(14,  "RSI Length",     group="RSI Confirmation")
rsiOB    = input.int(70,  "RSI Overbought", group="RSI Confirmation")
rsiOS    = input.int(30,  "RSI Oversold",   group="RSI Confirmation")
adxLen   = input.int(14,  "ADX Length",     group="Range Filter")
adxMax   = input.int(25,  "Max ADX (Range)",group="Range Filter")

// ── Indicators
[bbMid, bbUpper, bbLower] = ta.bb(close, bbLen, bbMult)
rsi = ta.rsi(close, rsiLen)

// ADX calculation
[plusDI, minusDI, adx] = ta.dmi(adxLen, adxLen)

// ── Range condition: low ADX = choppy/ranging market
isRanging = adx < adxMax

// ── Entry conditions
buyCondition  = close <= bbLower and rsi < rsiOS  and isRanging
sellCondition = close >= bbUpper and rsi > rsiOB  and isRanging

// ── Orders: exit at the midline
if buyCondition and strategy.position_size == 0
    strategy.entry("Long", strategy.long)
    strategy.exit("Long X", "Long",
                  stop  = bbLower - (bbMid - bbLower) * 0.5,
                  limit = bbMid)

if sellCondition and strategy.position_size == 0
    strategy.entry("Short", strategy.short)
    strategy.exit("Short X", "Short",
                  stop  = bbUpper + (bbUpper - bbMid) * 0.5,
                  limit = bbMid)

// ── Plots
plot(bbUpper, "BB Upper", color.red,   linewidth=1)
plot(bbMid,   "BB Mid",   color.gray,  linewidth=1, style=plot.style_dashed)
plot(bbLower, "BB Lower", color.green, linewidth=1)

bgcolor(isRanging ? color.new(color.blue, 95) : na, title="Ranging Market")
```

---

### script46 — London Session Breakout Strategy

> Forex-specific strategy that captures the Asian session range, then trades breakouts of that range during the first two hours of the London open. One trade per day maximum with EOD exit.

```pine
//@version=5
strategy("London Breakout", overlay=true,
         initial_capital=10000, default_qty_type=strategy.percent_of_equity,
         default_qty_value=5, commission_value=0.07, slippage=3)

// ── Inputs
asiaStartHour = input.int(0,  "Asia Session Start (UTC)", minval=0, maxval=23)
asiaEndHour   = input.int(8,  "Asia Session End (UTC)",   minval=0, maxval=23)
londonStart   = input.int(8,  "London Open (UTC)",        minval=0, maxval=23)
londonEnd     = input.int(10, "Trade Until (UTC)",        minval=0, maxval=23)
eodHour       = input.int(20, "EOD Close Hour (UTC)",     minval=0, maxval=23)
slBuffer      = input.float(0.0002, "SL Buffer (price)",  step=0.0001)

// ── Session detection (uses UTC time)
isAsiaSession   = (hour(time, "UTC") >= asiaStartHour) and (hour(time, "UTC") < asiaEndHour)
isLondonEntry   = (hour(time, "UTC") >= londonStart)   and (hour(time, "UTC") < londonEnd)
isEOD           = hour(time, "UTC") == eodHour and minute(time, "UTC") == 0

// ── Track Asia range
var float asiaHigh = na
var float asiaLow  = na
var bool  asiaSet  = false
var bool  tradeDay = false

isNewDay = dayofweek != dayofweek[1]

if isNewDay
    asiaHigh := na
    asiaLow  := na
    asiaSet  := false
    tradeDay := true

if isAsiaSession
    asiaHigh := na(asiaHigh) ? high : math.max(asiaHigh, high)
    asiaLow  := na(asiaLow)  ? low  : math.min(asiaLow,  low)
    asiaSet  := true

// ── Breakout conditions
breakoutUp   = isLondonEntry and asiaSet and close > asiaHigh and
               strategy.position_size == 0 and tradeDay
breakoutDown = isLondonEntry and asiaSet and close < asiaLow  and
               strategy.position_size == 0 and tradeDay

if breakoutUp
    strategy.entry("Long",  strategy.long)
    strategy.exit("Long X", "Long",
                  stop  = asiaLow  - slBuffer,
                  limit = asiaHigh + (asiaHigh - asiaLow))
    tradeDay := false  // One trade per day

if breakoutDown
    strategy.entry("Short", strategy.short)
    strategy.exit("Short X", "Short",
                  stop  = asiaHigh + slBuffer,
                  limit = asiaLow  - (asiaHigh - asiaLow))
    tradeDay := false

// EOD: close all
if isEOD and strategy.position_size != 0
    strategy.close_all("EOD")

// ── Plots
plot(asiaHigh, "Asia High", color.new(color.red,   50), style=plot.style_circles)
plot(asiaLow,  "Asia Low",  color.new(color.green, 50), style=plot.style_circles)
bgcolor(isAsiaSession   ? color.new(color.blue,   95) : na, title="Asia Session")
bgcolor(isLondonEntry   ? color.new(color.orange, 92) : na, title="London Window")
```

---

### script47 — MACD + 200 EMA Trend Following Strategy

> Enters only in the direction of the 200 EMA trend (rising above = long only, falling below = short only). Uses MACD histogram zero-line crossovers as the entry trigger. 4:1 reward-to-risk target.

```pine
//@version=5
strategy("MACD + 200 EMA Trend", overlay=true,
         initial_capital=10000, default_qty_type=strategy.percent_of_equity,
         default_qty_value=5, commission_value=0.07, slippage=2)

// ── Inputs
ema200Len = input.int(200, "Trend EMA",      group="Trend Filter")
macdFast  = input.int(12,  "MACD Fast",      group="MACD")
macdSlow  = input.int(26,  "MACD Slow",      group="MACD")
macdSig   = input.int(9,   "MACD Signal",    group="MACD")
slMult    = input.float(2.0,"SL ATR Mult",   group="Exit")
tpMult    = input.float(4.0,"TP ATR Mult",   group="Exit")

// ── Indicators
ema200 = ta.ema(close, ema200Len)
[macdLine, signalLine, hist] = ta.macd(close, macdFast, macdSlow, macdSig)
atr    = ta.atr(14)

// ── Trend state
aboveTrend = close > ema200 and ema200 > ema200[5]  // Price above & EMA rising
belowTrend = close < ema200 and ema200 < ema200[5]  // Price below & EMA falling

// ── Entries: MACD crosses zero in direction of trend
longEntry  = ta.crossover(hist, 0)  and aboveTrend
shortEntry = ta.crossunder(hist, 0) and belowTrend

// ── Orders
if longEntry
    strategy.entry("Long", strategy.long)
    strategy.exit("Long X", "Long",
                  stop  = close - atr * slMult,
                  limit = close + atr * tpMult)

if shortEntry
    strategy.entry("Short", strategy.short)
    strategy.exit("Short X", "Short",
                  stop  = close + atr * slMult,
                  limit = close - atr * tpMult)

// ── Plots
plot(ema200, "200 EMA", color.orange, linewidth=3)
plotshape(longEntry,  style=shape.triangleup,   location=location.belowbar, color=color.lime)
plotshape(shortEntry, style=shape.triangledown, location=location.abovebar, color=color.red)

bgcolor(aboveTrend ? color.new(color.green, 93) :
        belowTrend ? color.new(color.red,   93) : na)
```

---

### script48 — Multi-Confluence Swing Trading Strategy

> The most complete strategy in the book. Requires five independent conditions to align before entering: 50/200 EMA trend stack, RSI pullback zone, bar close confirmation, minimum ATR volatility. Includes trailing stop, configurable risk parameters, and alerts.

```pine
//@version=5
strategy("Multi-Confluence Swing", overlay=true,
         initial_capital=10000, default_qty_type=strategy.percent_of_equity,
         default_qty_value=5, commission_value=0.07, slippage=2,
         calc_on_order_fills=false)

// ── Inputs
ema50Len  = input.int(50,   "EMA 50",          group="Trend")
ema200Len = input.int(200,  "EMA 200",         group="Trend")
rsiLen    = input.int(14,   "RSI Length",      group="Momentum")
rsiLongLo = input.int(40,   "RSI Long Min",    group="Momentum")
rsiLongHi = input.int(58,   "RSI Long Max",    group="Momentum")
rsiShtLo  = input.int(42,   "RSI Short Min",   group="Momentum")
rsiShtHi  = input.int(60,   "RSI Short Max",   group="Momentum")
atrLen    = input.int(14,   "ATR Length",      group="Volatility")
minAtr    = input.float(0.0005,"Min ATR",  step=0.0001, group="Volatility")
slMult    = input.float(1.5,"SL ATR Mult",     group="Exit")
tpMult    = input.float(3.0,"TP ATR Mult",     group="Exit")
trailAtr  = input.float(2.0,"Trail ATR Mult",  group="Exit")
useShort  = input.bool(false,"Trade Shorts",   group="Exit")

// ── Indicators
ema50  = ta.ema(close, ema50Len)
ema200 = ta.ema(close, ema200Len)
rsi    = ta.rsi(close, rsiLen)
atr    = ta.atr(atrLen)

// ── Trend alignment
bullTrend = close > ema50 and close > ema200 and ema50 > ema200
bearTrend = close < ema50 and close < ema200 and ema50 < ema200

// ── Entry triggers (bar-by-bar)
rsiBullZone  = rsi >= rsiLongLo and rsi <= rsiLongHi
rsiBearZone  = rsi >= rsiShtLo  and rsi <= rsiShtHi
priceConfLong  = close > high[1]     // Close above prior bar's high
priceConfShort = close < low[1]      // Close below prior bar's low
volatilityOK   = atr > minAtr        // Sufficient volatility

longCondition  = bullTrend and rsiBullZone  and priceConfLong  and volatilityOK
shortCondition = bearTrend and rsiBearZone  and priceConfShort and volatilityOK and useShort

// ── Orders
if longCondition and strategy.position_size <= 0
    strategy.entry("Long", strategy.long)
    strategy.exit("Long X", "Long",
                  stop         = close - atr * slMult,
                  limit        = close + atr * tpMult,
                  trail_offset = atr * trailAtr * syminfo.mintick * 10)

if shortCondition and strategy.position_size >= 0
    strategy.entry("Short", strategy.short)
    strategy.exit("Short X", "Short",
                  stop         = close + atr * slMult,
                  limit        = close - atr * tpMult,
                  trail_offset = atr * trailAtr * syminfo.mintick * 10)

// ── Plots
plot(ema50,  "EMA 50",  color.blue,   linewidth=2)
plot(ema200, "EMA 200", color.orange, linewidth=3)

plotshape(longCondition,  style=shape.triangleup,   location=location.belowbar,
          color=color.lime, size=size.normal, text="BUY")
plotshape(shortCondition, style=shape.triangledown, location=location.abovebar,
          color=color.red,  size=size.normal, text="SELL")

bgcolor(bullTrend ? color.new(color.green, 93) :
        bearTrend ? color.new(color.red,   93) : na)

// ── Alerts
alertcondition(longCondition,  "Long Setup",  "Multi-Confluence Long on {{ticker}} | {{interval}}")
alertcondition(shortCondition, "Short Setup", "Multi-Confluence Short on {{ticker}} | {{interval}}")
```

---

## Quick Stats

| | Count |
|---|---|
| Chapter examples (scripts 1–38) | 38 |
| Sample indicators (scripts 39–43) | 5 |
| Sample strategies (scripts 44–48) | 5 |
| **Total scripts** | **48** |

---

> **Book:** *Pine Script: From Zero to Indicator*
> **Language:** Pine Script v5
> **Platform:** [TradingView](https://www.tradingview.com)
> **Disclaimer:** All scripts are for educational purposes only. Past performance does not guarantee future results. Always test on a demo account before trading with real capital.
