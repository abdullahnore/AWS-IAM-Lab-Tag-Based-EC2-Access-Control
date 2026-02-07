
Objective

This lab demonstrates how AWS IAM policies can be designed to enforce least-privilege access using resource tags, a common real-world cloud security pattern.

Instead of granting broad EC2 permissions, access is restricted based on environment type (development vs production).

Lab Scenario

Two EC2 instances were created:

Production (Env=production)

Development (Env=development)

An IAM user represents an intern / junior engineer

Access is granted only to development resources

Production resources remain protected

This mirrors how enterprises safely onboard interns and junior staff.

Architecture

AWS Services Used

IAM (Users, Groups, Policies)

EC2

Resource Tags

Region

Asia Pacific (Mumbai)


<img width="922" height="323" alt="ec2 page" src="https://github.com/user-attachments/assets/6e284bb1-440a-44e3-8ef5-1aeefa2cd877" />

EC2 Setup

Two EC2 instances were launched:

prod-ec2 → tagged Env=production

dev-ec2 → tagged Env=development

Tags are used as the security control point, not instance names.



IAM Design
User & Group

IAM User: intern-ganesh

IAM Group: intern-lab

Permissions are assigned to the group, not directly to the user

This follows AWS best practices and scales cleanly.

<img width="688" height="300" alt="user-group-attach" src="https://github.com/user-attachments/assets/58a848be-f932-415b-833d-928d542a87d6" />

<img width="704" height="356" alt="creating-usersiam" src="https://github.com/user-attachments/assets/1379e4aa-64bf-4de3-84b4-6d7597e6e1ca" />
<img width="937" height="338" alt="addusser-togruop" src="https://github.com/user-attachments/assets/4c69c7cb-27e0-4b39-85be-86ce3a73070f" /><img width="937" height="347" alt="user-created" src="https://github.com/user-attachments/assets/c60a415d-9072-483b-9381-373a05772e56" />


Custom IAM Policy – Dev-limits
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Env": "development"
        }
      }
    },
    {
      "Effect": "Allow",
      "Action": "ec2:Describe*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": [
        "ec2:DeleteTags",
        "ec2:CreateTags"
      ],
      "Resource": "*"
    }
  ]
}

Policy Logic

Full EC2 access only for Env=development

Read-only visibility across EC2

Explicit deny prevents tag manipulation (no privilege escalation)


<img width="833" height="355" alt="policy-json" src="https://github.com/user-attachments/assets/c0113e58-98a1-4d2a-8c70-f0153ff466b5" />

Validation & Errors (Expected Behavior)
Production Instance

❌ ec2:StopInstances → Access Denied

Reason: instance tagged as production

S3 & Service Catalog

❌ Access denied due to no permissions assigned

These failures confirm the policy is working as intended.

<img width="926" height="382" alt="ganesh acces-check1" src="https://github.com/user-attachments/assets/a0eea564-a71c-442a-9b26-3c4573fa3b6e" />

<img width="735" height="377" alt="error-ec2" src="https://github.com/user-attachments/assets/201b2c60-b7dd-4ab8-86ac-e9f961362d3e" />

Security Takeaways

Tag-based IAM conditions enable fine-grained control

Explicit denies are critical for security

Groups > individual user permissions

Access-denied errors often mean good security, not misconfiguration
