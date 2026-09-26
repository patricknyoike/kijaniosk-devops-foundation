# KijaniKiosk DevOps Foundation

Technical blueprint for the KijaniKiosk online platform. This repository is the Week 2 DevOps starter kit: how we work in Git, which cloud model we use, where the system runs, who may do what, and how the network is segmented.

The graded work lives in [`starter-kit/`](starter-kit/).

| File | Purpose |
|---|---|
| [starter-kit/delivery-notes.md](starter-kit/delivery-notes.md) | Flow, Feedback and Learning in this repo’s workflow |
| [starter-kit/cloud-model.md](starter-kit/cloud-model.md) | IaaS / PaaS / SaaS choice and why |
| [starter-kit/regions-azs.md](starter-kit/regions-azs.md) | Region and multi-AZ reliability |
| [starter-kit/iam-least-privilege.md](starter-kit/iam-least-privilege.md) | IAM role for one application task |
| [starter-kit/network-topology.png](starter-kit/network-topology.png) | Public / private subnet diagram |
| [starter-kit/network-routing.md](starter-kit/network-routing.md) | Routing logic that matches the diagram |

## Branch model

- `main` — production-ready documentation
- `develop` — integration branch
- `feature/starter-kit-files` — this starter kit, merged through a pull request

## Publish to GitHub (required for Canvas)

GitHub CLI is not installed on this machine. After you set `user.name` and `user.email` locally, create the remote and the pull request:

```bash
cd C:\xampp\htdocs\devops015
git remote add origin https://github.com/YOUR_USER/kijaniosk-devops-foundation.git
git push -u origin main
git push -u origin develop
git push -u origin feature/starter-kit-files
```

On GitHub: open a Pull Request from `feature/starter-kit-files` into `develop`, paste the description in `starter-kit/pr-description.md`, then merge it.
