---
title: "Configure MongoDB Atlas"
date: 2026-07-19
weight: 1
chapter: false
pre: " <b> 5.6.1. </b> "
---


## Objectives

Allow Amazon ECS Fargate to connect to MongoDB Atlas through the NAT Gateway, restrict access through the IP Access List, and manage the connection string with AWS Secrets Manager.

## Implementation steps

1. Sign in to MongoDB Atlas and confirm that the Cluster status is **Available**.

2. Create a dedicated Database User for the BravelSport application with minimum required permissions, such as `Read and Write to Any Database` or permissions appropriate for the application.

3. Go to:

```text
Network Access → IP Access List
```

4. Add the Elastic IP of the NAT Gateway in this format:

```text
<NAT_GATEWAY_ELASTIC_IP>/32
```

Allow only the public IP address of the NAT Gateway to access MongoDB Atlas.

5. After testing is complete, remove the following rule:

```text
0.0.0.0/0
```

so that access is not allowed from the entire Internet.

6. Retain administrator IP addresses only when they are actually required.

7. Create the MongoDB Connection String.

Example:

```text
mongodb+srv://<username>:<password>@cluster.mongodb.net/<database>
```

Do not include the Connection String or password in the workshop documentation.

8. Store the following variable:

```text
MONGO_URI
```

in:

```text
AWS Secrets Manager
```

Then reference this secret in the ECS Task Definition.

9. Register a new Task Definition Revision and update the ECS Service so that the backend uses the new Connection String.

<!-- INSERT IMAGE HERE:
Capture MongoDB Atlas → Network Access → IP Access List.

The screenshot should show:
- NAT Gateway Elastic IP (/32)
- Active status

Mask:
- Full public IP address
- Project ID
- Username
- Connection String
-->

![MongoDB Atlas IP Access List](/images/5-Workshop/5.6-Data-and-media/configure-mongodb-atlas.jpg)




## Verification

- MongoDB Atlas allows access only from the Elastic IP of the NAT Gateway.
- The `0.0.0.0/0` rule has been removed after testing.
- The ECS Service was successfully updated with the new Task Definition Revision.
- The backend connects successfully to MongoDB Atlas.
- The API can read and write data normally.
- CloudWatch Logs does not contain the Connection String or password.

## Common issues

| Issue | Resolution |
|---|---|
| Server selection timeout | Check the NAT Gateway Elastic IP, Route Table, NAT Gateway, and Connection String |
| Authentication failed | Check the Database User, password, `authSource`, and URL encoding |
| ECS cannot read the secret | Check `ecsTaskExecutionRole` and `secretsmanager:GetSecretValue` permission |
| The backend cannot connect to Atlas | Check the IP Access List and Security Group outbound rules |
| The Connection String appears in logs | Remove sensitive logging, update the Secret, and redeploy the ECS Service |

## Common issues

| Issue | Resolution |
|---|---|
| Server selection timeout | Check the EIP, NAT Gateway, DNS, and connection string |
| Authentication failed | Check the database user, `authSource`, and URL encoding |
| The secret is not injected | Check the execution role and secret ARN or name |
| The URI appears in logs | Correct the logging, rotate the secret, and handle logs according to policy |

## Summary

ECS is connected to MongoDB Atlas through the fixed outbound address of the NAT Gateway.



## Navigation

- Previous section: [5.6. Database and media](../)
- Next section: [5.6.2. Create S3 Media](../5.6.2-Create-media-bucket/)
