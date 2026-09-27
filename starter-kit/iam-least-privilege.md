# IAM least privilege

## The task

When a kiosk sale completes, the **order service** writes one PDF receipt to object storage.

That is the only production permission this role exists to grant.

Identity: IAM role `kijani-order-receipt-writer`  
Assumed by: the PaaS task role for the order service (ECS task role or Elastic Beanstalk instance profile)  
Not assumed by: developers, CI, the storefront, or a human “break-glass” user

## Why this is a separate role

If the storefront, the order API and a developer laptop share one access key, a leaked key can read the database snapshots, change IAM and delete receipts.

The order service needs to **create a new object** under one prefix in one bucket. It does not need to list the account, delete objects, or read another shop’s prefix.

## Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "WriteReceiptsOnly",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::kijani-kiosk-receipts-prod/receipts/${aws:PrincipalTag/kioskId}/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-server-side-encryption": "aws:kms"
        }
      }
    },
    {
      "Sid": "AllowKmsForThatBucket",
      "Effect": "Allow",
      "Action": [
        "kms:Encrypt",
        "kms:GenerateDataKey"
      ],
      "Resource": "arn:aws:kms:af-south-1:111122223333:key/kijani-receipts-key",
      "Condition": {
        "StringEquals": {
          "kms:ViaService": "s3.af-south-1.amazonaws.com"
        }
      }
    }
  ]
}
```

Replace the account ID and KMS key id when the account is created. Do not replace `PutObject` with `s3:*`.

## Trust policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs-tasks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Only the compute platform may assume this role. There is no long-lived access key on a laptop.

## What this policy refuses

| Request | Why it is denied |
|---|---|
| `s3:ListBucket` on the whole bucket | Stops a compromised task from enumerating every kiosk’s receipts |
| `s3:GetObject` | Writing a receipt is not the same as reading another shop’s receipt |
| `s3:DeleteObject` | A bug or an attacker cannot wipe the audit trail |
| `s3:*` on `*` | That is administrator access wearing an application name |
| `iam:PassRole` / `iam:CreateUser` | The app must not mint new identities |
| KMS use except via S3 in `af-south-1` | Stops the role being used as a general decryptor |

## How the kiosk boundary is enforced

The role is tagged with `kioskId`. The object key must start with `receipts/<that id>/`. A task serving kiosk `NBO-004` cannot write to `receipts/NBO-009/`.

If the runtime cannot supply that tag, the service writes to `receipts/unassigned/` only in the non-production account, never in production.

## How this will be reviewed

- The pull request that changes this policy must name the new action and the new resource.
- CI should reject `Action: "*"` and `Resource: "*"`.
- Access Advisor (or the equivalent) is checked after the first month. Unused actions come out.

Least privilege is not a slogan. It is a list of verbs and a list of objects, and the list stays short.
