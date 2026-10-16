# vps for discord bot: cheap 24/7 hosting from $10.99 a year, how to pick the right plan, and a setup that auto-restarts your bot

A Discord bot is one of the lightest things you will ever host. A typical discord.py or discord.js bot sits at 50–200 MB of RAM, idles the CPU, and pushes a few kilobytes per second across the wire, even in a few hundred servers. The specs on a listing page matter less than two things people tend to overlook: the server never goes to sleep, and the bot comes back on its own after a crash. Get those right and almost any entry-level VPS does the job.

If you're shopping for a VPS for a Discord bot, the cheapest reliable way to do it right now is RackNerd's 1 GB KVM plan at **$10.99 a year** — prepaid, no setup fee, roughly 92 cents a month. One box that size comfortably runs two or three small bots. Below is how to size a plan to your bot, what RackNerd's annual pricing looks like next to the monthly-billed competition, and the 20-minute setup that keeps a bot alive through crashes and reboots.

> **Quick picks**
> - **Most people:** RackNerd's **1 GB plan at $10.99/year** — enough for one to three small bots. [👉 check the current 1 GB price](https://bit.ly/my-rn)
> - **Music bot, database, or four-plus bots:** the **2.5 GB / 2 vCPU tier at $26.99/year**. Audio encoding is the only thing in this category that genuinely wants CPU.
> - **Want everything preinstalled?** OliveVPS ships a bot-tuned stack (Node.js, Python, PM2 ready to go) at $7.99/mo — convenient, but roughly nine times the annual cost.
> - **Paying with crypto, no KYC:** EQVPS's Nano plan ($3/mo, 1 GB RAM, Germany or Finland) is the usual answer.

## What a Discord bot actually needs from a VPS

Less than almost anything else you could put on a server. That's the starting point, because it changes what "cheap" means here.

The resource profile, roughly: a Python or Node bot idles at 50–300 MB of RAM, and even bots serving 1,000+ servers rarely cross 1 GB. CPU is mostly idle with short spikes when a command fires. A text bot's network use is a trickle. Disk needs are tiny — the bot, its dependencies, a small SQLite database — and 10–20 GB covers it with room to spare.

Two requirements do the real work:

