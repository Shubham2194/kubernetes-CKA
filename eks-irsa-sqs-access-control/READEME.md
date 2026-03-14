<img width="1704" height="734" alt="image" src="https://github.com/user-attachments/assets/be594732-51bc-41c3-828f-80465477edf4" />


**🚀 Securing SQS Access in EKS using IRSA (Production Story)
**

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
**1️⃣ Created IAM Policy for SQS + KMS
**
We created a policy allowing:

```
sqs:ReceiveMessage
sqs:DeleteMessage
sqs:GetQueueAttributes
sqs:GetQueueUrl
kms:Decrypt
kms:GenerateDataKey
```

Attached it to an IAM Role:

```
prod-sqs
```


<img width="686" height="178" alt="image" src="https://github.com/user-attachments/assets/f5e11160-9449-4df9-868e-32c8987a5a9f" />


**2️⃣ Created IRSA Role
**
Using EKS OIDC provider:

```
Action = "sts:AssumeRoleWithWebIdentity"
Condition = {
  StringEquals = {
    "system:serviceaccount:backend:prod"
  }
}
```


Now only pods using:

```
serviceAccountName: prod
namespace: backend
```

can assume this role.


**3️⃣ Annotated Kubernetes ServiceAccount, add this in values.yaml in which Service account in getting created
**

```
serviceAccount:
  name: prod
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::ACCOUNT:role/prod-sqs
```


<img width="676" height="147" alt="image" src="https://github.com/user-attachments/assets/fe3347fa-ecb5-43c7-a3b4-18a8dfb60220" />


Pods automatically received:

```
AWS_ROLE_ARN
AWS_WEB_IDENTITY_TOKEN_FILE
```

No secrets. No access keys. Secure by design.

**4️⃣ Updated SQS Queue Policy (Critical Step)
**
Important realization:

If we restrict SQS only to IRSA role…

🚨 SNS will fail to publish.

Because SNS acts as a producer.

So we added:
```
{
  "Principal": {
    "Service": "sns.amazonaws.com"
  },
  "Action": "sqs:SendMessage"
}
```

And for consumer:

```
{
  "Principal": {
    "AWS": "arn:aws:iam::ACCOUNT:role/prod-sqs"
  },
  "Action": [
    "sqs:ReceiveMessage",
    "sqs:DeleteMessage"
  ]
}
```

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


terraform code :

SQS queue
```
resource "aws_sqs_queue" "dummy" {
  name                        = "dummy.fifo"
  fifo_queue                  = true
  content_based_deduplication = true

  tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}
```

SNS 
```
resource "aws_sns_topic" "dummy" {
  name                        = "dummy.fifo"
  fifo_topic                  = true
  content_based_deduplication = true
}
```

Subscribe Queue to Topic

```
resource "aws_sns_topic_subscription" "dummy_sub" {
  topic_arn = aws_sns_topic.dummy.arn
  protocol  = "sqs"
  endpoint  = aws_sqs_queue.dummy.arn
}
```


🚀 Attach Queue Policy (🔥 Most Important)

This is the secure part.

```
data "aws_caller_identity" "current" {}

resource "aws_sqs_queue_policy" "dummy_policy" {
  queue_url = aws_sqs_queue.dummy.id

  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [

      # ✅ Allow SNS to publish to SQS
      {
        Sid = "AllowSNSPublish",
        Effect = "Allow",
        Principal = {
          Service = "sns.amazonaws.com"
        },
        Action = "sqs:SendMessage",
        Resource = aws_sqs_queue.dummy.arn,
        Condition = {
          ArnEquals = {
            "aws:SourceArn" = aws_sns_topic.dummy.arn
          }
        }
      },

      # ✅ Allow IRSA role to consume
      {
        Sid = "AllowIRSAConsume",
        Effect = "Allow",
        Principal = {
          AWS = var.irsa_role_arn
        },
        Action = [
          "sqs:ReceiveMessage",
          "sqs:DeleteMessage",
          "sqs:GetQueueAttributes",
          "sqs:GetQueueUrl"
        ],
        Resource = aws_sqs_queue.dummy.arn
      }

    ]
  })
}

```


🧪 Test Flow
Publish:
aws sns publish \
  --topic-arn <dummy-topic-arn> \
  --message "hello" \
  --message-group-id test

You can also public from UI:

<img width="1607" height="604" alt="image" src="https://github.com/user-attachments/assets/838d17a1-f462-4c27-91ad-46f801a8ce4d" />


Watch pod:
kubectl logs -f deploy/sqs-worker -n backend

<img width="1186" height="234" alt="image" src="https://github.com/user-attachments/assets/c78730de-c9b9-43e6-a050-360d4a97b845" />



