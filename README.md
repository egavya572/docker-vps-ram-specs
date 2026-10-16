# best vps for docker: exact RAM specs for Compose and k3s, the KVM vs OpenVZ trap, and annual plans from $10.99/year

Most people searching for the best VPS for Docker are running into one of two problems: containers getting killed with exit code 137 on an underpowered box, or monthly bills that grow faster than the workloads. Both have concrete fixes. Docker needs dedicated RAM, KVM virtualization, and reasonably fast disk. Once those boxes are ticked, an annual-billing budget provider undercuts the big clouds by a wide margin — RackNerd's current promo KVM lineup starts at $10.99/year for 1 GB of RAM, and the 2.5 GB plan at $20.69/year runs a realistic Compose stack (app + PostgreSQL + Redis + reverse proxy) with headroom.

> **Quick picks**
> - Running a 2–4 service Compose stack: **RackNerd KVM 2GB** — 2 vCPU, 2.5 GB RAM, 40 GB SSD, $20.69/year, renews at the same rate.
> - Adding monitoring, a busier database, or single-node k3s: **KVM 4GB** — 4 vCPU, 5 GB RAM, 80 GB SSD, $39.99/year.
> - Need monthly billing or EU locations: Hetzner's CX22 (2 vCPU / 4 GB / 40 GB NVMe, roughly €3.79/month in the EU) is the usual community pick — you pay more per year, but you can leave any month.
>
> 👉 [check RackNerd's current KVM VPS promotions](https://bit.ly/my-rn)

## What a good Docker VPS actually needs

Three things decide whether containers run well: enough RAM, KVM virtualization, and fast disk. CPU matters less than beginners assume for typical self-hosted stacks — a reverse proxy, an app, a small database, and a Redis instance barely tickle two cores.

RAM is where people get burned. Every container ships with overhead, and the Linux kernel enforces memory limits through cgroups. When your containers collectively exceed the ceiling, the kernel's OOM killer terminates a process and Docker reports exit code 137. On shared hosting this happens repeatedly because memory is pooled across tenants; on a VPS with a dedicated allocation, it only happens if you genuinely overcommitted. The ranges below line up with what recent 2026 comparisons and host documentation converge on:

| Workload | Min RAM | Comfortable RAM | vCPU | Storage |
| --- | --- | --- | --- | --- |
| Single container (dev/test) | 1 GB | 2 GB | 1 | 20 GB SSD |
| Compose, 2–4 services | 2 GB | 4 GB | 2 | 40 GB SSD |
| Compose, 5+ services / monitoring | 4 GB | 8 GB | 4 | 80 GB SSD |
| k3s single node | 4 GB | 8 GB | 2 | 60 GB SSD |
| k3s, control plane + 1 worker | 8 GB total | 12–16 GB total | 4 total | 100 GB total |

Disk speed is the sleeper factor. Extracting a 1 GB image takes a couple of seconds on NVMe versus 15–25 seconds on SATA SSD, and if you ever run k3s, etcd writes synchronously to disk — a slow volume makes pod scheduling lag and `kubectl` time out. Local SSD with low fsync latency is what you want; network-attached block storage is where clusters get flaky.

## RackNerd's KVM plans and pricing

RackNerd's current promotional KVM lineup runs from $8.99 to $136.99 per year. Every plan is KVM-virtualized with full root access, billed annually, and renews at the same promotional rate — a policy RackNerd advertises openly, and one of the main reasons budget VPS communities keep recommending them.

| Plan | vCPU / RAM | SSD storage | Billing | Price | Order |
| --- | --- | --- | --- | --- | --- |
| KVM 768MB | 1 vCPU / 768 MB | 10 GB | Annual | $8.99/yr | [Order →](https://bit.ly/my-rn) |
| KVM 1GB | 1 vCPU / 1 GB | 17 GB | Annual | $10.99/yr | [Order →](https://bit.ly/my-rn) |
| KVM 2GB | 2 vCPU / 2.5 GB | 40 GB | Annual | $20.69/yr | [Order →](https://bit.ly/my-rn) |
| KVM 3GB | 3 vCPU / 3.5 GB | 60 GB | Annual | $31.19/yr | [Order →](https://bit.ly/my-rn) |
| KVM 4GB | 4 vCPU / 5 GB | 80 GB | Annual | $39.99/yr | [Order →](https://bit.ly/my-rn) |
| KVM 6GB | 5 vCPU / 6 GB | 100 GB | Annual | $52.99/yr | [Order →](https://bit.ly/my-rn) |
| KVM 8GB | 6 vCPU / 8 GB | 150 GB | Annual | $68.99/yr | [Order →](https://bit.ly/my-rn) |
| KVM 12GB | 8 vCPU / 12 GB | 200 GB | Annual | $103.99/yr | [Order →](https://bit.ly/my-rn) |
| KVM 16GB | 10 vCPU / 16 GB | 250 GB | Annual | $136.99/yr | [Order →](https://bit.ly/my-rn) |

Two details worth noticing. First, the plans ship with bonus RAM relative to their names — the "2 GB" plan actually has 2.5 GB, the "3 GB" has 3.5 GB, and the "4 GB" has a full 5 GB. That extra half-gig is exactly the headroom a Compose stack needs before OOM becomes a theme. Second, monthly transfer allowances scale with the plan, from under 1 TB on the smallest tiers to several TB on the larger ones; check the exact figure for your chosen configuration before ordering. RackNerd also runs a separate monthly-billed standard lineup and periodic flash deals, so the table above covers the promotional plans most people actually buy.

## Which plan fits which Docker workload

Match the plan to the stack, and the decision gets boring — which is what you want.

- **One or two light containers** (a reverse proxy plus a single app): the KVM 1GB plan at $10.99/year works. Add swap, keep expectations modest, and don't put a database under real load on it.
- **A typical Compose stack** — app, PostgreSQL, Redis, reverse proxy: the KVM 2GB plan at $20.69/year is the default answer. Steady-state usage on a stack like this usually lands around 1.5–2.5 GB, so 2.5 GB fits with a little room.
- **Database-heavy setups or a monitoring stack** (Prometheus and Grafana are RAM-hungry for what they do): jump to the KVM 4GB plan at $39.99/year. The 80 GB disk matters here too, because Prometheus data and accumulated image layers eat storage faster than you'd expect.
- **Single-node k3s**: the control plane alone wants about 1 GB before you run anything useful, and a handful of workloads (web app, Postgres, cert-manager, ingress) settles around 2–3 GB steady-state. KVM 4GB is the floor, KVM 6GB at $52.99/year is comfortable.
- **Multi-node k3s**: two KVM 4GB instances split across control plane and worker, or a single KVM 8GB if you just want to experiment.

What this lineup is bad for: full kubeadm Kubernetes with HA control planes, and anything expecting dedicated CPU performance. This is budget shared-core hosting, priced accordingly.

👉 [view RackNerd's current plans and data center options](https://bit.ly/my-rn)

## What $20.69 a year buys you elsewhere

On pure price-per-RAM, RackNerd's annual plans undercut the popular monthly hosts by half or more — and against the big clouds, by an order of magnitude. Here's the comparison at the specs most Docker users actually need:

| Host | Comparable plan | Price | Billing | Notes |
| --- | --- | --- | --- | --- |
| RackNerd KVM 2GB | 2 vCPU / 2.5 GB / 40 GB SSD | $20.69/yr (~$1.72/mo) | Annual | Same-rate renewals; US + Amsterdam DCs |
| Hetzner CX22 | 2 vCPU / 4 GB / 40 GB NVMe | ~€3.79/mo EU, ~$4.15/mo US | Monthly | Strong EU latency; per-TB traffic allowance, then per-GB |
| Contabo Core VPS 4 | 4 vCPU / 8 GB / 200 GB SSD | €5.24/mo | Monthly (12-mo pricing) | Flat-rate traffic; ~1.3k IOPS caps and CPU-steal reports documented on budget lines |
| DigitalOcean | 2 vCPU / 4 GB / 80 GB SSD | $24/mo | Hourly/monthly | Best docs and ecosystem; priciest per GB by far |

Run the annual math and the gap gets stark. Hetzner's CX22 costs roughly €45/year. DigitalOcean's equivalent 4 GB droplet runs about $288/year. RackNerd's KVM 4GB — which has more RAM than both — costs $39.99/year. For a self-hosted stack you plan to keep running, that difference compounds fast.

One nuance the monthly-price tables hide: at the 8 GB tier, Contabo and RackNerd converge. Contabo's Core VPS 4 (8 GB RAM, 200 GB SSD) at €5.24/month comes to roughly €63/year, against $68.99/year for RackNerd's KVM 8GB. At that point price stops being the deciding factor — Contabo wins on disk size and monthly flexibility, RackNerd wins on renewal predictability and fixed transfer allowances without fair-use ambiguity, and Contabo's documented IOPS caps and CPU-steal complaints are a real consideration for database containers.

Where the monthly hosts genuinely earn their premium: EU data centers with low latency to European users, hourly or monthly billing for short-lived experiments, and mature ecosystem tooling (a community Terraform module makes K3s on Hetzner nearly one-command to deploy). If you're in Berlin serving Berlin, paying 2–3x more per year for Hetzner is a defensible choice. If you're serving a global or US audience and the box will run for years, the annual plans win on math alone.

## The KVM vs OpenVZ check

Docker needs kernel-level features — namespaces, cgroups, overlay filesystems — and only KVM-based VPSes provide them reliably. RackNerd uses KVM across its entire lineup, so this is a non-issue there. The check matters when you shop elsewhere. OpenVZ shares the host kernel with every tenant on the machine: custom kernel modules (WireGuard's in-kernel mode, eBPF-based networking like Cilium) won't load, cgroup memory limits inside containers behave unpredictably, and nested virtualization for KIND, Minikube, or k3d is flatly impossible. A $2/month "VPS" is almost certainly OpenVZ. Walk away, no matter how good the RAM numbers look.

## Five things to check before ordering

1. **Data center location.** RackNerd has US data centers (Dallas, Los Angeles, San Jose, Chicago, New York, Seattle) plus Amsterdam, but the cheapest promotional tiers are usually limited to a subset of locations. Latency from Europe to a US box is 90–120ms — fine for a personal dashboard, annoying for a web app with EU users.
2. **Disk headroom.** Container images pile up. Run `docker system prune` regularly or the 17 GB disk on the KVM 1GB plan fills within months. If your images and volumes are the growing part of the workload, step up a tier for the disk, not the RAM.
3. **Transfer allowance.** Fixed monthly allotments are generous relative to the price, but a CI pipeline pulling multi-GB images plus production traffic adds up. Sanity-check your expected monthly volume against the plan's figure.
4. **Billing commitment.** Promo plans are annual and renew at the same rate, which is great for year two and a real commitment for month two. If there's a decent chance you'll abandon the project in eight weeks, a monthly-billed host is the better tool.
5. **Terms and support scope.** RackNerd is unmanaged — support covers the hardware, network, and virtualization layer, not your Docker configuration. Read the refund terms in the TOS before a larger annual spend, and note that OS reinstalls are self-service from the client area.

👉 [order the KVM 2GB plan at $20.69/year](https://bit.ly/my-rn)

## FAQ

**Can you run Docker on a 1 GB VPS?**
Yes, for one or two light containers — add swap and keep the database off it. A Compose stack with app, database, and cache wants 2 GB minimum, and the 2.5 GB RackNerd plan exists precisely for that middle ground.

**Is RackNerd usable for Kubernetes?**
For single-node k3s, yes — 4 GB or more, and the local SSD storage handles etcd's synchronous writes better than network block storage would. Full kubeadm clusters with HA control planes are a poor fit for any budget VPS; k3s is the CNCF-certified lightweight distribution that ships as a single binary and includes Traefik and CoreDNS by default.

**Do RackNerd promo prices increase on renewal?**
No — promotional rates renew at the same price for the life of the service. The lineup itself rotates, though, so a specific configuration can disappear between visits; if the plan you want is listed, that's the moment to grab it.

## So which one should you actually get?

If you're pinching every dollar and running one or two static containers, the KVM 1GB plan at $10.99/year does the job. For the standard self-hosted Compose stack most readers are building, the KVM 2GB at $20.69/year is the pick — it's the cheapest plan that fits a real stack with margin, and it costs less than two months of the equivalent Hetzner plan. Add monitoring or a busy database, take the KVM 4GB at $39.99/year and stop thinking about it.

If you need EU latency, monthly flexibility, or plan to rebuild the environment every few weeks, a monthly-billed host like Hetzner or DigitalOcean is genuinely the better fit — RackNerd's advantage is the total cost of a box that stays up for years, and that advantage evaporates on short-lived workloads.

Before clicking order: pick the data center closest to your actual users, note the plan's transfer allowance, and keep your Compose files in version control so a rebuild takes minutes. 👉 [lock in a RackNerd annual plan while the current promo lineup is live](https://bit.ly/my-rn)
