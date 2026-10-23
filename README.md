# virtual private server hosting: what $4 a month actually buys, and when paying 6–10× for a China route pays off

Two VPS plans can both say "2 vCPU / 4 GB / 80 GB SSD" and perform ten times apart. The differences that matter — whether the virtualization is real hardware virtualization, whether the cores are dedicated or burstable, whether the disk is SATA or NVMe, and which city the box sits in — are almost never on the pricing page. Everything else in VPS shopping follows from getting those four things right and ignoring the rest of the marketing.

This guide covers what a virtual private server actually is, how to read a spec sheet without getting fooled, what the market charges in 2026, and where the hidden costs live. Along the way I'll use GoMami — a Hong Kong/Japan/Singapore/Los Angeles provider built specifically around mainland China routing — as the worked example for the one scenario where paying six to ten times the entry price of a generic cloud is the rational decision.

> **Quick pick.** If your users are mostly in mainland China or greater China, start with a Pulse Nano or Mini: **$29/month** (Japan, Singapore, LA) or **$49–59/month** (Hong Kong), 2 vCPU and 2–4 GB RAM on the same CN2/9929/CMIN2 routing as the expensive tiers. If your users are in Europe or North America and nobody in China needs to reach the box, buy a $4–6 VPS from a mainstream cloud and put the savings into backups. 👉 [Check GoMami's current plan list and pricing](https://bit.ly/Gomami)

## What you're actually renting when you buy VPS hosting

A VPS is a slice of a physical server that behaves like its own machine: root access, its own IP, its own kernel and OS, and CPU/RAM/disk walled off from the other tenants on the same hardware. The isolation is the whole point — a neighbour's runaway script can't read your files or crash your process the way it can on shared hosting.

Whether that isolation is real depends on the hypervisor.

**KVM** is hardware virtualization. Your VPS boots its own kernel, Docker works normally, custom sysctl values stick, and a kernel panic inside your box doesn't touch anyone else. **OpenVZ and LXC** are container-based: you share the host's kernel, so you can't load custom kernel modules. That single limitation cascades — WireGuard and other kernel-module-dependent setups generally won't work, Docker tends to run "but behave strangely," and your swap is fake, meaning it's RAM wearing a swap costume, which turns the host running out of memory into your problem.

The practical check takes five seconds: search the provider's site for "KVM." Providers selling KVM say so loudly. Providers selling OpenVZ at $2/month tend to leave that word out. GoMami's store pages list "VirtIO Port" and third-party reviews describe the platform as KVM, but the documentation doesn't print the word "KVM" anywhere, so treat that as strongly indicated rather than officially certified.

The second thing to check is whether your cores are dedicated or shared. "vCPU" is not a standardized unit. Dedicated cores give you those cycles no matter what; burstable cores compete with neighbours and get throttled under load. AWS T-series, DigitalOcean's Basic Droplets, and most $5 specials are burstable.

Storage has the same problem. "SSD" is a category, not a specification. SATA SSD tops out around 550 MB/s sequential and roughly 100,000 IOPS; NVMe runs 3–7 GB/s and 600,000+ IOPS with about a tenth the random-read latency. For a database or a busy web server, that gap is the difference between snappy and soggy. And RAM: reputable hosts use ECC server memory that corrects single-bit errors. Budget hosts use desktop non-ECC RAM that quietly corrupts data over months.

One more: the "1 Gbps port" on the spec sheet is the host's link speed, not your guaranteed throughput. Real sustained speeds range from 50 Mbps on oversold budget hosts to a clean 1 Gbps+ on well-provisioned ones.

## Sizing: how much VPS you actually need

Most people overbuy. Here's the honest mapping, and it hasn't changed much in years.

- **1 GB / 1 vCPU** — personal blog, low-traffic WordPress, Discord bot, WireGuard endpoint, a 4–8 player Minecraft server.
- **2 GB / 1–2 vCPU** — production WordPress up to ~50,000 monthly visitors with caching, a small SaaS backend, a Postgres database with a few GB of data, 4–6 small Docker containers. This is the sweet spot for freelance and small-business work.
- **4 GB / 2 vCPU** — WordPress at 200,000+ monthly visits, an active Node or Django app, Postgres plus Redis, Nextcloud for a small team.
- **8 GB and up** — busy databases, JVM apps, Elasticsearch, ML inference. At this size, two smaller boxes behind a load balancer usually beat one big one on reliability.

Notice what's absent: nobody's sizing advice says "buy the tier with the highest clock speed" unless the workload is single-thread bound. Which brings us to where the real money goes.

## What VPS hosting costs in 2026, and what the cheap tiers leave out

Entry prices in the mainstream cloud market sit between $2.50 and $5 per month. That's the number most comparison articles quote, and it's accurate — for 512 MB to 1 GB of RAM with shared CPU.

| Provider | Entry price/mo | Entry spec | Notes worth knowing |
| --- | --- | --- | --- |
| DigitalOcean | $4 | 1 vCPU / 512 MB / 10 GB SSD | 500 GiB outbound included; per-second billing with 60-second minimum |
| Vultr | $2.50 IPv6-only / $3.50 IPv4 | 1 vCPU / 512 MB | Backups add 20%; snapshots $0.05/GB/mo; DDoS protection +$10/mo; overage $0.01/GB |
| Hetzner Cloud | ~€4.49 (~$5) | 2 vCPU / 4 GB / 40 GB (CX22) | Cheapest per unit of RAM; raised prices in April 2026 |
| Contabo | ~$4.95 | 4 vCPU / 8 GB | Renewal price holds; network performance varies |
| Akamai Linode | $5 flat | 1 vCPU / 1 GB / 25 GB / 1 TB | Flat pricing has stayed stable for years |
| AWS Lightsail | $5 ($3.50 IPv6-only) | 2 vCPU / 512 MB | Fixed bundles, easiest AWS on-ramp |
| OVHcloud | ~$4.54 | 2 vCore / 4 GB | Anti-DDoS included; intro rates step up |
| Hostinger VPS | $4.99 intro | 1 vCPU / 4 GB / 50 GB NVMe | Renews around $19.49/mo |
| Cloudways | ~$11 | 1 vCPU / 1 GB | Fully managed layer on top of another cloud |

Prices checked August 2026; treat them as a snapshot, not a promise.

The pattern: cheap entry tiers are cheap because they give you 512 MB of RAM and shared CPU. The moment you need 4 GB and dedicated cores, the price climbs toward $20–40/month at the mainstream clouds — which is roughly where the China-optimized providers start.

## Where cheap plans get expensive

The sticker price is rarely the bill. Six things routinely add 20–60%:

1. **Bandwidth overage.** DigitalOcean includes 500 GiB outbound. Vultr charges $0.01/GB past the quota. Serve video or large downloads and this line item dwarfs the server cost.
2. **Backups.** Vultr adds 20% to the base fee. Others bundle it only on higher tiers.
3. **Snapshots.** $0.05/GB/month sounds trivial until you're keeping five of them on a 300 GB disk.
4. **DDoS protection.** A $10/month add-on at Vultr. Whether the default protection is enough depends entirely on what you're hosting.
5. **IPv4.** Vultr and Lightsail both advertise cheaper IPv6-only tiers, with a public IPv4 adding roughly $1–1.50/month.
6. **Renewal cliffs.** Hostinger's $4.99 becomes about $19.49. OVHcloud's intro rates step up. Always budget the ongoing price, not the first invoice.

GoMami handles these differently in one specific respect: the premium China routing is baked into the plan price rather than sold as an add-on. DDoS protection is the opposite story — the docs describe default protection as best-effort, with scrubbing up to 600 Gbps available as an optional advanced tier, and GoMami doesn't publish how that tier is billed. So treat the "600 Gbps" figure on the homepage as a ceiling, not something every $29 plan automatically carries.

Traffic works the other way. Extra bandwidth is sold in self-service packs at **$6 per 100 GB**, linear up to 20 TB for $1,200 — about $0.06/GB, six times Vultr's overage rate. For a normal website that never matters. For a media server, it very much does.

## When the thing you're buying is routing, not specs

Here's the number that reframes the whole comparison: every 100 ms of round-trip latency between your server and your users costs roughly 7% of conversions. A server in Frankfurt serving Jakarta isn't "a bit slower" — it's a different business outcome.

That's the entire premise of the China-optimized VPS category. The relevant routes are:

- **CN2 GIA (AS4809)** — China Telecom's premium backbone; less congestion and packet loss than the ordinary 163 network, especially at 8–11pm.
- **AS9929** — China Unicom's premium international line, steadier than the ordinary 4837 path during peak hours.
- **CMIN2 (AS9808)** — China Mobile's second-generation international network.

A provider offering "CN2 GIA" and nothing else has optimized for one of three carriers. Real triple-network optimization means Telecom goes out on CN2, Unicom on 9929, and Mobile on CMIN2, each on its own premium path. And for website hosting specifically, it's the return path (China → your server → back to China) that determines how fast the site feels, which is why Asian nodes with premium return routing are the usual recommendation for China-facing sites.

GoMami's pitch is exactly this, and it's narrow by design. The site's headline is "The Fastest China Route. The Strongest Performance," with a claimed RTT under 50 ms to mainland China. Four locations: Hong Kong, Japan, Singapore, Los Angeles. Hong Kong, Japan, and Singapore run premium return paths with mainline direct forward paths; Los Angeles runs premium routing on all three carriers in both directions, which is the most aggressive configuration in the lineup.

Independent testing by DigVPS, which tracks these routes over months, rates the Hong Kong Turin and Hong Kong Pulse lines **E2 (excellent)**, Japan and Singapore Pulse **E3 (good)**, and LAX Pulse **E2** after upgrading it from E3 in July 2026 once stability held up. Those ratings come with caveats worth reading: after the October 2025 route switch to CN2/9929/CMIN2, Telecom and Mobile improved while Unicom dropped roughly 30% with added jitter on the Hong Kong line, and Japan saw 40–100% throughput gains across all three carriers.

## Every GoMami plan currently listed

Four product lines, plus a separate set of plans featured on the homepage. Prices are monthly, before any promo code.

### Hong Kong — Turin (AMD EPYC 9575F, Zen 5, up to 5.0 GHz, PCIe Gen5 NVMe, DDR5)

The performance flagship. DigVPS's note is that the 9575F's single-core score nearly matches a desktop Ryzen 9 9950X, which matters for MySQL InnoDB queries that can only use one thread.

| Plan | vCPU / RAM | NVMe | Traffic / port | Price/mo | Order |
| --- | --- | --- | --- | --- | --- |
| HKG.Turin.Mini | 2 / 4 GB | 100 GB | 1 TB / 2 Gbps | $69 | [Order Mini](https://gomami.io/aff.php?aff=415&pid=14) |
| HKG.Turin.Air | 4 / 8 GB | 140 GB | 2 TB / 2 Gbps | $129 | [Order Air](https://gomami.io/aff.php?aff=415&pid=15) |
| HKG.Turin.Pro | 6 / 16 GB | 180 GB | 5 TB / 5 Gbps | $299 | [Order Pro](https://gomami.io/aff.php?aff=415&pid=16) |
| HKG.Turin.Ultra | 12 / 32 GB | 220 GB | 10 TB / 5 Gbps | $599 | [Order Ultra](https://gomami.io/aff.php?aff=415&pid=22) |

Pro and Ultra support Windows installs.

### Hong Kong — Pulse (EPYC 7003-class, around 3.5 GHz)

GoMami's store pages don't publish the CPU model for the Pulse line. The homepage cites EPYC 7763, while third-party listings give 7763, 7773X, 7K83, 7663, or 7K62 depending on location — all EPYC 7003-class parts in the 3.3–3.5 GHz range, so treat the exact SKU as indicative rather than guaranteed.

| Plan | vCPU / RAM | NVMe | Traffic / port | Price/mo | Order |
| --- | --- | --- | --- | --- | --- |
| HKG.Pulse.Nano | 2 / 2 GB | 40 GB | 500 GB / 1 Gbps | $49 | [Order Nano](https://gomami.io/aff.php?aff=415&pid=26) |
| HKG.Pulse.Mini | 2 / 4 GB | 60 GB | 1 TB / 1 Gbps | $59 | [Order Mini](https://gomami.io/aff.php?aff=415&pid=4) |
| HKG.Pulse.Air | 4 / 8 GB | 80 GB | 2 TB / 1 Gbps | $119 | [Order Air](https://gomami.io/aff.php?aff=415&pid=5) |
| HKG.Pulse.Pro | 8 / 16 GB | 100 GB | 5 TB / 3 Gbps | $269 | [Order Pro](https://gomami.io/aff.php?aff=415&pid=6) |
| HKG.Pulse.Ultra | 12 / 32 GB | 300 GB | 10 TB / 3 Gbps | $499 | [Order Ultra](https://gomami.io/aff.php?aff=415&pid=25) |

### Japan — Pulse (EPYC 7003-class, around 3.5 GHz)

| Plan | vCPU / RAM | NVMe | Traffic / port | Price/mo | Order |
| --- | --- | --- | --- | --- | --- |
| JPN.Pulse.Nano | 2 / 2 GB | 40 GB | 500 GB / 1 Gbps | $29 | [Order Nano](https://gomami.io/aff.php?aff=415&pid=13) |
| JPN.Pulse.Mini | 2 / 4 GB | 60 GB | 1 TB / 1.5 Gbps | $49 | [Order Mini](https://gomami.io/aff.php?aff=415&pid=10) |
| JPN.Pulse.Air | 4 / 8 GB | 80 GB | 2 TB / 1 Gbps | $89 | [Order Air](https://gomami.io/aff.php?aff=415&pid=11) |
| JPN.Pulse.Pro | 8 / 16 GB | 100 GB | 5 TB / 3 Gbps | $169 | [Order Pro](https://gomami.io/aff.php?aff=415&pid=12) |
| JPN.Pulse.Ultra | 12 / 32 GB | 300 GB | 10 TB / 3 Gbps | $338 | [Order Ultra](https://gomami.io/aff.php?aff=415&pid=24) |

### Singapore — Pulse (EPYC 7003-class, around 3.5 GHz)

| Plan | vCPU / RAM | NVMe | Traffic / port | Price/mo | Order |
| --- | --- | --- | --- | --- | --- |
| SIN.Pulse.Nano | 2 / 2 GB | 40 GB | 500 GB / 1 Gbps | $29 | [Order Nano](https://gomami.io/aff.php?aff=415&pid=21) |
| SIN.Pulse.Mini | 2 / 4 GB | 60 GB | 1 TB / 1 Gbps | $49 | [Order Mini](https://gomami.io/aff.php?aff=415&pid=17) |
| SIN.Pulse.Air | 4 / 8 GB | 80 GB | 2 TB / 1 Gbps | $89 | [Order Air](https://gomami.io/aff.php?aff=415&pid=18) |
| SIN.Pulse.Pro | 8 / 16 GB | 100 GB | 5 TB / 3 Gbps | $169 | [Order Pro](https://gomami.io/aff.php?aff=415&pid=19) |
| SIN.Pulse.Ultra | 12 / 32 GB | 300 GB | 10 TB / 3 Gbps | $338 | [Order Ultra](https://gomami.io/aff.php?aff=415&pid=23) |

### Los Angeles — Pulse (EPYC 7003-class, triple-network premium in both directions)

| Plan | vCPU / RAM | NVMe | Traffic / port | Price/mo | Order |
| --- | --- | --- | --- | --- | --- |
| LAX.Pulse.Nano | 2 / 2 GB | 40 GB | 1 TB / 1 Gbps | $29 | [Order Nano](https://gomami.io/aff.php?aff=415&pid=27) |
| LAX.Pulse.Mini | 2 / 4 GB | 60 GB | 2 TB / 1 Gbps | $59 | [Order Mini](https://gomami.io/aff.php?aff=415&pid=28) |
| LAX.Pulse.Air | 4 / 8 GB | 80 GB | 4 TB / 2 Gbps | $129 | [Order Air](https://gomami.io/aff.php?aff=415&pid=29) |
| LAX.Pulse.Pro | 6 / 16 GB | 100 GB | 8 TB / 3 Gbps | $259 | [Order Pro](https://gomami.io/aff.php?aff=415&pid=30) |
| LAX.Pulse.Ultra | 12 / 32 GB | 300 GB | 15 TB / 5 Gbps | $599 | [Order Ultra](https://gomami.io/aff.php?aff=415&pid=31) |
| LAX.Pulse.Titan | 12 / 32 GB | 600 GB | 30 TB / 10 Gbps | $999 | [Order Titan](https://gomami.io/aff.php?aff=415&pid=32) |

Note the traffic difference: the LA Nano gives 1 TB where the Asian Nano plans give 500 GB. If you're serving both China and North America, this is the only node with premium routing in both directions. The line launched in June 2026 with a 20% recurring launch code, `Hi,LAX`, which brought the Nano down to $23.20/month — codes like this have a habit of expiring, so check the summary panel at checkout before you assume it still applies. 👉 [Check the LAX Nano's current price and launch discount](https://gomami.io/aff.php?aff=415&pid=27)

### Hong Kong — Forge (dedicated servers, EPYC 7663, 56 cores / 112 threads)

| Plan | RAM / NVMe | Traffic / port | Price | Order |
| --- | --- | --- | --- | --- |
| HKG.Forge.Mini | 128 GB / 960 GB | 10 TB / 2 Gbps | $599/mo + $68 setup | [Ask about Forge Mini](https://bit.ly/Gomami) |
| HKG.Forge.Air | 256 GB / 4 TB | 20 TB / 2 Gbps | $899/mo + $68 setup | [Ask about Forge Air](https://bit.ly/Gomami) |

### The homepage set (separate from the store lines)

GoMami's homepage displays a different set of six plans — three on Ryzen 9 9950X (up to 5.7 GHz) and three on EPYC 7763 — which don't line up with the store product lines and shouldn't be read as the complete price list. They're worth knowing about if you want the highest single-core clock in the lineup: Ryzen 9 9950X at $59 (2 vCPU / 4 GB / 40 GB / 1 TB / 2 Gbps), $99 (4 / 8 / 60 GB / 2 TB), and $199 (6 / 16 / 80 GB / 5 TB). The EPYC 7763 trio runs $39, $79, and $169. 👉 [See which of these are currently orderable](https://bit.ly/Gomami)

## The billing rules and limits that decide whether this is worth it

**Payment covers the obvious gaps, including Alipay.** GoMami takes credit and debit cards through Stripe, Alipay through Stripe with CNY accepted, and major cryptocurrencies including USDT. You can also pre-fund an account balance and let renewals draw from it — worth doing if you don't want a suspended server because a card quietly expired. The payment method you pick also sets how fast a refund resolves: Alipay lands in 1–3 business days, cards take 5–10, crypto 1–3. If you need a formal invoice, support issues one from a ticket with company name, tax ID, amount, and contact details.

**Annual billing is where the pricing gets reasonable.** GoMami's docs say annual billing typically saves 15–20% versus monthly. The widely circulated code `GOMAMI365` advertises 20% off recurring on annual plans across the lineup. Whether the code stacks with the standard annual discount isn't documented anywhere I could find, so treat the numbers below as estimates rather than quotes:

- **HKG.Pulse.Mini** at $59/month is $708/year monthly-billed; with 20% off it lands near **$566/year**.
- **HKG.Turin.Mini** at $69/month: $828/year monthly, roughly **$662/year** discounted.
- **JPN.Pulse.Nano** at $29/month: $348/year monthly, roughly **$278/year** discounted.

Confirm the code on the checkout page before you commit — promo validity changes. 👉 [Apply GOMAMI365 on the annual checkout and see the discounted total](https://bit.ly/Gomami)

**The refund window is 24 hours, once, on first purchases only.** Full refund within 24 hours of activation, limited to one time per user, returned to the original payment method. Renewals don't qualify. Neither do services terminated for ToS violations, used account credit, or an IP blackholed by a DDoS attack. There is no free trial — the 24-hour window is the trial, so use it to run your own latency tests rather than to "try it later."

**Expiry has no grace period.** Service is suspended on the due date. Five days overdue and it's terminated, with data potentially unrecoverable. Renewal invoices are generated seven days ahead and, if you have a card on file, charged automatically. If you're the type who pays when a reminder lands, set a calendar entry.

**Plan changes require a support ticket.** There's no self-service upgrade or downgrade in the control panel, which matters if you expect to scale on a Friday night. Support is ticket-first, with published business-day targets of under 2 hours for a fully unavailable server, under 6 for partial outages, and under 24 for general enquiries.

**Sustained full-core load gets throttled.** GoMami's FAQ explicitly says instances flagged as noisy neighbours may face CPU performance throttling. Fine for normal workloads; a problem if your plan involves pinning all cores 24/7.

**On traffic overages**, bandwidth may be throttled after the monthly quota, with specifics depending on the product — and the throttle rate isn't published. Third-party posts cite around 20 KB/s; GoMami doesn't confirm that number. Traffic packs ($6/100 GB up to $1,200/20 TB) are added to the current cycle immediately.

Two facts to plan around rather than assume: GoMami doesn't publish an uptime SLA, and the daily AWS S3 automatic backups that several third-party write-ups mention appear nowhere on the homepage, store pages, or docs. Don't build your recovery plan on either.

## Who should not buy this

If your audience is in Europe, North America, or basically anywhere other than China and its neighbours, a GoMami plan is the wrong purchase. You'd be paying $29–$69/month for routing you can't use, when DigitalOcean, Vultr, or Hetzner will sell you a comparable or larger box for $4–6.

It's also a poor fit for bandwidth-heavy workloads. Serving video, mirroring large files, or running a backup target will burn through 500 GB–1 TB quotas fast, and at $6/100 GB the top-up packs get expensive quickly.

And if you need metered, elastic, spin-up-and-down infrastructure — a fleet of short-lived instances, Kubernetes nodes, managed databases — this isn't that product. It's a flat-monthly VPS with a specific network advantage.

Where it does make sense: e-commerce checkouts serving East Asian customers, game servers where single-core clock speed dominates (a CS server operator's testimonial on GoMami's homepage emphasizes exactly this), cross-border API and microservice backends, and any site where the alternative is apologizing for peak-hour lag every evening.

## Your first ten minutes on a fresh box

Payment to deployment is typically 1–5 minutes, with IP and credentials arriving by email. The control panel is built on VirtFusion and shows billing cycle, expiry, and live CPU/memory/traffic graphs. Supported images include Ubuntu 22.04/24.04, Debian 11/12, CentOS Stream 9, AlmaLinux 8/9, and Windows Server 2019/2022, with custom ISO mounting for anything else.

Then do this, in order:

1. `apt update && apt upgrade -y`, reboot if the kernel updated.
2. Create a non-root user, add to sudo or wheel.
3. `ssh-copy-id` your public key.
4. In `/etc/ssh/sshd_config`: `PermitRootLogin no`, `PasswordAuthentication no`, `PubkeyAuthentication yes`. Reload sshd, then verify in a new terminal before closing the old session.
5. UFW: deny incoming by default, allow outgoing, open 22/80/443.
6. Install fail2ban and enable it on boot.
7. Install Netdata or monit so you find out about a full disk at 3am from an alert instead of a customer.

Ten minutes of work puts you ahead of most internet-facing servers.

## FAQ

**Is a China-optimized VPS necessary if only some of my users are in China?** Judge it by what those visitors do. If they hit a checkout flow, a login, or any conversion-sensitive step, the return-path routing is worth the premium. If China shows up as a handful of sessions in analytics and nothing transactional, it isn't.

**Which GoMami plan fits a normal production website?** HKG.Pulse.Mini at $59 or JPN.Pulse.Mini at $49 — 2 vCPU, 4 GB RAM, 1 TB traffic. The Turin line only pays off when a single thread does most of the work, like InnoDB queries or a game server tick loop.

**Can I change plans later?** Yes, but through a support ticket rather than the control panel.

**What happens when I hit the traffic cap?** Bandwidth is throttled until the next billing cycle. No overage charges, no surprise invoice — just slow until reset, or buy a traffic pack.

**Can I pay with Alipay?** Yes. Stripe Alipay is a checkout option alongside cards and crypto, and it accepts CNY. Refunds to Alipay typically resolve in 1–3 business days.

**Is there a free trial?** No. There's a 24-hour refund window on first purchases, which in practice is the trial period. 👉 [Start with a small Nano and test the routes yourself](https://gomami.io/aff.php?aff=415&pid=13)

**Which CPU do I get on a Pulse plan?** An EPYC 7003-class part around 3.3–3.5 GHz. The store pages don't name the SKU, the homepage says 7763, and third-party listings vary by location — so buy on core count, RAM, and routing, not on a specific model number.

## How to decide, concretely

**Personal blog, side project, or a VPN endpoint, users anywhere but China:** a $4–6 VPS from a mainstream cloud. Spend the difference on backups.

**WordPress or a small SaaS with a mostly-China audience:** JPN.Pulse.Nano at $29 to test the waters for a month, then move to HKG.Pulse.Mini at $59 once you've confirmed the routes behave for your actual users at 9pm.

**Single-thread-bound workload — database queries, game server, real-time app:** HKG.Turin.Mini at $69. This is the one case where paying for clock speed beats paying for cores.

**Serving China and North America from one box:** LAX.Pulse.Nano at $29 or Mini at $59. Only LAX gets premium routing in both directions, and the traffic allowances are the most generous per dollar.

**Multi-tenant or genuinely heavy load:** HKG.Forge, where $599/month plus $68 setup buys an entire 56-core EPYC with 128 GB of RAM and no neighbours.

One last thing before you order: the 24-hour refund window is the only real trial you get, so decide in advance what "good enough" looks like — a ping test from three Chinese cities during peak hours, or an actual page-load measurement against your own site. 👉 [Pick a plan and run the test within the first 24 hours](https://bit.ly/Gomami)
