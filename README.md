# GPU cloud pricing: what an H100, L40S or B200 costs per hour, which one your model needs, and what the $300 Vultr credit really covers (full plan table)

An H100 GPU-hour starts at $2.30 on Vultr. The same GPU-hour lists at $4.59 at Lambda, and Google Cloud bills its GPUs by the second. Those three facts explain most of the gap between a $40 experiment and a $400 one, and none of them show up in a pricing headline.

Below is Vultr's GPU lineup plan by plan, followed by the parts of the bill that sit outside the hourly rate: rounding, egress, idle time, and the difference between an advertised "starting at" figure and the on-demand rate you actually pay.

> **Quick pick:** A single **L40S at $1.67/hour** covers development work and inference for models up to roughly 13B parameters. For full fine-tuning of 70B or serving it at FP8, the **8-GPU HGX H100 node at a listed $2.30/GPU/hour** (about $18.40/hour for the node) is the realistic entry point. New accounts get a **$300 promotional credit** for eligible products, with conditions below. 👉 [See Vultr's GPU plans and launch an instance](https://bit.ly/VuLtr)

## The full Vultr GPU lineup

Vultr sells two shapes of GPU instance. Single-GPU and multi-GPU cloud instances run L40S, A40 and A100 PCIe cards. The H100 and B200 boards are sold only as 8-GPU HGX nodes, so the per-GPU price is a division of the node price, not something you can buy one of.

| Plan | GPU and memory | Per-GPU hourly | Sold as | Get it |
| --- | --- | --- | --- | --- |
| NVIDIA L40S | 48 GB GDDR6, Ada Lovelace | from $1.67 | 1, 2, 4 or 8 GPUs | [L40S plans](https://bit.ly/VuLtr) |
| NVIDIA A40 | 48 GB GDDR6 | from $1.71 | 1 or 4 GPUs | [A40 plans](https://bit.ly/VuLtr) |
| NVIDIA A100 PCIe | 80 GB | from $2.40 | 1 or 8 GPUs | [A100 PCIe plans](https://bit.ly/VuLtr) |
| NVIDIA HGX A100 SXM | 80 GB HBM2e per GPU | from $2.80 | 8-GPU node | [HGX A100 node](https://bit.ly/VuLtr) |
| NVIDIA HGX H100 SXM | 80 GB HBM3 per GPU | from $2.30 | 8-GPU node | [H100 plan details](https://www.vultr.com/products/cloud-gpu/nvidia-h100?ref=9738262-9J) |
| NVIDIA HGX B200 | 180 GB HBM3e per GPU, Blackwell | from $3.50 | 8-GPU node | [HGX B200 node](https://bit.ly/VuLtr) |

The L40S price holds at $1.67 per GPU whether you rent one card or eight, which makes it the only plan where scaling up doesn't change your unit economics.

Bundled compute and network for the single-GPU configurations:

| Plan | vCPU | RAM | Local storage | Bandwidth included |
| --- | --- | --- | --- | --- |
| L40S (1 GPU) | 16 | 180 GB | 1.20 TB | 10 TB |
| A40 (1 GPU) | 24 | 120 GB | 1.40 TB | 15 TB |
| A100 PCIe (1 GPU) | 12 | 120 GB | 1.40 TB | 10 TB |
| HGX H100 node (8 GPUs) | 216 total | 1914 GB total | — | — |
| HGX B200 node (8 GPUs) | 248 total | 2826 GB total | — | — |

## Is the "starting at" price the rate you'll actually pay?

Not always, and this is the single most expensive thing to get wrong before you budget.

Vultr's H100 page shows "Starting at $2.300/hour" for the HGX H100. That same figure appears in Vultr's bare metal section, where "starting from" is explicitly tied to pre-paid contract terms. Vultr's own public plan API returns the H100 node at $23.92/hour, which is $2.99 per GPU, with on-demand deployment flagged off, plus a preemptible rate of $18.40/hour ($2.30 per GPU). The B200 shows a wider gap: $3.50/GPU/hour on the product page against $68.00/hour for the node in the API ($8.50 per GPU), with a preemptible rate of $25.60/hour ($3.20 per GPU).

Read that as three different products sharing one headline number: preemptible capacity, contract capacity, and self-serve on-demand. Before you commit to a multi-week run, open the deploy screen and read the rate and the billing label on the configuration you selected. A 3x difference between the advertised and actual per-GPU rate turns a $3,000 training budget into $9,000.

Preemptible capacity is real money when your job can survive interruption. It is a bad deal for a fine-tune that checkpoints every six hours.

## What the hourly rate leaves out

Every GPU cloud quotes an hourly rate. Almost none of them quote the three line items that decide the invoice.

**Egress.** Moving checkpoints, datasets and inference responses out of a cloud is billed by volume at most providers, sometimes more than the GPU time itself. Lambda states plainly that it charges no egress fees, and UpCloud's private cloud GPU offering is zero-egress on single-tenant hardware in Finland. Vultr bundles 10 TB of bandwidth into its single-GPU L40S and A100 PCIe plans and 15 TB into the A40 plan, which covers a lot of inference traffic before overage becomes a line item. Check what happens past the bundle.

**Storage between sessions.** A 22 TiB SSD allocation attached to an idle instance is still an allocation. If your workflow is "train for two hours, analyze for two days," you are paying for the two days.

**Idle GPUs.** The number that reframes everything: a single L40S left running around the clock costs about **$1,220 a month** at $1.67/hour. An 8-GPU H100 node running continuously lands near **$13,400 a month**. Nobody intends to leave a node up for a month. Plenty of teams do it because shutting down and restarting felt like a hassle.

## Billing granularity decides what a short job really costs

Three providers, three answers to "what does 40 minutes cost?"

Google Cloud bills Compute Engine GPUs by the second. Lambda bills by the minute. UpCloud charges only for active compute time, so a powered-off server stops accruing GPU cost. If Vultr's H100 is priced per hour with rounding, a 40-minute run on the 8-GPU node bills as a full hour at $18.40 instead of the $12.27 the wall clock earned.

Run that job 20 times a week and the rounding alone costs about **$123 a week**, roughly $6,000 a year, for compute you never used. Short interactive jobs and hyperparameter sweeps are exactly where hourly rounding hurts, and exactly where per-minute or per-second billing pays for itself.

Three habits fix this regardless of provider:

1. Batch small experiments into one session instead of starting and stopping a node for each.
2. Snapshot your environment so a restart is minutes, not an afternoon of reinstalling CUDA.
3. Check whether your provider charges for stopped instances. Some do, for storage and IP addresses.

## Pick the card by memory first, price second

The most common overspend in GPU cloud is renting an H100 for a job an L40S handles, or renting an L40S for a model that doesn't fit. Work the problem in this order.

**Step 1: how much memory do the weights need?** At bf16, a model needs about 2 GB per billion parameters. At FP8, about 1 GB per billion.

- 7B at bf16: 14 GB of weights. With KV cache and activations, plan for 20–25 GB. An L4-class 24 GB card is tight; an L40S at 48 GB is comfortable.
- 13B at bf16: 26 GB. L40S 48 GB, done.
- 70B at bf16: 140 GB. Two H100 80 GB cards leave almost no room for context, so this means 4× A100 80 GB, or one B200 at 180 GB.
- 70B at FP8: about 70 GB. A single H100 80 GB fits with careful context limits; the B200 is the comfortable answer.
- 405B at FP8: about 405 GB, which means an 8-GPU H100 node (640 GB total) or 4× B200.

**Step 2: does the workload need FP8 or FP4?** The H100's Transformer Engine and FP8 support are why Vultr's own page cites up to 4x faster training and inference for transformer models compared with the previous generation. Blackwell cards go further: the B200 lists 9.00 PFLOPS of FP8 tensor performance, and the B300 adds 18.00 PFLOPS of FP4 for long-context and reasoning workloads. If your stack runs in FP8 or FP4, the older cards cost less per hour and finish later. Multiply it out before deciding.

**Step 3: check memory bandwidth for inference.** Token generation is bandwidth-bound. The L40S moves 864 GB/s, the H100 moves 3.35 TB/s, and the B200 moves 8.0 TB/s. Same model, same batch size, very different tokens per second.

**Step 4: only now compare hourly prices.** DigitalOcean's guidance is blunt and correct: prototypes and side projects can run on a T4-class card, and only large, complex workloads justify the top-end silicon. A 7B chatbot demo does not need an H100. A 70B production endpoint does.

## What Vultr adds beyond the card

Vultr runs 34 cloud data center locations across six continents, which matters when you are serving inference and want the GPU near the user rather than three hops away. Multi-node work goes through Vultr Clusters, connected with NVIDIA InfiniBand, provisioned from the console or the API, and schedulable with Slurm or Kubernetes. Vultr states that clusters on both Cloud GPU and Bare Metal need no reservation or special request, so an H100 node is a self-serve purchase rather than a sales call. The platform also carries ISO and SOC 2 compliance coverage, which is usually the first question a security review asks.

For teams that need to be running this week, that self-serve path is the practical difference between Vultr and clouds where an 8-GPU allocation means a quote and a wait.

## The $300 credit: what it covers and what it doesn't

Vultr advertises $300 in promotional credit for new accounts. It is a trial balance with an expiry, not a permanent discount. The credit applies to select products only, cannot be combined with other offers, and requires a verified payment method before it is released.

Do the arithmetic before you plan around it:

- One L40S at $1.67/hour: about **179 hours** of credit.
- One 8-GPU H100 node at the listed $18.40/hour: about **16 hours**.
- One 8-GPU B200 node at $28/hour: under **11 hours**, or about 4 hours if the node bills at the API's on-demand rate.

That is enough for a serious proof of concept and nowhere near enough for a month of training. After your first deploy, open the billing page and confirm the instance type you picked is actually drawing down the promo balance; some products are excluded, and you want to find that out on hour one rather than on the invoice. 👉 [Claim the credit and check current eligibility](https://bit.ly/VuLtr)

## When Vultr isn't the right call

Two situations where a different shape of purchase wins.

**Your workload tolerates interruption.** Spot and preemptible capacity runs 20–50% cheaper, and UpCloud's spot instances discount selected GPUs by up to 25%. The catch is documented and severe: when capacity is reclaimed, the instance is terminated, not paused. Batch inference, rendering, data preprocessing and anything that checkpoints properly belong here. A fine-tune that can't resume from disk does not.

**You're serving a small model to bursty traffic.** A dedicated L40S at $1.67/hour costs $1,220 a month whether you get one request or a million. If your 7B endpoint is idle most of the day, a shared or serverless GPU will beat a dedicated card on cost, and you give up predictable latency for it.

Everything else, from a single-card dev box to an InfiniBand-connected H100 cluster, is a reasonable fit for the plans above.

## The final decision rules

- **Prototyping, RAG pipelines, 7B–13B inference:** single L40S at $1.67/hour. Add the A40 instead if the workload is CPU-heavy, since it brings 24 vCPU against the L40S's 16.
- **Fine-tuning up to 13B, or 70B with LoRA:** A100 80 GB PCIe at $2.40/GPU/hour.
- **Full fine-tune of 70B, or serving 70B at FP8 with real context:** 8-GPU HGX H100 node at a listed $2.30/GPU/hour.
- **400B-class models or FP4 workloads:** HGX B200 node, after you confirm which rate you're being quoted.

Two things to verify on the deploy screen before you click: whether the price shown is on-demand, preemptible or contract, and whether your chosen instance type draws from the promo credit. Those two checks are worth more than any comparison table, including this one.

👉 [Start with Vultr's H100 plan and confirm the current per-GPU rate](https://www.vultr.com/products/cloud-gpu/nvidia-h100?ref=9738262-9J)
