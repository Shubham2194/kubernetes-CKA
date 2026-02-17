🚀 Securing SQS Access in EKS using IRSA (Production Story)

Recently, I implemented a secure access model between EKS Pods and SQS using IRSA (IAM Roles for Service Accounts) — and I want to share why this matters.

🎯 The Problem

We had:

Multiple FIFO SQS queues

SNS publishing messages to those queues

Backend pods consuming messages

But…

❌ Any IAM entity with permissions could access SQS
❌ No strict boundary between namespaces
❌ Risk of accidental or unauthorized access

In production, that’s not acceptable.

🧠 The Goal

Ensure that:

✅ Only pods using the prod ServiceAccount in backend namespace can consume SQS
✅ SNS continues working as producer
✅ No static AWS credentials in pods
✅ Everything managed via Terraform

🏗 Step-by-Step Implementation
1️⃣ Created IAM Policy for SQS + KMS

We created a policy allowing:

sqs:ReceiveMessage

sqs:DeleteMessage

sqs:GetQueueAttributes

sqs:GetQueueUrl

kms:Decrypt

kms:GenerateDataKey

Attached it to an IAM Role:

proview-prod-sqs

2️⃣ Created IRSA Role

Using EKS OIDC provider:

Action = "sts:AssumeRoleWithWebIdentity"
Condition = {
  StringEquals = {
    "system:serviceaccount:backend:prod"
  }
}


Now only pods using:

serviceAccountName: prod
namespace: backend


can assume this role.

3️⃣ Annotated Kubernetes ServiceAccount
serviceAccount:
  name: prod
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::ACCOUNT:role/proview-prod-sqs


Pods automatically received:

AWS_ROLE_ARN
AWS_WEB_IDENTITY_TOKEN_FILE


No secrets. No access keys. Secure by design.

4️⃣ Updated SQS Queue Policy (Critical Step)

Important realization:

If we restrict SQS only to IRSA role…

🚨 SNS will fail to publish.

Because SNS acts as a producer.

So we added:

{
  "Principal": {
    "Service": "sns.amazonaws.com"
  },
  "Action": "sqs:SendMessage"
}


And for consumer:

{
  "Principal": {
    "AWS": "arn:aws:iam::ACCOUNT:role/proview-prod-sqs"
  },
  "Action": [
    "sqs:ReceiveMessage",
    "sqs:DeleteMessage"
  ]
}


Now:

✅ SNS can publish
✅ Only prod pods can consume
❌ Other namespaces fail with NoCredentialsError

Exactly what we wanted.

🧪 Production Safety Practice

Before touching prod queues:

✔ Created dummy FIFO queue
✔ Deployed test consumer pod
✔ Validated SNS → SQS → Pod flow
✔ Tested failure scenario in another namespace

Never test security changes directly in production.

🔥 Key Lessons

Terraform replaces SQS policies entirely — it does NOT merge.

Importing resources doesn’t protect existing permissions.

Always account for producers (SNS) when locking down consumers.

IRSA eliminates static credentials completely.

Test with dummy infra before tightening prod.

💡 Why This Matters

This change:

Eliminated credential sprawl

Enforced namespace-level access control

Reduced blast radius

Made infra declarative and auditable

Security without slowing development.

If you're running EKS + SQS and still using static credentials inside pods…

It’s time to switch to IRSA.

#DevOps #AWS #EKS #Kubernetes #Terraform #CloudSecurity #IRSA #SQS #SNS #CloudArchitecture

If you want, I can:

Make a shorter version (more viral style)

Add a simple architecture diagram

Convert into carousel content

Add a stronger storytelling hook

Tell me your vibe — technical, leadership, or impact-focused? 🚀
