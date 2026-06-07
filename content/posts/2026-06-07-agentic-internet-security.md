---
title: "AI Agents are your new threat model"
date: 2026-06-07
---

## The threat to your data in the age of AI.
![computing](/assets/posts/agentic-internet-security/secure-computing.jpg)

**TL;DR:** If you are not a spy, criminal, or journalist, the steps in the Defensive Measures section are enough to protect your data in the age of agents.

As the web develops, the personal protections a user should be prudent to take keep shifting. They have to evolve with the technology, and the most recent shift is agents. Which leaves the question this post tries to frame and answer: what is accessing my data, how do I limit my exposure, and what protections are practically available for the motivated netizen?

The internet was initially designed to be open, public, and free. It is arguably still public and perhaps more arguably free, but time has shown that protections must be in place to limit access to sensitive data and to guard the personal information held by any system connected to the internet. Much of that landscape is challenged by mismanagement, lack of resources, and law-enforcement requirements. Often, as users of a service, we are required to hand over personal information and trust that the service will be a good steward of it. Most examples of this trust have proven misplaced — the companies that demand our information mishandle it, sell it, and, possibly worst of all, use it against us.

All of this is known. But consider something new. What if we aren't protecting our data from humans anymore? What if the primary concern isn't our information being leaked, consumed, and sold by companies, but by systems — agentic systems? What does that mean, and what are the implications?

### LLMs

#### Anthropic

The most obvious example of users handing information to a system is the LLM. Using ChatGPT, Claude, or any other hosted chat agent feeds it your personal information. Think about what OpenAI or Anthropic could assemble about a regular user: personal questions, travel advice, recipes, health questions, relationship advice, resume building, large-purchase research. Any of those conversations can reveal — or leak — a great deal about you. The more conversations, the more leakage, the more complete the profile.

Another example is email. For all the Gmail users out there: have you opted out of the Google agent services that comb through your email history to "help" you summarize a thread or suggest draft language? Are these all benign helper robots trying to make your day easier?

The primary mitigation is that this is voluntary. The user willingly engages with the service and can moderate what gets relegated to the LLM. But that's a slippery slope, because of how easy it is to innocuously pass private information into a conversation. This isn't limited to passwords — which usually trigger an explicit warning — but to subtler things: *tell me a wedding joke*, *what should I buy an eight-year-old*. Revealing, all of it.

Second, and arguably more important, are the privacy policies. There are acknowledged limits on what a company can do with this information, and the privacy policy of any service you engage with is well worth the time to read. Focusing on the LLM services lets us define the battlezone.

Anthropic's policy has clear language about what is retained — here, chat conversations — and what it can be used for. The retention period is the thing to understand, and there is a real distinction between the **consumer** and **commercial** tiers.

Under the consumer tier (Free, Pro, Max), users can choose to allow their chat data to be used for training to help improve the models. If you allow this, your data may be retained in **de-identified** form for up to **5 years**. If you opt out, your data stays on the system only as long as *you* keep it there — it remains in your control. When you delete a conversation it leaves your history immediately and is purged from the back end within **30 days**. So the headline contrast is: opt in and a de-identified copy lives in the training pipeline for up to five years; opt out and you control retention, with a 30-day deletion lag.

Commercial licenses work differently. The **API** standard tier auto-deletes inputs and outputs within 30 days and is never used for training. The **Team and Enterprise** tier is also never used for training — there is no opt-in toggle at all — and deleted material carries the same 30-day window. Eligible enterprise API customers can additionally arrange **Zero Data Retention** (subject to Anthropic approval), under which inputs and outputs aren't stored beyond what's needed to screen for abuse. Note the pattern: the privacy tier you land in tracks the *contract*, not the price. An API developer paying per token has stronger default privacy than a Pro subscriber paying monthly.

**The trust-and-safety system.** Here is the rub. The safety classifiers run on *everything*, across every tier — there is no opt-out, and you can't buy your way around it (even Zero Data Retention still keeps the safety classifier results). The classification is universal; the *retention* is conditional. If your content trips a Usage Policy flag, the inputs and outputs are kept for **2 years** and the classification scores for **7 years**. And like any law-abiding business, Anthropic will comply with valid legal process and hand over user data when required to do so.

So the case study leaves the reader with a transferable habit: when you read any AI privacy policy, ask *which layer is this — the one I control, or the one I don't?*

#### Google

Google, this year, turned on Skynet. Well — they turned on Google, and aimed it at your Google data, which is apparently *their* data; you just rent it. In 2026 Google flipped Gemini **on by default** across its services. That means Google's agentic systems get access to everything on their systems: Gmail, Drive, photos, videos, attachments, documents, spreadsheets — all of it. *Don't be evil.* (Worth noting: this default-on behavior is the US setting. In the EU, UK, Japan, and Switzerland it ships **off** by default — which tells you whether "private by default" is a technical limit or a policy choice.)

And in not being evil, Google deployed Gemini Nano, a ~4GB on-device model that downloads to your machine for "Chrome enhancement." It's an *Optimization Guide On Device Model* feature that doesn't ask. Chrome silently writes the model file (`weights.bin`) into your profile the moment it decides your hardware qualifies:

