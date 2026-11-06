# backup vps: What "Backup VPS" Actually Means, How to Set One Up for Under $10, and What GreenCloud's Free Snapshot Really Covers

Search "backup vps" and you're usually trying to do one of two things: back up an existing VPS so a dead server doesn't take your data with it, or buy a second cheap VPS purely to store those backups. This article covers both — what budget hosts like GreenCloud actually include for free, why that's less than it sounds, how to set up automated rsync backups in about five minutes, and which plans make sense as a backup target.

## Quick take

If you want the short version: a single snapshot from your host is not a backup strategy. GreenCloud includes **1 free Backup/Snapshot** with each Budget KVM plan (from **$6/month**), which is fine for rolling back a bad update and nearly useless if the whole server dies. The practical setup most people land on is a cheap main VPS plus a second cheap VPS in a different location as the backup target, synced nightly with rsync — total cost around $12/month if both are GreenCloud Budget KVM plans.

👉 [Check GreenCloud's Budget KVM plans from $6/mo](https://greencloudvps.com/billing/aff.php?aff=6679)

## What backup features do VPS hosts actually include?

Less than most product pages imply. Here's what three providers currently offer, based on their own documentation:

- **GreenCloud**: every Budget KVM plan ships with 1 free Backup/Snapshot by default. There's no automated daily backup service — the snapshot is manual, and one copy stored on the same infrastructure can't survive a serious host-level failure.
- **OVH**: every VPS includes "Standard" automatic backups covering the system disk only, with a retention of just 24 hours. A paid Premium option extends that to 7 days, and backups are stored off the VPS with triple replication inside the same datacenter. Extra disks are excluded.
- **Contabo**: Auto Backup is a paid add-on for Core and Performance VPS. It takes one backup per day (incremental after the initial full), keeps up to 10 backups for a maximum of 10 days each, and stores them off-server. Restores are whole-VPS only — you can't pull a single file back — and Contabo explicitly states it does not guarantee recoverability.

Notice the pattern: even where "automatic backups" are included in the price, retention is short (24 hours at OVH), scope is limited (system disk only), or the feature costs extra (Contabo). GreenCloud sits at the most minimal end of this range — one snapshot, no automation — which is why the rest of this article assumes you'll handle backups yourself.

One correction worth making here, because it trips up a lot of people: a snapshot is a point-in-time image of the disk, useful for rolling back before a risky update. It is not a substitute for real backups. If your server is compromised or the host has a hardware failure, a snapshot stored on that same host may be gone with everything else.

## GreenCloud's full plan lineup

GreenCloud (greencloudvps.com, run by a Delaware LLC operating since 2013) sells several product lines relevant here. All Budget KVM and Ryzen KVM plans are Linux-only with 10Gbps ports and run on an unmanaged basis — the company handles the physical server and uptime, and nothing else. Prices below are the official monthly rates.

**Budget KVM VPS** — enterprise SSD in RAID-10, available in US, EU, and APAC locations, includes 1 free Backup/Snapshot and a cloud control panel for reinstalls and power management:

| Plan | Storage | CPU / RAM | Bandwidth | Price | Order |
| --- | --- | --- | --- | --- | --- |
| SSDKVM-1 | 15GB SSD | 1 core / 1GB | 1.5 TB | $6/mo | [ Order SSDKVM-1](https://greencloudvps.com/billing/aff.php?aff=6679) |
| SSDKVM-2 | 15GB SSD | 1 core / 2GB | 2 TB | $10/mo | [ Order SSDKVM-2](https://greencloudvps.com/billing/aff.php?aff=6679) |
| SSDKVM-3 | 30GB SSD | 2 cores / 4GB | 4 TB | $20/mo | [ Order SSDKVM-3](https://greencloudvps.com/billing/aff.php?aff=6679) |
| SSDKVM-4 | 60GB SSD | 4 cores / 8GB | 8 TB | $40/mo | [ Order SSDKVM-4](https://greencloudvps.com/billing/aff.php?aff=6679) |
| SSDKVM-5 | 120GB SSD | 8 cores / 16GB | 12 TB | $80/mo | [ Order SSDKVM-5](https://greencloudvps.com/billing/aff.php?aff=6679) |

**Ryzen KVM VPS** — NVMe storage on Ryzen cores clocked at 3.5+ GHz, same prices as Budget KVM but with different storage sizing. Bandwidth differs sharply by region:

| Plan | Storage | CPU / RAM | Bandwidth (USA/EU) | Bandwidth (APAC) | Price | Order |
| --- | --- | --- | --- | --- | --- | --- |
| RyzenKVM-1 | 15GB NVMe | 1 core / 1GB | 1.5 TB | 750 GB | $6/mo | [ Order RyzenKVM-1](https://greencloudvps.com/billing/aff.php?aff=6679) |
| RyzenKVM-2 | 30GB NVMe | 1 core / 2GB | 2 TB | 1 TB | $10/mo | [ Order RyzenKVM-2](https://greencloudvps.com/billing/aff.php?aff=6679) |
| RyzenKVM-3 | 50GB NVMe | 2 cores / 4GB | 4 TB | 2 TB | $20/mo | [ Order RyzenKVM-3](https://greencloudvps.com/billing/aff.php?aff=6679) |
| RyzenKVM-4 | 80GB NVMe | 4 cores / 8GB | 8 TB | 4 TB | $40/mo | [ Order RyzenKVM-4](https://greencloudvps.com/billing/aff.php?aff=6679) |
| RyzenKVM-5 | 150GB NVMe | 8 cores / 16GB | 12 TB | 6 TB | $80/mo | [ Order RyzenKVM-5](https://greencloudvps.com/billing/aff.php?aff=6679) |

**Budget Windows VPS** — Windows Server 2022, 1Gbps port, unmetered bandwidth. The "Normal" tier is monthly billing; the "SuperSaving" tier requires paying at least quarterly to get the discounted rate:

| Plan | Storage | CPU / RAM | Tier | Price | Order |
| --- | --- | --- | --- | --- | --- |
| US1 | 25GB SSD | 1 core / 1GB | Normal | $8/mo | [ Order US1](https://greencloudvps.com/billing/aff.php?aff=6679&pid=335) |
| US1.5 | 30GB SSD | 1 core / 1.5GB | Normal | $12/mo | [ Order US1.5](https://greencloudvps.com/billing/aff.php?aff=6679&pid=339) |
| US2 | 40GB SSD | 2 cores / 2GB | Normal | $16/mo | [ Order US2](https://greencloudvps.com/billing/aff.php?aff=6679&pid=340) |
| US3 | 60GB SSD | 2 cores / 3GB | Normal | $24/mo | [ Order US3](https://greencloudvps.com/billing/aff.php?aff=6679&pid=341) |
| US4 | 80GB SSD | 4 cores / 4GB | Normal | $32/mo | [ Order US4](https://greencloudvps.com/billing/aff.php?aff=6679&pid=342) |
| US5 | 100GB SSD | 5 cores / 5GB | Normal | $40/mo | [ Order US5](https://greencloudvps.com/billing/aff.php?aff=6679&pid=343) |
| US6 | 120GB SSD | 6 cores / 6GB | Normal | $48/mo | [ Order US6](https://greencloudvps.com/billing/aff.php?aff=6679&pid=344) |
| Budget-1 | 25GB SSD | 1 core / 2GB | SuperSaving | $10/mo | [ Order Budget-1](https://greencloudvps.com/billing/aff.php?aff=6679&pid=334) |
| Budget-2 | 40GB SSD | 2 cores / 4GB | SuperSaving | $20/mo | [ Order Budget-2](https://greencloudvps.com/billing/aff.php?aff=6679&pid=337) |
| Budget-3 | 80GB SSD | 4 cores / 8GB | SuperSaving | $40/mo | [ Order Budget-3](https://greencloudvps.com/billing/aff.php?aff=6679&pid=338) |

All plans can be upgraded with add-ons at $3 per extra 1GB of RAM, $3 per CPU core, or $3 per 10GB of SSD per month, capped at doubling the original configuration (a 4GB plan can take up to 4GB more). Availability fluctuates: when we checked the Budget KVM store, most EPYC/NVMe locations were sold out and in-stock nodes were running older Xeon E5v4 hardware with SATA SSD, so if NVMe performance matters to you, check what's actually in stock before ordering.

👉 [See GreenCloud's current KVM locations and stock](https://greencloudvps.com/billing/aff.php?aff=6679)

## How to set up automated backups with rsync and a second VPS

The cheapest reliable method is a nightly rsync job from your main VPS to a backup VPS over SSH. RackNerd's own blog walks through essentially this setup and notes you can have it running in under five minutes; the same approach works on any two Linux servers, including a pair of GreenCloud Budget KVM plans.

Setup steps:

1. Generate an SSH key on the main server: `ssh-keygen -t rsa -b 4096`
2. Copy the public key to the backup server: `ssh-copy-id root@BACKUP_VPS_IP`
3. On the backup server, create a target directory: `mkdir -p /home/backup`
4. Make sure rsync is installed on both machines — `sudo yum install rsync -y` on CentOS/AlmaLinux/Rocky, or `sudo apt-get update && sudo apt-get install rsync -y` on Ubuntu/Debian
5. Add a cron job on the main server for a nightly 2 AM sync:


(crontab -l 2>/dev/null; echo "0 2 * * * rsync -avz --delete --exclude={'/dev/*','/proc/*','/sys/*','/tmp/*','/run/*','/mnt/*','/media/*','/lost+found'} -e ssh / root@BACKUP_VPS_IP:/home/backup") | crontab -


6. Verify with `crontab -l`.

The backup VPS should have at least as much storage as the main one, and putting it in a different datacenter than your primary server is the whole point — a fire or network failure in one facility shouldn't take out both copies.

Know the limitations of this script before you trust it. RackNerd's guide flags them itself: there's no check on whether the backup server is reachable or has free disk space, so if the target fills up, you get no notification. Worse, `--delete` makes the backup an exact mirror — if you accidentally delete a file on the main server, the next nightly sync deletes it from the backup too. Your recovery window is whatever time remains before 2 AM. If that risk bothers you, look at tools like restic or BorgBackup, which keep versioned history instead of a mirror.

The economics work out well either way. Two GreenCloud SSDKVM-1 plans (15GB each) run $6/month each, so $12/month total for a primary-plus-backup pair. If your main server is bigger, an SSDKVM-3 (30GB, 4TB bandwidth) at $20/month covers a 15–25GB main server comfortably. For comparison, RackNerd has promoted a 100GB storage VPS at $35/year as a dedicated backup box, and Reddit users commonly pair a cheap VPS with Hetzner's Storage Box, whose lowest tier was listed at €3.81/month — so a dedicated second box doesn't have to cost much, whichever route you pick.

👉 [Get a Budget KVM plan for your backup target](https://greencloudvps.com/billing/aff.php?aff=6679)

## The fine print that affects your decision

A few details that don't make it into the headline pricing, all confirmed from GreenCloud's own pages:

- **APAC bandwidth is roughly half.** The same-priced Ryzen KVM plan gets 1.5TB of monthly traffic in the US/EU but 750GB in Singapore, Japan, Hong Kong, or Vietnam. Budget KVM follows the same pattern. If your backup jobs or sites move serious traffic, US/EU locations give you twice the headroom per dollar.
- **Refund policy has a catch.** New customers can get a refund on their first VPS within 7 days, but special discounted plans are excluded — and GreenCloud's own Budget KVM store listing tags specific configurations (for example, the Singapore DC1 plans) as no-refund. Read the note attached to your exact plan and location before paying, because the general policy and the store-page note don't always match.
- **Unmanaged means unmanaged.** GreenCloud's stated policy: "We take care of the physical server and its uptime only." Software problems, backup script debugging, restore assistance — all on you. This is normal at these prices, but it matters for a backup setup, since you're the one who has to test restores.
- **Payment and activation.** PayPal, cards, Alipay, UnionPay, crypto (BTC/ETH/BNB/USDT), and US/UK/EU bank transfers are accepted. First-time card or PayPal customers go through a one-time fraud verification, and provisioning takes 15 minutes at best, up to 2 hours in slower cases.
- **Network scale, with caveats.** GreenCloud advertises 32 datacenters across 30 locations, 600+ Gbps of aggregate bandwidth, 10Gbps uplinks, and a 99.99% network/power uptime target, with average ticket responses around 9 minutes. These are the company's own figures rather than third-party measurements, though the multi-region footprint is real and matters if you want your backup box geographically separated from your main server.

## FAQ

**Does GreenCloud back up my VPS for me automatically?**
No. Each Budget KVM plan includes one free Backup/Snapshot that you trigger yourself through the VirtFusion control panel. There is no scheduled or off-site backup service — plan on rsync, restic, or similar for anything you care about.

**How much backup retention should I aim for?**
Liquid Web's guidance is 30–90 days for typical needs, with daily backups for critical business data and weekly backups for system files. A single nightly rsync mirror gives you one day of protection; adding restic with weekly snapshots gets you closer to that range without much extra cost on a cheap backup VPS.

**Can I use a GreenCloud VPS as a backup target for a server hosted elsewhere?**
Yes — that's exactly the second-VPS pattern described above, just with the roles reversed. Pick a location different from your main host, and remember APAC plans carry half the bandwidth of US/EU plans at the same price.

**What about restoring a single file?**
That depends entirely on your method. rsync mirrors let you just copy the file back. Contabo-style whole-VPS restores (on providers that offer managed backups) don't allow single-file recovery at all, which is one more reason to run your own file-level backups.

## Which setup should you pick?

Match the plan to the job:

- **Main server for a small site or personal project** → SSDKVM-1 ($6/mo) or RyzenKVM-1 ($6/mo, NVMe, faster CPU clock) if a 15GB disk is enough. Step up to SSDKVM-3/RyzenKVM-3 ($20/mo, 4GB RAM) once you're running databases.
- **Backup target only** → the cheapest plan that fits your data with headroom. An SSDKVM-1 holds 15GB for $6/mo; for a 50–60GB main server, SSDKVM-4 at $40/mo or a RyzenKVM-4 in a different region.
- **Windows required** → the Budget Windows line starts at $8/mo (US1), and Budget-1 gives you 2GB RAM for $10/mo if you're willing to pay quarterly.
- **Need the host to handle backups for you** → GreenCloud won't; OVH includes daily system-disk backups in every VPS and Contabo sells a 10-day add-on, so weigh that against doing it yourself with scripts.

One last thing before you order: confirm the location is actually in stock and check the refund note on the specific plan — Budget KVM stock rotates quickly, and discounted or certain APAC configurations are excluded from the 7-day new-customer refund. If both check out, 👉 [open GreenCloud's plan page and lock in your setup](https://greencloudvps.com/billing/aff.php?aff=6679), then get the cron job running the same day. A backup plan that hasn't been tested yet is just a theory.
