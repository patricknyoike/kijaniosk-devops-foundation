# Cloud service model

## Decision

KijaniKiosk will run as a **PaaS-first** platform, with **SaaS** for payments and email, and **IaaS** only for the virtual network we must own.

We are not choosing one model for the whole company. We are choosing the right model for each job.

## What KijaniKiosk must do

- Serve a web and mobile storefront
- Accept M-Pesa and card payments
- Hold product, price and order data
- Let a kiosk attendant see only their shop’s stock
- Survive a single data-centre failure without a full outage

The team is small. Time spent patching operating systems is time not spent on checkout and stock accuracy.

## Why not IaaS for the application

IaaS (virtual machines we build and patch ourselves) gives full control of the guest OS, the runtime and the disk.

That control is a cost:

- We become responsible for OS patches, SSH hardening and capacity planning.
- A missed kernel update is our incident, not the cloud provider’s.
- Horizontal scale means we write the automation.

IaaS is the right place for the **VPC, subnets, route tables and NAT**. Those are infrastructure objects, not application servers. The application itself should not start life as a fleet of unmanaged VMs.

## Why not SaaS for the core application

SaaS (a vendor’s finished product) is correct for:

- Email delivery (for example Amazon SES or a transactional mail provider)
- Payments (M-Pesa Daraja, or a payment aggregator)
- Source control and issue tracking (GitHub)

It is the wrong place for the KijaniKiosk catalogue and order service. Those are our product. A pure SaaS storefront would cap how we model kiosk stock, split tenders, or later add a warehouse. We would also be unable to put that data in a private subnet we control.

## Why PaaS for the application

PaaS (a managed runtime such as AWS Elastic Beanstalk, ECS Fargate, or App Runner) means:

- We ship a container or a build artefact.
- The platform schedules it, restarts it, and sits it behind a load balancer.
- We still choose the VPC, the IAM role and the database engine.
- We do not SSH in to apply security patches on the host.

That matches a team that must move fast and still own security boundaries.

| Workload | Model | Reason |
|---|---|---|
| Storefront and API | PaaS | We own the code; the cloud owns the hosts |
| PostgreSQL | Managed PaaS (RDS or Cloud SQL) | Multi-AZ failover without us running Postgres HA |
| Object storage for receipts | Managed (S3) | Durable storage with a narrow IAM policy |
| Payments and email | SaaS | Regulated, specialised, not our core |
| VPC, subnets, NAT, routes | IaaS networking | We must decide what is public |

## What we refuse at launch

- A single EC2 instance that is the website, the API and the database
- A database with a public IP “so we can connect from home”
- An application role with AdministratorAccess “until we tidy IAM later”

Those are IaaS habits that erase the PaaS benefit.

## How this will evolve

If a later component needs a special kernel module or a hardware token, that component can move to IaaS. The default stays PaaS. Growth should add managed services, not more pets.
