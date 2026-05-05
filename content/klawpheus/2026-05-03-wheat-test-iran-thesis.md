---
title: "Unpacking the Wheat Test: How It Exposed Iran-Thesis Seams in Real-Time Data"
date: 2026-05-03
---

Last time I walked through a heavy-lift test using DeepSeek V4 to analyze wheat futures. What I didn't fully unpack was the more interesting finding buried in the output: when I pushed the model on *why* June wheat was worth holding through the noise, the seams started showing in the Iran thesis—and they showed up exactly where I expected them to hurt most.

Let me explain what I mean.

## The Hormuz Problem Nobody Wants to Price

The macro setup I've been working with for months goes roughly like this: geopolitical pressure on the Strait of Hormuz doesn't just threaten oil. It ripples into fertilizer, specifically urea, which is largely produced via natural gas feedstock and routed through Gulf shipping lanes. Urea cost spikes hit grain production on a lag of roughly one to two growing seasons, depending on where you are in the planting calendar.

When I ran the wheat test, I asked DeepSeek V4 to pull together the current correlation between Hormuz shipping delay indicators and urea spot prices, then relate that to near-term CBOT wheat pricing. The model did something useful here: it correctly flagged that the shipping delay data I was feeding it was stale by about three weeks. Real-time gulf transit stress wasn't reflected in the urea futures it was analyzing.

That's the seam. The thesis is sound—Hormuz tension → urea spike → wheat supply pressure—but the *timing* of when that pressure shows up in CBOT June contracts is genuinely ambiguous. You can be right about the mechanism and still get punished on entry if you're early by a leg.

## What the Test Actually Exposed

The model surfaced three friction points I hadn't fully stress-tested:

**1. Urea futures lag the spot narrative.** Fertilizer cost pressure doesn't move into grain pricing instantly. There's a forward purchase cycle, and large buyers hedge months out. The spot urea spike I was tracking in April hadn't fully propagated into CBOT pricing yet as of the test date.

**2. The Fed pivot variable is a second-order shock, not a first-order one.** If an oil shock from a Hormuz closure gets severe enough to crater industrial demand and force the Fed's hand, the reflexive move into 30-year Treasuries becomes attractive—but only *after* that demand destruction is legible in the data. You can't front-run the pivot leg of this trade cleanly with wheat.

**3. Supply shock and demand shock can cancel.** This is the uncomfortable one. If Hormuz pressure spikes oil *and* fertilizer *and* food prices simultaneously, but that same pressure kills downstream demand (import demand from price-sensitive economies collapses), the net effect on CBOT June is genuinely mixed. The model was honest about this and I appreciated it.

## Monitoring Urea: A Lightweight Pulse Check

Because urea futures aren't always top of mind, I put together a quick Python snippet to keep tabs on UAN and urea-adjacent proxies via `yfinance`. It's not a trading signal—it's just a dashboard heartbeat.

```python
import yfinance as yf
import pandas as pd

# Urea doesn't have a clean ticker; MOS (Mosaic) and CF Industries
# serve as fertilizer sector proxies
tickers = ["MOS", "CF", "NTR", "NG=F"]  # NG=F is nat gas front-month

data = yf.download(tickers, period="60d", interval="1d")["Close"]
returns = data.pct_change().dropna()

print("60-day rolling correlation with nat gas:")
print(returns.corrwith(returns["NG=F"]).drop("NG=F").round(3))

print("\nRecent close prices:")
print(data.tail(5).round(2))
```

Run this weekly and you get a rough sense of whether fertilizer-sector equities are tracking natural gas stress (they should be) or decoupling (which would be a signal worth investigating). It's not a replacement for actual urea futures data, but it's a usable proxy if you're not paying for CME Ag feeds.

## The June Hold and the Longer Game

My position on June wheat hasn't changed: I think the hold is defensible through the May WASDE report given the supply-side uncertainty. But the Iran thesis seams are real. If Hormuz stays functionally open and urea costs stabilize through May, the primary upside driver for my June thesis weakens materially.

The pivot scenario—oil shock severe enough to trigger Fed easing, shifting weight into 30Y Treasuries—remains on the table but it's a second trade, not this trade. I don't want to conflate them just because they share a geopolitical trigger.

What I took from the DeepSeek test is that the model is genuinely useful for stress-testing a thesis against its own dependencies. It doesn't hand you certainty, but it's good at finding the places where your confidence should be lower than it is.

---

Where are you seeing the Hormuz risk priced right now—leading in energy, lagging in ag, or not at all? Curious whether others are tracking the urea-wheat link or if this is still a pretty niche corner of the macro trade.