1. **The machine stays on.** This is where free tiers (Replit, Glitch, Render's free plan) fall apart: they sleep idle containers after a few minutes, cap monthly runtime, and some reset the filesystem on every restart — which quietly deletes your SQLite database on each deploy. A paid VPS has none of those behaviors.
2. **Something restarts the bot when it dies.** Bots crash. A crash at 3 a.m. shouldn't leave the bot offline until you wake up. A process supervisor — systemd or PM2 — with auto-restart handles this. The exact config is in the setup section below.

One thing you don't need: open inbound ports or a dedicated IP. A Discord bot dials out to Discord's gateway and holds a websocket open; nothing on the internet needs to reach your server. Latency also matters less than people assume — the difference between a 30 ms and a 200 ms route to Discord's gateway is barely perceptible in a bot's response time.

**Sizing by bot type:**

| Bot | RAM needed | vCPU | Fits on |
| --- | --- | --- | --- |
| Simple command / utility bot | 512 MB | 1 | 768 MB or 1 GB plan |
| Moderation bot + database | ~1 GB | 1 | 1 GB plan |
| Music bot (Lavalink + ffmpeg) | ~2 GB | 1–2 | 2.5 GB plan |
| AI bot calling LLM APIs | 1–2 GB | 1–2 | 2.5 GB plan |
| Several bots + a web dashboard | 4 GB+ | 2+ | 3.5 GB or larger |

If your bot lives in a dozen servers and handles moderation plus a few games, the smallest tier is genuinely enough. Don't over-buy.

## RackNerd's plans: what you actually pay

RackNerd sells KVM VPSes through a rotating set of annual specials, and the small tiers are where the pricing gets unusual. Every plan includes full root access, a dedicated IPv4, a 1 Gbps port, and your pick of datacenter at checkout — Los Angeles, San Jose, Seattle, Dallas, Chicago, New York, Atlanta, and Amsterdam are the usual options. Payment is card or PayPal, and provisioning is usually done within minutes.

The current specials lineup:

| Plan | RAM / vCPU | Storage | Traffic @ 1 Gbps | Price | Order |
| --- | --- | --- | --- | --- | --- |
| 768 MB | 768 MB / 1 | 12 GB SSD | 750 GB | **$8.99/yr** | [ order 768 MB](https://bit.ly/my-rn) |
| 1 GB | 1 GB / 1 | 20 GB SSD | 1 TB | **$10.99/yr** | [ order 1 GB](https://bit.ly/my-rn) |
| 2.5 GB | 2.5 GB / 2 | 45 GB SSD | 5.5 TB | **$26.99/yr** | [ order 2.5 GB](https://bit.ly/my-rn) |
| 3.5 GB | 3.5 GB / 2 | 60 GB SSD | 6 TB | **$33.99/yr** | [ order 3.5 GB](https://bit.ly/my-rn) |
| 5 GB | 5 GB / 3 | 80 GB SSD | 7 TB | **$58.99/yr** | [ order 5 GB](https://bit.ly/my-rn) |
| 6.5 GB | 6.5 GB / 4 | 100 GB SSD | 8 TB | **$76.99/yr** | [ order 6.5 GB](https://bit.ly/my-rn) |

All six are annual prepay on KVM virtualization, and you choose the Linux distro at setup — Ubuntu 24.04 is a safe pick for bots. One caveat before you click: RackNerd rotates this lineup several times a year, and these are promo prices rather than a fixed permanent catalog, so confirm the exact config and price on the order page. [👉 see the full current lineup and locations](https://bit.ly/my-rn)

**The cost math nobody runs.** Most bot-hosting articles quote monthly prices, which hides how different annual billing is. Over the same twelve months:

- RackNerd 1 GB: **$10.99**
- EQVPS Nano (1 GB): $3/mo → **$36**
- OliveVPS bot plan (1 GB): $7.99/mo → **$95.88**
- Hostinger (4 GB): $4.99/mo → **$59.88**

That list is the core tradeoff in four lines. RackNerd is by far the cheapest way to keep a small bot online around the clock. What you give up is worth stating plainly:

- **A preinstalled bot stack.** You get a bare Ubuntu box with root. Installing Node or Python and a process supervisor is on you — two or three commands, covered below.
- **Monthly billing.** The specials are annual, so you commit a year upfront. If you want month-to-month flexibility, Hetzner or DigitalOcean bill that way.
- **Managed anything.** Support is ticket-based, which is fine for account and network issues, but nobody configures your bot for you.

The traffic allowances look small next to "unmetered" competitors but aren't in practice. A text bot uses kilobytes per second, and even a bot that spends all day streaming in two voice channels (128 Kbps each is about 1.4 GB per channel per day) stays under 100 GB a month. You'd need a large public music bot to push a 1 TB cap.

## How RackNerd compares to the other cheap options

On pure price-per-year for a small bot, RackNerd wins. The real comparison is about what each provider optimizes for.

| Provider | Cheapest relevant plan | Listed price | What it's really for |
| --- | --- | --- | --- |
| RackNerd | 1 GB / 1 vCPU | $10.99/yr | Cheapest 24/7 uptime; DIY setup |
| EQVPS | Nano: 1 GB / 2 vCPU | $3/mo ($36/yr) | Crypto payment, no KYC, EU locations, unmetered |
| Hetzner | 4 GB / 2 vCPU | €3.79/mo | EU-targeted bots, monthly billing, real headroom |
| Hostinger | 4 GB / 1 vCPU | $4.99/mo | Beginners who want a polished panel and live chat |
| OliveVPS | 1 GB / 1 vCPU | $7.99/mo | Preinstalled Node/Python/PM2 bot stack |

Two more worth a sentence each. Contabo (€4.99/mo for 8 GB / 4 vCPU) is the pick if you want maximum specs per dollar for many heavy bots and don't mind slower support. DigitalOcean ($6/mo for 1 GB) is the standard upgrade path when a bot grows into needing managed PostgreSQL, Redis, or hourly-billed infrastructure.

To be fair about the tradeoff: Hetzner gives you four times the RAM at roughly four times the annual cost, with monthly billing and excellent European networking. If you want headroom and flexibility over the lowest possible bill, that's a defensible choice. If the goal is simply "small bot, online 24/7, minimal spend," RackNerd's annual pricing is hard to beat. [👉 compare RackNerd's plans and order](https://bit.ly/my-rn)

## Setting it up: 20 minutes, and the one step everyone skips

The workflow is: order → SSH in → install the runtime → upload the bot → register it with systemd. That last step separates a bot that survives crashes from one you babysit.

1. **Order a plan and pick a location.** Closest to you or your community is fine; gateway latency differences are small.
2. **SSH in** (`ssh root@your-server-ip`) and run `apt update && apt upgrade -y`.
3. **Install the runtime.** Python is ready on Ubuntu 24.04:

   bash
   apt install -y python3 python3-venv python3-pip git
   

   For Node, install it from NodeSource or nvm so you get a current version rather than the older distro package.

4. **Upload the bot.** Clone your repo into `/opt/bot`, install dependencies, and put the token in a `.env` file. Run `chmod 600 .env` and make sure the file is in `.gitignore` — leaked bot tokens are the most common way bots get taken over.

5. **Register it with systemd.** Create `/etc/systemd/system/discordbot.service`:

   ini
   [Unit]
   Description=Discord bot
   After=network-online.target

   [Service]
   WorkingDirectory=/opt/bot
   ExecStart=/usr/bin/node index.js
   Restart=always
   RestartSec=5
   Environment=NODE_ENV=production

   [Install]
   WantedBy=multi-user.target
   

   Then:

   bash
   systemctl daemon-reload
   systemctl enable --now discordbot
   

   `Restart=always` is the line doing the real work: any crash gets a restart within five seconds, and `enable` brings the bot back after a server reboot. A Python bot is identical — point `ExecStart` at your venv's python instead of node.

6. **The step everyone skips: cap your logs.** systemd logs everything to journald, and uncapped logs are the most likely way a 20 GB disk fills up. Set a limit in `/etc/systemd/journald.conf`:

   ini
   SystemMaxUse=200M
   

   then run `systemctl restart systemd-journald`. Two minutes now saves you a full disk six months from now.

If you prefer PM2 for a Node bot, `pm2 start index.js --name bot && pm2 startup && pm2 save` achieves the same thing. Pick one supervisor — running both leaves two processes fighting over the bot. When the setup is done, test it: reboot the server, wait a minute, and check `systemctl status discordbot`. If the bot is online in Discord without you touching anything, you're set.

## FAQ

**Is hosting a Discord bot on a VPS allowed?**
Yes. Discord's terms don't restrict where a bot runs, and the API treats every bot the same regardless of host — it only enforces rate limits. Virtually every serious bot runs on a VPS or dedicated server.

**How many bots fit on the 1 GB plan?**
Two or three small ones. Typical Python or Node bots use 50–300 MB each; give every bot its own systemd service so they restart independently.

**Do I need a dedicated IP or open ports?**
No. Bots connect outbound to Discord's gateway and hold the websocket open, so nothing needs to reach the server from outside.

**What happens when the VPS reboots?**
With `systemctl enable` (or `pm2 startup`) configured, the bot starts on boot and reconnects to the gateway on its own. Bot libraries handle the reconnection handshake.

**What about music bots?**
They're the heavy exception. Run Lavalink with ffmpeg, expect real CPU and RAM use per active voice channel, and take the 2.5 GB tier for a handful of simultaneous channels. Discord has also tightened enforcement around large public YouTube-music bots — a private music bot for your own community remains a normal use case.

**Where should the VPS be located?**
Near you or your community. The response-time difference between a 30 ms and a 200 ms route to Discord's gateway is barely perceptible. Music bots care slightly more about voice-server proximity — US East and Frankfurt are common defaults.

## Which plan to order

- **One or two text bots** (moderation, utility, a few servers): the **1 GB plan at $10.99/year**. More than enough — don't over-buy.
- **Music bot, database, or four-plus bots:** **2.5 GB at $26.99/year**.
- **Want the stack preinstalled and someone to ask when things break:** OliveVPS's $7.99/mo bot plans do that, at roughly nine times the annual bill.
- **Crypto payment or a no-KYC signup matters:** EQVPS's $3/mo Nano plan.

One check before you order the RackNerd plan: the specials rotate, so confirm the current lineup and your preferred datacenter on the order page, and remember you're prepaying the full year. After that, it's a 20-minute setup and your bot stops depending on your laptop staying open. [👉 order the 1 GB plan — $10.99 for the year](https://bit.ly/my-rn)
