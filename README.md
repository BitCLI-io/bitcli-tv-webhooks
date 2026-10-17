# ⚡ Hosted TradingView to Telegram Webhook Gateway | BitCLI
### Zero-Setup • No VPS • No Coding Required • Cloud Managed

[![No Server Required](https://img.shields.io/badge/Server-Zero_Setup_(No_VPS)-10b981?style=for-the-badge)](#)
[![TradingView](https://img.shields.io/badge/TradingView-PineScript_v5-2962FF?style=for-the-badge&logo=tradingview)](https://www.tradingview.com/)
[![Telegram](https://img.shields.io/badge/Telegram-Instant_Relay-2CA5E0?style=for-the-badge&logo=telegram)](https://t.me/bitclibot)

> 🛑 **Stop wasting time hosting your own Flask/Node.js servers, paying for VPS, and configuring ngrok.**  
> BitCLI provides an institutional-grade, **cloud-hosted webhook relay** that routes your Pine Script alerts directly into Telegram private chats, VIP channels, or groups in **<15ms**.

### 💡 Why BitCLI instead of self-hosted scripts?
- ❌ **Old Way:** Rent a Linux VPS ($60/yr), configure Docker, renew SSL certificates, handle bot crashes.
- ✅ **BitCLI Way:** Open [@bitclibot](https://t.me/bitclibot), generate your secret URL in 1 tap, paste into TradingView. Zero maintenance.

Ultra-low latency bridge between TradingView Pine Script alerts and Telegram (Private chats, VIP channels, and Groups).

Delivered in **<15ms** via Nginx Lua edge caching, Redis Streams, and a high-throughput async processing pipeline.

---

## 🚀 Quick Setup (30 Seconds)

1. Open [@bitclibot](https://t.me/bitclibot) in Telegram and run:
   ```text
   /webhooks
   ```
2. Click **➕ Create Webhook**, choose your tag name and target chat (*Private*, *Channel*, or *Group*).
3. Copy your unique endpoint URL:
   ```http
   https://wh.bitcli.io/v1/YOUR-SECRET-UUID
   ```
4. In TradingView, create any Alert (`Alt + A`), check the **Webhook URL** box, and paste your URL.

---

## 📦 Payload Templates

BitCLI supports two rendering modes: **Standard Structured JSON** (auto-rendered into a compact terminal card) and **Custom Jinja2 HTML Templates**.

### Mode 1: Standard Structured Card (Recommended)
Paste this into the **Message** box of your TradingView alert:

```json
{
  "coin": "{{ticker}}",
  "action": "BUY",
  "price": {{close}},
  "leverage": "20x",
  "comment": "RSI Oversold Bounce + EMA Cross"
}
```

#### 📲 How it appears in Telegram:
> 🚨 **SIGNAL: BTCUSDT**  
> 🎯 Action: **BUY** (20x)  
> 💵 Price: `95420.50`  
> 📝 *RSI Oversold Bounce + EMA Cross*

---

### Mode 2: Custom Jinja2 Template (Full HTML Control)
If you want complete control over formatting, icons, and dynamic PineScript plots, pass the payload inside a `"message"` property:

```json
{
  "message": "🔥 <b>VIP SCALP SIGNAL</b>\n\nAsset: <code>{{ticker}}</code>\nAction: <b>STRONG {{strategy.order.action}}</b>\nEntry: <code>{{close}}</code>\nSL: <code>{{plot_0}}</code>\nTP: <code>{{plot_1}}</code>\n\n⚠️ <i>Risk: 1.5% max capital</i>"
}
```

> **Allowed HTML Tags:** `<b>`, `<i>`, `<code>`, `<s>`, `<u>`, `<pre>`, `<a>`, `<tg-spoiler>`, `<blockquote>`.  
> *All unclosed tags or syntax malformations are automatically sanitized on the fly by BitCLI.*

---

## 📈 Ready-to-Use Pine Script v5 Examples

### Example 1: RSI Divergence & Level Bounce Indicator
Attach this script to any chart to fire formatted webhook alerts on RSI extremes:

```pinescript
//@version=5
indicator("BitCLI - RSI Extreme Alerts", overlay=true)

rsiLength = input.int(14, "RSI Length")
oversold  = input.int(30, "Oversold Level")
overbought= input.int(70, "Overbought Level")

rsiVal = ta.rsi(close, rsiLength)

longCondition  = ta.crossover(rsiVal, oversold)
shortCondition = ta.crossunder(rsiVal, overbought)

plotshape(longCondition, title="Buy Signal", location=location.belowbar, color=color.green, style=shape.triangleup, size=size.small)
plotshape(shortCondition, title="Sell Signal", location=location.abovebar, color=color.red, style=shape.triangledown, size=size.small)

if longCondition
    alert('{"coin":"' + syminfo.ticker + '","action":"LONG","price":' + str.tostring(close) + ',"comment":"RSI Bullish Rebound"}', alert.freq_once_per_bar_close)

if shortCondition
    alert('{"coin":"' + syminfo.ticker + '","action":"SHORT","price":' + str.tostring(close) + ',"comment":"RSI Bearish Exhaustion"}', alert.freq_once_per_bar_close)
```

---

### Example 2: Strategy Automated Execution Alert
If you are running a backtested `strategy()`, use dynamic placeholder replacements:

```pinescript
//@version=5
strategy("BitCLI - Supertrend Strategy", overlay=true, margin_long=100, margin_short=100)

atrPeriod = input.int(10, "ATR Length")
factor    = input.float(3.0, "Factor")

[supertrend, direction] = ta.supertrend(factor, atrPeriod)

if ta.change(direction) < 0
    strategy.entry("Long", strategy.long)
    alert('{"coin":"' + syminfo.ticker + '","action":"BUY / LONG","price":' + str.tostring(close) + ',"comment":"Supertrend Bullish Flip"}', alert.freq_once_per_bar)

if ta.change(direction) > 0
    strategy.entry("Short", strategy.short)
    alert('{"coin":"' + syminfo.ticker + '","action":"SELL / SHORT","price":' + str.tostring(close) + ',"comment":"Supertrend Bearish Flip"}', alert.freq_once_per_bar)
```

---

## 🛡️ Security & Zero-Spam Architecture

* **Official TradingView IP Whitelisting:** Toggle `🛡 TV IP Filter` inside `/webhooks` to silently drop any request originating outside TradingView's official server clusters:
  * `52.89.214.238`
  * `34.212.75.30`
  * `54.218.53.128`
  * `52.32.178.7`
* **Zero Rate-Limit Lockout:** Webhook signals destined for high-throughput channels are dynamically bucketed and dispatched via private Redis Streams to avoid Telegram API rate limits (30 msg/sec).
* **Instant URL Rotation:** If your secret URL is ever exposed publicly, regenerate it in one tap via the bot interface without re-creating settings.

---

## 💡 Troubleshooting & FAQ

| Problem | Cause | Solution |
| :--- | :--- | :--- |
| **HTTP 401 Unauthorized** | The UUID token is wrong, paused, or was rotated. | Check active URLs inside `/webhooks` in Telegram. |
| **HTTP 403 Forbidden** | Request originated from a non-TradingView IP while the TV filter was enabled. | Disable `TV IP Filter` in `/webhooks` if testing from Postman/cURL. |
| **HTTP 429 Limit Exceeded** | Daily webhook quota exhausted. | Upgrade your quota in `/tariffs` or wait until 00:00 UTC. |
| **Telegram Delivery Error** | Bot was removed from the destination Channel/Group. | Re-add [@bitclibot](https://t.me/bitclibot) as Admin with "Post Messages" permission. |

---

<div align="center">
  <b>Built for quant traders by BitCLI</b><br>
  Docs: <a href="https://docs.bitcli.io/webhooks">docs.bitcli.io/webhooks</a> | Support: <a href="mailto:support@bitcli.io">support@bitcli.io</a>
</div>
```
