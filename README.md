# Anchored VWAP Alert — MQL4 Script

A MetaTrader 4 script that computes an **Anchored Volume Weighted Average Price (AVWAP)** by accumulating `typicalPrice × volume` and total volume exclusively from a user-defined `AnchorPoint` datetime forward via `iTime()` time comparison, fires alerts when the current close deviates from the anchored VWAP by more than the configurable `AlertBuffer` price distance in either direction — providing a precision fair-value reference anchored to any significant market event: swing highs/lows, news events, earnings, or session opens.

---

## Overview

The Anchored VWAP, popularized by Paul Levine and later adopted widely by institutional and retail traders through the work of Brian Shannon (*Anchored VWAP*), extends the standard VWAP concept by allowing the accumulation window to begin at any arbitrary point in time rather than resetting at session open. This flexibility is its defining advantage: by anchoring to a significant market event — the low of a major selloff, the open of a breakout candle, the date of an earnings release — the AVWAP measures the volume-weighted fair value for all participants who entered the market from that event forward. If price is trading above the AVWAP, the average participant who entered since the anchor is in profit; below it, they are underwater. This supply/demand balance creates dynamic support and resistance at the AVWAP level with institutional significance that a time-based moving average cannot replicate. This script allows the trader to set the anchor via the `AnchorPoint` datetime input and continuously monitors the AVWAP deviation in real time.

---

## Features

- **`AnchorPoint` datetime-gated accumulation** — `CalculateAnchoredVWAP()` iterates all `iBars()` bars from most recent to oldest; `if (time < anchorPoint) break` stops accumulation at the anchor; only bars after the anchor contribute to `totalVolumePrice` and `totalVolume`
- **Typical price VWAP formula** — `typicalPrice = (high + low + close) / 3.0`; `totalVolumePrice += typicalPrice × volume`; `anchoredVWAP = totalVolumePrice / totalVolume` — standard VWAP formula applied to the anchor-gated window
- **`totalVolume > 0.0` division guard** — `CalculateAnchoredVWAP()` returns `false` if no volume accumulated (anchor point in the future or no data after anchor); main loop skips alert evaluation on `false` return
- **`AlertBuffer` fixed-distance deviation gate** — `currentPrice > anchoredVWAP + AlertBuffer` → **Price Above Anchored VWAP**; `currentPrice < anchoredVWAP − AlertBuffer` → **Price Below Anchored VWAP** — fixed price-unit buffer rather than percentage, appropriate for instruments where absolute distance is more meaningful than relative percentage
- **Rich alert message** — `AlertVWAP()` formats with both `anchoredVWAP` and `currentPrice` via `"Anchored VWAP: %.5f, Current Price: %.5f"` for immediate reference
- **Default `PERIOD_M15` timeframe** — balances computational load with AVWAP resolution for intraday anchoring; configurable to any timeframe
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)

---

## How It Works

1. Every minute, `CalculateAnchoredVWAP()` iterates from most-recent bar backward, accumulating `typicalPrice × volume` until `iTime(i) < AnchorPoint`; computes `anchoredVWAP = totalVolumePrice / totalVolume`
2. `currentPrice = iClose(..., 0)` fetched; deviation conditions evaluated:
   - `currentPrice > anchoredVWAP + AlertBuffer` → **Price Above Anchored VWAP**
   - `currentPrice < anchoredVWAP − AlertBuffer` → **Price Below Anchored VWAP**
3. Alert dispatched via all enabled channels with both AVWAP and current price

---

## Input Parameters

| Parameter        | Type            | Default                  | Description                                                          |
|------------------|-----------------|--------------------------|----------------------------------------------------------------------|
| `TradeSymbol`    | string          | `EURUSD`                 | Symbol for analysis                                                  |
| `Timeframe`      | ENUM_TIMEFRAMES | `PERIOD_M15`             | Timeframe for AVWAP computation                                      |
| `AnchorPoint`    | datetime        | `D'2024.11.01 00:00'`    | Datetime from which VWAP accumulation begins                         |
| `AlertBuffer`    | double          | `0.0050`                 | Fixed price distance from AVWAP required to trigger an alert         |
| `EnableAlerts`   | bool            | `true`                   | Fire an on-screen/sound alert                                        |
| `EnableEmail`    | bool            | `false`                  | Send an email notification                                           |
| `EnablePush`     | bool            | `false`                  | Send a mobile push notification                                      |

---

## Alert Message Format

```
Price Above Anchored VWAP detected on EURUSD (Timeframe: PERIOD_M15)
Anchored VWAP: 1.08290, Current Price: 1.08850
```

---

## Installation

1. Copy `VWAP_Anchored_001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Set `AnchorPoint` to your desired reference datetime (e.g. a major swing low, session open, or news event)
4. Drag onto any chart from Navigator → Scripts; configure inputs and click **OK**

> **Note:** `AnchorPoint` must be a datetime in the past with available bar history. Setting it to a future date or a date before the terminal's available history will result in zero accumulation and no alerts.

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
