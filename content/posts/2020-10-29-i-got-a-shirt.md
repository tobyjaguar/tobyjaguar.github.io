---
title: "I Got A Shirt"
date: 2020-10-29
---

## Own The Internet’ Hackathon: Post Mortem
![shirt](/assets/posts/i-got-a-shirt/_hns-hackathon-tee.jpeg)

Following the Sia and Namebase hackathon via
<a href="https://gitcoin.co/" target="_blank" rel="noopener noreferrer">gitcoin</a> 
I recieved a runner up prize of 250 HNS and this pretty fun shirt, but
<a href="https://github.com/tobyjaguar/handybot" target="_blank" rel="noopener noreferrer">handybot</a> also got used in production for a bit in the 
<a href="https://t.me/handshaketalk" target="_blank" rel="noopener noreferrer">HandshakeTalk</a> 
telegram chat group, which was great to see.

The following is a recap of the use of the bot and some lessons for using telegram bots in a production environment.

### Handybot in production

I posted to twitter that I had made handybot for the hackathon, and
the moderator for the HandshakeTalk telegram channel asked if 
he could add it to the group. I was intrigued to see how the bot
handled a pretty active channel, and I was also a bit eager for the
opportunity because that was the exact venue handbybot was designed for.

The bot failed amazingly! The first challenge was standing it up on 
a server, which I had done for the hackathon, and also running it against a db to query the HSN price. I wanted the bot to also give
the HNS/USD crossrate, but I didn't want to just forward api calls
to the <a href="https://www.namebase.io/" target="_blank" rel="noopener noreferrer">namebase</a> api, as it would blow out the api call quota. So for some interval, handybot would write the HNS price to mongoDB and then any user
requesting the price from the bot would result in a db call, rather than an api call.

All of this was working fine, and handybot was added to the telegram 
channel. It worked for a bit. Success!

### Handybot falls down

Someone was able to take down the bot by feeding it null values, which was a greate test case. The bot was hardened against this, but ultimately the bot was sending garbage responses to the chat channel in certain cases, in which it was determined that the bot should be removed. 

I did not have time to debug all the issues at the time, and considering the running costs, I suggested removing the bot from the channel, and later shut down the server. So Handybot had a short
but brilliant run, of which I am proud and grateful for.

### Lessons

I was able to learn a couple things about running telegram bots in the wild. I still maintain 
<a href="https://t.me/gsbitbot" target="_blank" rel="noopener noreferrer">GSBitBot</a>
so I was interested in how to keep Handybot running and hardening against a large user base.

I was able to account for the null value issue, as well as other bad
values, from cheeky users poking the bot. There is also an issue
I came across, while researching garbage return values, where users
making requests to the bot from countries that block telegram can result in error messages:

```
error: [polling_error] {"code":"EFATAL","message":"EFATAL: Error: getaddrinfo EAI_FAIL api.telegram.org api.telegram.org:443"}
```

This can happen if uses from, say, the Russian Federation make a request to the bot. This solution dicussed 
<a href="https://github.com/yagop/node-telegram-bot-api/issues/696" target="_blank" rel="noopener noreferrer">here</a> 
is to use a SOCKS5 proxy. I wasn't interested in implementing this, 
nor paying for a proxy service, so I never hardened against this.

I did, however, add better error checking to Handybot and the other
maintained bot to account for all possible errors that are described
in the telegram bot api 
<a href="https://core.telegram.org/bots/api" target="_blank" rel="noopener noreferrer">documentation</a>
which are detailed in the 
<a href="https://github.com/yagop/node-telegram-bot-api" target="_blank" rel="noopener noreferrer">
node-telegram-bot-api</a> documentation as well.

The polling errors can be handled with:

```js
bot.on('polling_error', (error) => {
  console.log(error.code);  // => 'EFATAL'
});
```

and any webhook errors:

```js
bot.on('webhook_error', (error) => {
  console.log(error.code);  // => 'EPARSE'
});
```

After adding error checking for these, GSBitBot has been running
smoothly so far. Handybot is not currently running, but these
additional checks should have caught any unwarranted return messages 
from the bot back to the channel, and the bot would most likely run
just fine there.

Note GSBitBot runs without incident for long periods of time, in the
order of months. Every now and again it will become unresponsive. Adding the additional error check is being tested in a production environment, as this bot is public, to see if the error checking makes
the bot more resilient. 

### Bye For Now

The hackathon was a fun experience, and it was great to have the
project used for its designed purpose, even if only for a brief, shining moment. We love you Handybot!

![handybot](/assets/posts/i-got-a-shirt/_handy.png)

*disclaimer: these musings are offered, at best, as educational, and at worst for entertainment purposes. Do not take action on the descriptions above, as they contain risks, and are not intended as financial advice. Do not do anything above.*