> "It does not ask for consent, and sends no notification."
>
> — [Malwarebytes](https://www.malwarebytes.com/blog/news/2026/05/google-chromes-silent-4gb-ai-download-problem)

Local is private, right? *Right?* Unfortunately, the "AI Mode" pill in Chrome's address bar routes every query to Google's servers anyway — the on-device model mostly powers buried features like "Help me write" that few people use. So you pay the storage and bandwidth for a local model while the headline AI feature still phones home. The local-looking surface is doing persuasive work the architecture doesn't back up. Disable it: `chrome://flags` → search "optimization guide on device" → disable; verify at `chrome://on-device-internals`.

### Defensive Measures

There is no silver bullet for LLMs. We invite them in, and unchecked they can leak more about us than we'd ever expect. The list below splits two ways: a few items just shrink the blast radius for *any* adversary, and a few are aimed squarely at the agent problem. If you're a sophisticated security user, or you need sophisticated solutions, this isn't your list.

- **Know your privacy policy** — read it through the "which layer do I control" lens; the retention you opt out of today is the data an agent can't be trained on tomorrow.
- **Use private LLMs** — local models or non-retaining providers; the triage agent you actually *want* shouldn't have to ship your inbox to someone else's servers to do its job.
- **Use email aliases** — not just anti-spam; this is compartmentalization, so no single agent — yours or theirs — ever assembles a complete profile from one address.
- **Use private services when possible** — every service you don't feed is one whose agents can't read, summarize, or train on you.
- **Passkeys first** — they bind your credential to the real domain, so an agent serving you a convincing fake login page has nothing to harvest. (Pair every passkey with a recovery path.)
- **Hardware keys next** — the physical backup that makes passkeys safe; a key in your pocket is the one factor no remote agent can phish, scrape, or replay.
- **Password managers third** — unique credentials per site cap the blast radius: an agent that breaches one service walks away with one password, not the keys to your whole life.

**Caveats (a little friction, worth it)**

- **Use Advanced Data Protection** — Apple's opt-in end-to-end encryption for iCloud; it puts your photos, backups, and notes beyond the reach of any cloud agent, Apple's own included. (Better than blanket-disabling iCloud backup, which mostly just risks data loss.)
- **Disable notification previews on iOS** — closes a small over-the-shoulder leak on your lock screen. Not an agent defense, just cheap and sensible. (Don't hate me.)
- **Encrypted vaults** — the one folder an agent can't index even if it reaches your disk; encrypt what matters, don't re-architect your life.

**Services (not endorsements)**

- **Email:** Proton Mail
- **Passwords:**
  - **PearPass** ([pass.pears.com](https://pass.pears.com/)) — open-source, local-first, P2P sync, no cloud, independently audited; by Tether Data. (The browser extension needs the desktop app running — a consequence of the local-only design.)
  - **KeePassXC** — open-source, fully offline, you own the vault file
  - **Proton Pass** — if you want a cloud option from a privacy-aligned provider
  - **Bitwarden** — open-source and audited, but proceed with eyes open: it's PE-backed, and a February 2026 leadership change to an M&A-focused CEO has fueled speculation it's being prepped for resale. Worth watching.
- **Messaging:** Any E2EE app. **Signal** is the gold standard. **iMessage** is E2EE *only Apple-to-Apple* — green-bubble SMS falls back to plaintext. Avoid **Telegram** (not E2EE by default).
- **LLMs:** Run your own. Otherwise [Venice.ai](https://venice.ai/)
- **Phone:** Google Voice (number tiering)
- **Encrypted vaults:** VeraCrypt

---

If you *are* a spy, thief, or journalist, you could research the following.
*Just prefix everything with "secure."*

- **Operating systems:**
  - **Tails** — amnesic OS booted from USB, routes everything through Tor, leaves nothing behind. The canonical journalist tool.
  - **Whonix** — Tor-gateway VM that pairs directly with Qubes (Qubes-Whonix is the standard combo)
  - **Qubes-OS** — compartmentalization by virtualization
- **Files & metadata:**
  - **MAT2** / **ExifTool** — strip EXIF and document metadata *before* sharing. A leaked PDF's author field or a photo's GPS tag has burned more sources than weak crypto ever has.
- **Communications:**
  - **GnuPG (PGP)** — end-to-end mail encryption that doesn't trust the provider
  - **SecureDrop** / **OnionShare** — anonymous tip submission and Tor-based file sharing; SecureDrop is what newsrooms actually run
- **Tunnels:** Tor; self-hosted VPN, or [Mullvad](https://mullvad.net/en)
- **Payments:** **Monero (XMR)** — where Bitcoin's public ledger would deanonymize you
- **Phone:** **GrapheneOS**; plus **MySudo** or **JMP.chat** for compartmentalized numbers

A closing note for this tier: none of these tools work without OPSEC. Tools give you capability; discipline gives you privacy. A hardened phone that you log into your real Google account from is just an expensive normal phone.