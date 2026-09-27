# Pull request: starter-kit files → develop

## Summary

Adds the KijaniKiosk DevOps foundation before the platform takes real customers.

- Delivery notes map this workflow to Flow, Feedback and Learning.
- Cloud model is PaaS-first for the app, SaaS for payments and email, IaaS only for the VPC.
- Region is `af-south-1` (Cape Town) with two availability zones.
- IAM role `kijani-order-receipt-writer` may `s3:PutObject` to one encrypted prefix only.
- Network diagram and routing notes keep the internet on public subnets; app and database stay private and exit through NAT.

## Test plan

- [ ] `starter-kit/` contains every deliverable file
- [ ] Branches `main`, `develop` and `feature/starter-kit-files` exist
- [ ] Network diagram shows public vs private subnets and the IGW / NAT routes
- [ ] IAM policy has no `*` actions and names one application task
- [ ] Region / AZ reasoning distinguishes a building failure from a regional outage
