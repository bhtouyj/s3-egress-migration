# Amazon S3 Alternatives: Real Egress Math, Managed vs Self-Hosted Picks, and How to Move a Bucket Without Breaking Your Tools

Most searches for Amazon S3 alternatives trace back to one of three frustrations: the egress line on the bill, the creeping sense that petabytes behind a single API is a trap, or a compliance rule that bans public cloud for certain data. The practical picture in 2026 is straightforward. The S3 API won the object storage wars, so almost every credible alternative — managed or self-hosted — speaks the same PUT, GET, LIST, multipart upload and bucket-policy language. Your SDKs, the `aws` CLI and rclone configs keep working. What actually changes is who runs the hardware and how you pay for data leaving the bucket.

That second part is where bills explode. AWS charges $0.09 per GB for data transfer out to the internet, with the first 100 GB each month free. Serve 10 TB in a month and you're north of $900 in egress alone, before storage and request fees. Any comparison that leads with per-GB storage price is solving the smaller half of the problem.

> **Quick pick:** If you mostly serve files to users, price egress first. If you want storage and compute under one account with a low entry point, Vultr's Object Storage starts at **$6/month** for 250 GB of storage plus 1 TB of outbound transfer, with extra bandwidth at $0.01/GB — and Vultr advertises $250 in signup credits for new accounts, valid for 30 days. 👉 [check Vultr's current object storage tiers and signup credit](https://www.vultr.com/products/object-storage/?ref=7268283)

## Why teams leave S3 — and what actually decides the bill

Egress cost, lock-in, and control are the three drivers, roughly in that order of frequency.

Run the numbers on a realistic month and the shape is clear. A store holding 50 TB at S3 Standard's $0.023/GB costs about $1,150/month in storage. Serve that same 50 TB to customers and egress adds roughly $4,500 at $0.09/GB, with API request charges stacked on top. Read-heavy and multi-region workloads get hit hardest, because every byte that crosses out to users or another region goes through the meter.

Lock-in is the quieter problem. Once petabytes sit behind the S3 API in us-east-1, leaving is a multi-week migration project. Regulation is starting to push back here: under the EU Data Act, providers can't charge switching fees — including egress for moving to another provider — from 12 January 2027. Notice what that covers, though: the act applies when you switch away. The meter for everyday reads keeps running either way, so it changes the cost of exit, little about the cost of staying.

To be fair to S3: it launched in 2006 and has offered strong read-after-write consistency since December 2020. The ecosystem around Athena, Glue, and Lambda triggers is real value. Leaving is a business decision about money and control, and for plenty of workloads the alternatives below win it on the numbers.

## Managed S3-compatible cloud: the field in one table

Every provider in this table speaks the S3 API, so the differences come down to egress pricing, per-operation fees, and how much surrounding platform you give up.

| Provider | Storage price | Egress to internet | Worth knowing |
| --- | --- | --- | --- |
| Amazon S3 | $0.023/GB (Standard) | $0.09/GB after the first 100 GB/month | Deepest ecosystem; egress and request fees stack |
| Wasabi | $6.99/TB | $0 | Single hot tier; check minimums and usage policies against your access pattern |
| Backblaze B2 | ~$6/TB | Free up to 3× your stored volume, then $0.01/GB | Cheap base; you bring the compute |
| Cloudflare R2 | $0.015/GB | $0 | Writes $4.50/million, reads $0.36/million; CDN on the same network |
| DigitalOcean Spaces | $0.02/GB | $0.01/GB | Developer-simple, similar bundling logic to Vultr's |

Wasabi fits predictable, storage-heavy workloads where you want one flat number on the invoice. Backblaze B2 is the natural home for backups with modest read-back, since the first 3× of your stored volume in monthly egress is free. Cloudflare R2 makes sense for read-heavy public content — media, static assets — especially if you're already on Cloudflare. It even ships its own import tool for pulling buckets over from S3.

The column most roundups skip: these are storage products, not data platforms. You don't get native Athena, Glue, or Lambda triggers, so if your pipeline leans on those, budget for running that layer elsewhere. For backup, archival, and app storage where you control the compute, that trade is usually acceptable.

## Vultr Object Storage: bundled transfer with a $6 floor

Vultr's approach differs from Wasabi and B2 in one useful way: each tier bundles both storage and outbound transfer, which changes the math for anyone who serves data rather than just archiving it. These are the three tiers currently listed on Vultr's pricing page:

| Tier | Price/month | Storage included | Outbound transfer included | Good fit |  |
| --- | --- | --- | --- | --- | --- |
| Starter | $6 | 250 GB | 1 TB | Small backups, app assets, personal projects | [ Get Starter](https://www.vultr.com/products/object-storage/?ref=7268283) |
| Scale | $10 | 1 TB | 3 TB | Production buckets for apps and sites | [ Get Scale](https://www.vultr.com/products/object-storage/?ref=7268283) |
| Performance | $15 | 1 TB | 5 TB | Read-heavy delivery that needs more throughput | [ Get Performance](https://www.vultr.com/products/object-storage/?ref=7268283) |

Bandwidth beyond the included allotment bills at $0.01/GB — a ninth of AWS's $0.09/GB — and additional storage beyond the included allotment bills per GB as well. Compatibility is the standard drill: the AWS CLI, rclone, s3cmd, and the common SDKs work with just an endpoint and key swap (Vultr endpoints follow the pattern `ewr1.vultrobjects.com`, New Jersey being one of the most-used locations).

Where it fits best: teams already running VMs on Vultr who want servers and buckets on one bill in one console; anyone who wants to test with a low floor price before committing; workloads whose egress stays inside or hovers near the bundle. Where it doesn't: a multi-terabyte cold archive that's almost never read is cheaper on a pure-storage provider, and if your stack depends on AWS-native analytics services, no S3-compatible provider replaces those.

👉 [see the full tier list and current pricing](https://www.vultr.com/pricing/?ref=7268283)

## The egress math on a realistic month

List prices hide the real ranking — you have to model a month to see it. Take a common scenario: 1 TB stored, 5 TB served to users every month.

| Provider | Monthly total (this scenario) | How it adds up |
| --- | --- | --- |
| Amazon S3 | ≈ $464 | $23 storage + ~$441 egress (4,900 GB × $0.09) |
| Wasabi | ≈ $7 | $6.99 for 1 TB, egress $0 |
| Cloudflare R2 | ≈ $15–20 | $15 storage plus a few dollars of operation fees, egress $0 |
| Backblaze B2 | ≈ $26 | $6 storage + 2 TB over the 3× free allowance at $0.01/GB |
| Vultr (Scale) | ≈ $30 | $10 tier + 2 TB overage at $0.01/GB |

Assumptions: roughly 1 MB average object size for the operation-fee estimates, storage sitting right at the included allotment, and published list rates, which vary a little by region and date.

The honest read: Wasabi wins this round on raw dollars, with the caveat that its terms include storage minimums and usage policies you should check against your access pattern before committing. Vultr, R2, and B2 cluster in a $15–30 band — all roughly 15× cheaper than AWS in this scenario, with different burst behavior. The point of the table is the rightmost column of the first row. Whatever else you pick, a $464 month for the same workload is hard to defend.

## Self-hosted engines: MinIO, RustFS, Ceph and the lightweight crew

Self-hosting swaps provider fees for your hardware, your bandwidth, and your on-call rota. Sometimes that's a bargain; sometimes it's an expensive hobby.

**MinIO** is the default self-hosted pick for good reason: released in 2014, written in Go, runs as a single static binary, uses erasure coding for durability, and supports Object Lock for WORM-style compliance. The catch is the license. The core is AGPLv3, and embedding MinIO in a commercial product — or buying official support — requires a commercial license from MinIO, Inc. Fine for internal infrastructure; a legal conversation for ISVs shipping a product.

**RustFS** takes the opposite license approach: written in Rust, Apache 2.0, no commercial-use strings attached. Single binary or Docker image, erasure coding, distributed nodes, Object Lock. It's younger, so expect a thinner community and fewer Stack Overflow answers when something breaks at 2 a.m.

**Ceph** is the pick when object storage is one slice of a bigger problem — its RADOS gateway serves the S3 API while RBD and CephFS cover block and file from the same cluster. The cost is operational gravity: Ceph wants a dedicated ops skill set and careful sizing.

**SeaweedFS** and **Garage** occupy the lightweight niches: SeaweedFS for enormous file counts and small-object performance (thumbnail farms, content platforms), Garage for minimal-resource self-hosting with triple replication on heterogeneous hardware.

Self-hosting genuinely wins when you have sustained write-once-read-many traffic on your own uplink, an on-prem mandate, or an ops team already in place. Below roughly 10–20 TB without dedicated ops staff, managed tiers are cheap enough that running your own cluster rarely pays for itself in saved fees.

## Moving a bucket off S3: the rclone runbook

Because the API is identical, migration is mostly one command pointed at a different endpoint.

1. **Stand up the target.** Create a bucket and generate S3 keys. On Vultr, that's the console: pick Object Storage, choose a tier and location, generate your access and secret keys. 👉 [open Vultr's object storage page to create your first bucket](https://www.vultr.com/products/object-storage/?ref=7268283)
2. **Configure rclone** with the new endpoint — for Vultr, something like `ewr1.vultrobjects.com` as the host, plus your generated keys.
3. **Sync with checksums:** `rclone sync s3:bucket target:bucket --checksum`. Checksum mode avoids re-copying objects on reruns. Raise `--transfers` and `--checkers` to saturate your link.
4. **Cut reads over** by repointing your application's endpoint and keys, keeping the original S3 bucket live as a fallback for one billing cycle.
5. **Verify** with `rclone check` before you shut anything down.

Throughput reality check: a 12 TB move on a 1 Gbps line fits in a weekend, and the bottleneck is almost always your source egress cap rather than the destination. At 1 Gbps, 50 TB needs about a week of continuous transfer, so larger moves are mostly a planning exercise. While you're at it, test the three S3 features partial implementations tend to fumble: presigned URLs, versioning, and Object Lock. If your backups or compliance records depend on any of them, confirm support before migrating, and after.

## FAQ

### Will the AWS CLI and my SDKs work with these alternatives unchanged?

Yes, for the core API — PUT, GET, LIST, multipart upload, bucket policies. You change the endpoint and keys and keep your `aws s3` muscle memory. The long-tail features are where compatibility varies, so test presigned URLs, versioning, and lifecycle behavior with your actual code before the cutover.

### What's the cheapest option for read-heavy workloads?

It depends on the ratio. Small stored volume with heavy egress favors zero-egress providers like Cloudflare R2 or a flat-rate plan like Wasabi. Bundled-transfer tiers (Vultr includes 1–5 TB outbound depending on tier) work well when monthly egress stays inside or near the bundle. Bursty traffic that spikes far beyond 3× your stored volume lands in B2's $0.01/GB overage territory.

### Is MinIO free for commercial use?

For internal use, yes — AGPLv3 lets you run it on your own infrastructure. Embedding it in a product you sell, or wanting official paid support, requires a commercial license from MinIO, Inc. If that describes you, look at RustFS for a permissive-license alternative or budget for the commercial conversation.

### How long does a migration actually take?

The API being identical makes it bandwidth-bound. Under 12 TB on a decent link is a weekend project; 50 TB is about a week of continuous transfer at 1 Gbps, plus validation. The rclone `--checksum` flag matters here — it makes interrupted runs resumable without re-copying verified objects.

## Picking by situation

Rarely-read backups on a tight budget → B2's free-3× egress model or Wasabi's flat rate, whichever your read-back pattern fits. Heavy public delivery — media, static assets, downloads → a zero-egress provider, with your compute wherever it already lives. Storage plus compute in one console, a $6 floor, and credits to test with → that's Vultr's lane: pair a Starter or Scale bucket with a nearby compute instance and keep the whole stack on one bill. Your own rack, with ops time and an on-prem mandate → MinIO for maturity, RustFS if the AGPL license is a problem, Ceph when you also need block and file.

Whatever direction you lean, two things before you commit: run a 200 GB proof-of-concept with your real object sizes and access pattern, since per-operation fees and minimums never show up in a spreadsheet comparison. And check the current tier lineup when you sign up — 👉 [start with the $6 Starter tier and test it against your own data](https://www.vultr.com/products/object-storage/?ref=7268283) — because storage pricing shifts more often than most people expect.
