# DevOps delivery notes

KijaniKiosk is a last-mile retail platform. Shoppers and kiosk attendants will place orders, pay with mobile money, and check stock. This starter kit exists so those flows are designed before the first customer is live.

The Three Ways of DevOps shape how this repository was produced.

## Flow

Work moves in one direction: design on a feature branch, review on a pull request, then integrate.

| Branch | Role |
|---|---|
| `main` | Stable documentation that a new engineer can trust |
| `develop` | Integration line for accepted starter-kit work |
| `feature/starter-kit-files` | Isolated place to write this blueprint |

Nothing is committed straight to `main`. Cloud-model, region, IAM and network files travel together on the feature branch so a reviewer sees one coherent design, not five disconnected edits.

That is Flow: small batch, one path, no silent changes on the production branch.

## Feedback

The pull request from `feature/starter-kit-files` into `develop` is the feedback loop.

The PR description states:

- which cloud model we chose and why we rejected the other two
- which region and how many availability zones
- which IAM action the application is allowed to perform
- how public and private subnets route traffic

A reviewer can reject a file without blocking the rest of the argument. Comments stay on the PR, not in chat. That is faster than discovering a wide-open IAM policy after the first environment is built.

## Learning

This kit records decisions, not only diagrams.

If Cape Town (`af-south-1`) later proves too far from Nairobi customers, `regions-azs.md` already lists the fallback (a second region and CloudFront). If the receipt-upload role is ever asked to list every bucket, `iam-least-privilege.md` says why that request should be refused.

The reflection below is part of Learning: we write down the shortcuts we almost took so the next engineer does not take them.

## How this maps to the week

- Git collaboration is the branch model and the pull request, not a single commit on `main`.
- Cloud reasoning is in `cloud-model.md`.
- Reliability reasoning is in `regions-azs.md`.
- Least privilege is one named task in `iam-least-privilege.md`.
- Network segmentation is the diagram plus the routing notes.

## Reflection

**Where I was tempted to take shortcuts**

Putting every file on `main` would have been faster. Drawing a single subnet “for now” would have been faster. Giving the application `s3:*` would have been faster. Each of those hides a production incident: an unreviewed change, a database that is reachable from the internet, or a leaked key that can wipe every bucket.

**Which architectural decision required the most reasoning**

Region versus availability zone. A region is a geography. An AZ is an isolated data centre inside that geography. Multi-AZ in Cape Town protects us from one building failing. It does not protect us from a country-wide fibre cut back to Kenya. Those are different risks and they need different answers.

**If the platform grows significantly, what I would improve first**

Add a second region for disaster recovery and put the storefront behind a CDN. The next improvement is automated checks on the pull request (policy lint for IAM, a diagram review checklist) so Feedback is not only a human reading Markdown.
