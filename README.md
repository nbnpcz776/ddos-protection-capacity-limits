# DDoS Protected VPS: What Built-In Protection Really Covers, Where the Capacity Ceiling Bites, and How to Choose a Plan (Vultr's 10 Gbps Spec, Explained)

Almost every VPS host now advertises DDoS protection, and almost none of them will tell you how much of it you get. Vultr publishes a number: **10 Gbps of mitigation capacity per instance**, applied to Layer 3 and Layer 4 attacks, in every location it runs. That one sentence answers more buying questions than any "enterprise-grade security" badge on a landing page.

What follows is the part the marketing pages skip: which layers protection actually covers, why the free tier is usually smaller than the headline implies, three details buried in Vultr's documentation that decide whether mitigation works at all, and the security work a DDoS-protected VPS does not do for you.

> **Most readers:** a Vultr Cloud Compute instance with DDoS protection switched on handles volumetric SYN, UDP and ICMP floods up to 10 Gbps per instance, in all locations, without handing your traffic to a third party. Pick the instance size your workload needs, then enable protection from that instance's DDoS tab.
>
> 👉 [See Vultr Cloud Compute plans and deploy an instance](https://www.vultr.com/products/cloud-compute/?ref=9738262-9J)

## What Vultr's built-in protection actually includes

Vultr documents its mitigation stack in more detail than most competitors, which makes it a useful reference point even if you end up elsewhere.

| What you're checking | What Vultr's documentation states |
| --- | --- |
| Mitigation capacity | 10 Gbps per instance |
| Attack layers covered | Layer 3 and Layer 4, both TCP and UDP |
| Detection | Automated, based on proprietary detection algorithms; mitigation generally begins within 60 seconds of an attack being detected |
| Where filtering happens | Traffic is re-routed to Vultr's Attack Mitigation Farm, filtered against the specific attack's attributes, then tunnelled back to your server in the same data center |
| Coverage | All Vultr locations; all IPv4 addresses attached to the instance |
| Eligible instances | Cloud Compute and High Frequency Compute |
| How you turn it on | Customer Portal → your instance → DDoS tab → Enable DDoS Protection, or Terraform with `ddos_protection=true` |
| Prerequisite | The instance must use Vultr's recursive DNS resolver (108.61.10.10) |

The routing detail matters more than it looks. Because clean traffic is tunnelled back inside the same data center, your packets never leave Vultr's network on the way to a scrubbing provider and back. That's the usual source of the latency spike people associate with DDoS mitigation: a third-party scrubbing centre in another region adds a round trip to every legitimate request. Vultr's setup avoids that hop by keeping the whole loop in-house.

## "Built-in DDoS protection" means different things at different layers

The phrase gets used for anything from a firewall rule to a global scrubbing network. When you're comparing hosts, run through these layers and ask which ones are actually included.

**Network edge filtering.** Traffic is inspected and malformed packets, spoofed source IPs and volume spikes are dropped before they reach your server. Ask whether this is always on or only triggered after an attack is detected.

**Scrubbing centres.** High-capacity systems that strip attack signatures out of a flood in real time. They normally sit upstream, before traffic hits your hosting environment. Ask where they are located relative to your server.

**Real-time analytics.** Dashboards showing blocked IPs, request anomalies and post-attack logs. Without this you can't tell a mitigated attack from a slow afternoon, and you can't prove to anyone what happened.

**Firewall and rate limiting.** Port and protocol restrictions, access control, automatic drops on suspicious sources. Often partially your responsibility even on a "protected" plan.

**CDN and WAF.** A CDN serves cached content from edge servers so your origin survives a traffic surge. A WAF inspects HTTP requests and blocks bot spam, brute-force logins and query-based attacks. These are the pieces that address Layer 7, and they're the ones most often missing from a "DDoS protected" claim.

**IDS, geo-blocking, automatic patching.** Present on some platforms, absent on many. Worth asking about if you handle sensitive data.

Vultr's published protection covers Layer 3 and Layer 4. That's the flood layer: SYN floods, UDP floods, ICMP floods, DNS amplification. Application-layer attacks that look like ordinary HTTP requests are a different problem, and you'll want a WAF or CDN in front of the application for those.

## Free tier vs paid tier: the gap is capacity, not features

Liquid Web publishes the clearest breakdown of what "free DDoS protection" typically means, and the numbers are worth seeing side by side.

| Tier | Price | What it covers |
| --- | --- | --- |
| Standard | Free | Small targeted attacks; Layer 4 volumetric, up to 2 Gbps |
| Advanced | $99/mo per server | Large targeted attacks; volumetric, up to 10 Gbps |
| Premium | $499/mo per account | Complex or sustained attacks; "complete protection" |

The free tier isn't a stripped-down feature set. It's the same detection and mitigation machinery with a **2 Gbps ceiling** and no coverage for sustained campaigns. That's genuinely enough for a lot of small sites, because most attacks are short and small. It stops being enough the moment someone decides to spend real money targeting you.

Other providers publish claims you can't convert into a number. OVHcloud describes its VPS protection as always-on and unmetered, included with every plan at no extra cost, and explicitly notes that no solution can guarantee 100% mitigation in all cases. BlastVPS advertises always-on Layer 3/4 mitigation with "Tbps capacity" across global scrubbing centres, included in plans from $10.99/mo. Neither publishes a per-instance ceiling you can compare against 2 Gbps or 10 Gbps, and "unmetered" and "Tbps" aren't interchangeable units. Treat them as different claims, not different sizes of the same thing.

For hardware context: entry-level protected VPS plans in the $10.99–$39.99/mo range typically give you 2 to 6 vCores on Ryzen-class CPUs, 4 to 12 GB RAM, 40 to 100 GB NVMe and unmetered or multi-terabyte bandwidth. Liquid Web's protected VPS starts at $17/mo for 1 vCPU, 4 GB RAM, 120 GB storage and 2 TB bandwidth, or $22/mo for 3 vCPU, 2 GB RAM and 60 GB. Those pages date from early and mid-2025, so use them as a band, not as today's lowest price.

👉 [Compare Vultr Cloud Compute configurations](https://bit.ly/VuLtr)

## Three details in Vultr's docs that decide whether mitigation works

These are the things that turn "protection enabled" into "protection that actually fires".

**1. Your server has to use Vultr's DNS resolver.** Protection depends on the instance resolving through 108.61.10.10. Vultr sets this automatically via DHCP, but if you've edited your network configuration, a custom resolver, a VPN client or a hardening script can quietly break the link. Verify the resolver before you assume you're covered. This is a five-minute check that most people never do.

**2. Mitigation only applies to Vultr-owned IP addresses.** Vultr monitors all IP space for attacks, including customer-announced ranges, but scrubbing is performed only for Vultr IPs. If you bring your own IP block, an attack that exceeds certain heuristics gets a temporary null-route instead: your traffic is blackholed until the attack subsides. If your architecture depends on your own address space, that's a meaningful difference between "filtered" and "offline".

**3. It's a per-instance switch, not an account setting.** You enable it on each instance, through the portal or with one line of Terraform. Spin up three instances and you have three switches. An older Vultr doc page mentioned higher protection levels as a future add-on subscription; the current documentation page, updated in September 2025, describes only the per-instance enable flow. If you expect attacks larger than 10 Gbps, confirm what's available before you commit rather than assuming a bigger tier exists.

👉 [Deploy a Cloud Compute instance with DDoS protection enabled](https://www.vultr.com/products/cloud-compute/?ref=9738262-9J)

## What the protection doesn't do for you

A DDoS-protected VPS keeps your server answering during a flood. It does not close the other doors.

- **Your origin IP is still exposed** if you registered it publicly. Put a reverse proxy or CDN in front, or tunnel traffic, so attackers hit an edge network instead of your box.
- **Default ports and password logins** are the easiest way in after the flood ends. Switch SSH to key authentication, disable password login, move admin panels off default ports, close ports you don't use.
- **No rate limiting on your own endpoints** means an attacker can exhaust application resources with a handful of requests per second and never trip a volumetric threshold.
- **No off-site backup, or an untested one,** is the failure mode that ends businesses. Schedule backups off the server and actually run a restore.
- **No written incident response plan** turns a mitigated attack into three hours of confusion. Know who gets paged, what you check first, and what you tell customers.

Liquid Web's own summary of the limits is the fair version: a DDoS-protected VPS is often enough for small to mid-sized sites, SaaS apps and ecommerce stores, but it isn't a silver bullet. Finance, healthcare and online gaming face more sophisticated campaigns and usually need dedicated hardware, private cloud or managed security on top. What protection buys you is time and breathing room during an attack, not immunity from one.

## Which setup fits your situation

**Static site, blog, small brochure site.** Enable the built-in protection, skip the paid tier entirely. A 2 Gbps Layer 4 ceiling covers the random, opportunistic attacks that hit small properties. Spend the money on backups instead.

**SaaS app or ecommerce store with real revenue per hour.** You need Layer 4 capacity plus something at Layer 7. Check whether the protection includes a WAF or CDN integration, and check whether the free tier auto-escalates to a paid tier when an attack exceeds its ceiling, because that's a surprise invoice if you didn't ask.

**Game server.** UDP traffic is the attack vector, and game protocols need filtering rules that don't drop legitimate player packets. Ask specifically about UDP handling and whether game-specific rules exist, not just "DDoS protection included".

**Agency or reseller hosting multiple clients.** Per-account pricing beats per-server pricing at volume. $499/mo per account covering many servers is a different calculation from $99/mo per server multiplied by ten.

**Bank, hospital, government portal.** A VPS with built-in protection is a starting layer, not a compliance answer. Budget for managed detection and response alongside it.

## FAQ

**Can any DDoS-protected VPS stop 100% of attacks?** No. OVHcloud states this outright: no solution guarantees full mitigation in every case. Results depend on attack volume and complexity, the capacity of the mitigation system, and vulnerabilities in your own application. Anyone promising otherwise is selling.

**What's the difference between Layer 3/4 and Layer 7 protection?** Layer 3 and 4 cover the network and transport layers: floods of packets, SYN floods, UDP and ICMP floods, DNS amplification. Layer 7 covers HTTP and application logic: attacks that look like normal requests. Vultr's published protection is Layer 3/4. Layer 7 needs a WAF or CDN in front.

**Will I get charged automatically if an attack exceeds the free tier?** That's provider-specific and worth confirming in writing before you buy. Liquid Web sells its higher tiers as explicit add-ons at $99/mo and $499/mo, which means no surprise escalation. Ask any host what happens at the moment your attack exceeds the included capacity.

**Does a small site need a paid tier?** Usually not. If your traffic is modest and nobody has a reason to target you, the included protection handles the noise. Revisit when downtime costs you more per hour than the upgrade costs per month.

## Before you order

Two things to confirm at checkout, whichever host you pick: the billing period and what happens at renewal, since introductory pricing and renewal pricing are often different numbers, and the exact capacity figure behind the protection claim. A published number beats an adjective every time.

If the 10 Gbps per-instance ceiling and Layer 3/4 coverage match your risk profile, the setup is straightforward: pick an instance size, deploy, verify the DNS resolver, then flip protection on from the DDoS tab.

👉 [Start with a Vultr Cloud Compute instance](https://bit.ly/VuLtr)
