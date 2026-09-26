# Region and availability zones

## Region choice: `af-south-1` (Cape Town)

KijaniKiosk’s first customers are in Kenya. AWS Africa (Cape Town) is the closest AWS region with a full set of the services this starter kit uses: VPC, ALB, NAT Gateway, ECS/Fargate or Elastic Beanstalk, RDS and S3.

| Option | Why it is not first |
|---|---|
| `eu-west-1` (Ireland) | Mature and cheap, but every checkout round-trip crosses Europe |
| `me-south-1` (Bahrain) | Closer than Ireland for some routes, weaker service catalogue for this stack |
| `us-east-1` | Default in many tutorials; worst latency for Nairobi users |

Latency is not a vanity metric. A kiosk attendant on a mid-range phone will abandon a three-second stock lookup. Putting compute in Cape Town, then caching the storefront at the edge, is the honest first design.

If a Kenyan AWS region opens later, we can add it. We do not wait for it.

## What a region gives us — and what it does not

A **region** is a cluster of data centres in one geography. It has its own copy of IAM, its own S3 control plane and its own network.

A region **does not** survive:

- A long fibre cut between Kenya and South Africa
- A regional control-plane event
- A legal requirement to keep a second copy of data in another country

Those risks need a later disaster-recovery region, not more subnets in Cape Town.

## Availability zones

An **availability zone** is an isolated site inside the region. Zones have separate power and network. A fire or a cooling failure in one AZ should not take the others down.

KijaniKiosk will run across **two AZs** from day one.

```
af-south-1
├── af-south-1a
│   ├── public subnet  10.0.1.0/24   ALB node, NAT
│   └── private subnet 10.0.2.0/24   app tasks, RDS primary
└── af-south-1b
    ├── public subnet  10.0.3.0/24   ALB node, NAT
    └── private subnet 10.0.4.0/24   app tasks, RDS standby
```

Two AZs is the smallest design that makes “multi-AZ” true. One AZ with two subnets is still one building.

## Reliability reasoning

**Application**

The load balancer is regional. It has a node in each public subnet. If AZ-a fails, the balancer stops sending traffic there. PaaS tasks in AZ-b keep serving.

**Database**

RDS (or equivalent) runs Multi-AZ: a synchronous standby in the second AZ. Failover is a DNS change the application does not manage. We accept a short interruption. We do not accept “restore from last night’s backup” as the HA plan.

**NAT**

Each public subnet has its own NAT Gateway. Sharing one NAT in AZ-a would mean a private task in AZ-b dies on the internet path when AZ-a dies. That is a hidden single point of failure.

**What we do not claim**

Multi-AZ is not a backup. Multi-AZ is not a second region. Multi-AZ is not a substitute for tested restores. Backups still go to S3 with a lifecycle policy. Restore is still rehearsed.

## If the platform grows

1. Add a third AZ only if the provider offers it and the data-plane cost is justified.
2. Add `eu-west-1` as a warm standby for disaster recovery, with infrastructure as code so the VPC is not drawn twice by hand.
3. Put static storefront assets on a CDN so Nairobi users are not fetching HTML from Cape Town on every tap.

The first of those is capacity. The second is survival. The third is experience. They are not the same ticket.
