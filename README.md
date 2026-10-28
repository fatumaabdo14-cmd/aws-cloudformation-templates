# AWS CloudFormation Templates

A set of CloudFormation templates covering the core building blocks of an AWS environment: networking, compute, scaling, identity and storage. Each template deploys on its own and can be used as a starting point for larger stacks.

## Templates

| Template | What it deploys |
|---|---|
| `vpc.yaml` | VPC (172.16.0.0/16) with public and private subnets across 2 AZs, internet gateway, a bastion host and private app instances. SSH to the app tier is allowed only from the bastion security group. |
| `ec2.yaml` | Same multi-AZ network plus two EC2 web servers behind an Application Load Balancer (listener + target group). |
| `asg.yaml` | Production-style web tier: 6 subnets across 2 AZs, NAT gateway, launch template, Auto Scaling group (min 1 / desired 2 / max 3) behind an ALB, and a CloudWatch CPU alarm (> 70%) driving a scaling policy. AMI resolved from SSM Parameter Store. |
| `ec2-instances.yaml` | Single EC2 instance with a security group. |
| `iam.yaml` | IAM user, group, managed policy and role, with the user added to the group. |
| `s3-bucket.yaml` | S3 bucket. |
| `vpcexercise.yaml` | Minimal VPC. |
| `index.html` | Sample page for S3 static website hosting. |
| `image-sharing-platform/` | Early version of the serverless image platform. The full project lives in [image-sharing-platform](https://github.com/fatumaabdo14-cmd/image-sharing-platform). |

## Deploy

```bash
aws cloudformation deploy \
  --template-file asg.yaml \
  --stack-name web-asg \
  --parameter-overrides KeyName=<your-key-pair> \
  --capabilities CAPABILITY_IAM
```

Validate a template before deploying:

```bash
aws cloudformation validate-template --template-body file://vpc.yaml
```

Tear down:

```bash
aws cloudformation delete-stack --stack-name web-asg
```

## Design notes

- **Public / private split.** Load balancers and the bastion sit in public subnets; application instances run in private subnets with no direct inbound internet access.
- **Security groups reference each other** rather than IP ranges, so access follows the resource (for example, app instances accept SSH only from the bastion's security group).
- **Scaling on demand.** The Auto Scaling group adds capacity when average CPU crosses 70% and keeps at least one healthy instance at all times.
- **No hardcoded AMIs.** The latest Amazon Linux AMI is looked up from SSM Parameter Store at deploy time, so the template keeps working across regions and AMI updates.

---

**Fatuma Abdo** · Cloud Engineer · [LinkedIn](https://www.linkedin.com/in/fatuma-abdo-3b729a3a4)
