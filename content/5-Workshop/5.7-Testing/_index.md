---
title: "End-to-end testing"
date: 2026-07-19
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---


## Objectives

Verify the complete BravelSport system flow from the user to CloudFront, the Application Load Balancer, ECS Fargate, MongoDB Atlas, Amazon S3, and Amazon CloudWatch.

Testing must be based on actual responses, system logs, and supporting screenshots. Do not mark an item as **PASS** without appropriate evidence.

## Implementation steps

### 1. Record the tested version

Record the details of the version under test so that they can be referenced when troubleshooting or performing a rollback.

| Information | Value |
|---|---|
| Test date | `22/07/2026` |
| Domain | `bravelsport.com` |
| AWS Region | `ap-southeast-1` |
| ECS Cluster | `web-ban-ao-cluster` |
| ECS Service | `web-ban-ao-backend-service` |
| Task Definition | `web-ban-ao-backend-task:18` |
| Container image | `web-ban-ao-backend:latest` |
| Container port | `3000` |
| Frontend bucket | `myapp-frontend-sport` |
| Media bucket | `bravel-uploads` |

The commit hash and image digest can be checked directly in GitHub and Amazon ECR at deployment time.

The public documentation does not need to include the full Account ID, ARN, or image digest.

### 2. Test DNS and HTTPS

Run a DNS check:

```bash
nslookup bravelsport.com
```


Test HTTPS:

```bash
curl -I https://bravelsport.com/
```

Expected results:

- Domain `bravelsport.com` resolves to the CloudFront Distribution.
- The connection uses HTTPS.
- The certificate is valid.
- The website returns a successful HTTP status.
- Users do not access S3, the ALB, or ECS directly.




### 3. Test the frontend and CloudFront

Open:

```text
https://bravelsport.com
```

Perform the following steps:

1. Confirm that the homepage loads successfully.
2. Confirm that CSS, JavaScript, and image files load normally.
3. Open a React Router route such as:

```text
https://bravelsport.com/admin
```

4. Reload the page in the browser.
5. Confirm that no `403` or `404` error appears.
6. Confirm that direct S3 access is denied.

Expected results:

- The React/Vite frontend is loaded through CloudFront.
- Static assets are retrieved from the private S3 bucket through OAC.
- The React Router fallback works.
- Block Public Access remains enabled for the S3 bucket.

### 4. Test AWS WAF

In the AWS Management Console, go to:

```text
AWS WAF & Shield → Protection packs (web ACLs)
```

Select the Web ACL associated with the CloudFront Distribution.

Perform the following checks:

- Confirm that the Web ACL scope is `CloudFront`.
- Confirm that the CloudFront Distribution is associated with the Web ACL.
- Open Sampled Requests.
- Confirm that valid requests are not blocked.
- Monitor the CloudWatch Metrics for the Web ACL.
- Change rules from `Count` to `Block` only after testing is complete.

Do not perform attacks against the real system. Use only safe test requests.

### 5. Test authentication and JWT

Use test accounts to perform the following checks:

1. Register a new account.
2. Sign in with valid credentials.
3. Call a protected API after signing in.
4. Confirm that the backend accepts a request containing a valid JWT.
5. Confirm that an invalid or expired JWT is rejected.
6. Sign out and confirm that the protected route is no longer accessible.

Expected results:

- Sign-in succeeds.
- The protected API returns data when a valid token is provided.
- An invalid or expired token returns an appropriate error, such as `401 Unauthorized`.
- Do not display the full JWT in screenshots or logs.

### 6. Test the REST API `/api/*`

Open Chrome DevTools:

```text
F12 → Network → Fetch/XHR
```

Perform actions in the interface such as:

- Sign in.
- Open the administration page.
- View the user list.
- View orders.
- View statistics.
- View sports facility or voucher data.

Inspect requests with the following path:

```text
/api/*
```

Expected results:

- Requests use the `bravelsport.com` domain.
- Requests do not call the ALB DNS name directly.
- Valid requests return HTTP status `200`.
- Requests retain the `/api/*` path.
- CloudFront forwards the request to the ALB.
- The ALB forwards the request to ECS Fargate.

<!-- INSERT IMAGE HERE:
Capture the BravelSport website together with Chrome DevTools Network.

The screenshot should show:
- Domain bravelsport.com
- A working administration page
- Fetch/XHR requests
- HTTP Status 200
- Requests such as profile, active, or stats

Mask:
- JWT
- Cookie
- Authorization Header
- Sensitive request payloads
- Sensitive user data
-->

![End-to-end browser testing](/images/5-Workshop/5.7-Testing/end-to-end-browser-test.jpg)

```text
https://bravelsport.com/admin
```

The API requests in DevTools Network return status code `200`, demonstrating that the frontend communicates successfully with the backend through CloudFront and the Application Load Balancer.

### 7. Test Socket.IO

BravelSport Socket.IO uses:

```text
Transport path: /socket.io/*
Namespace: /chat
```

Clearly distinguish the following:

- `/socket.io/*` is the transport path.
- `/chat` is the application namespace.
- Do not create a CloudFront behavior for `/chat`.

Perform the following test:

1. Sign in with two test accounts.
2. Open the chat feature in two browsers or two private windows.
3. Open Chrome DevTools.
4. Select:

```text
Network → Socket
```

5. Inspect the Socket.IO handshake.
6. Send a message from the first session.
7. Confirm that the second session receives the message without reloading.
8. Confirm that the connection uses the HTTPS or WSS domain.

Expected results:

- The handshake succeeds.
- The request uses `/socket.io/*`.
- Namespace `/chat` connects successfully.
- The backend processes the JWT handshake.
- Real-time messages are sent and received.
- CloudFront does not cache Socket.IO traffic.

### 8. Test MongoDB Atlas

Create or update test data using an appropriate system operation, such as:

- Create a user.
- Create an order.
- Create a voucher.
- Create a sports facility.
- Update a profile.
- Perform another operation appropriate for the system.

Then open MongoDB Atlas and confirm that the data was stored.

Expected results:

- ECS Fargate connects successfully to MongoDB Atlas.
- Read and write requests work.
- The IP Access List allows the NAT Gateway Elastic IP.
- `0.0.0.0/0` is not required.
- The connection string does not appear in screenshots or logs.

### 9. Test media on Amazon S3

Upload a valid file through the application interface.

Check the following:

1. The upload request succeeds.
2. A new object appears in this bucket:

```text
bravel-uploads
```

3. MongoDB stores the corresponding Object Key or URL.
4. The image or media file is displayed on the website.
5. Refresh the browser and confirm that the media is still available.
6. Replace the ECS task and confirm that the media is not lost.

Expected results:

- The file is not stored in the Fargate local filesystem.
- Media remains available after deployment.
- The ECS Task Role has the required S3 permissions.
- Block Public Access remains enabled for the bucket.
- Access Keys and Secret Keys are not used inside the container.

### 10. Test the Application Load Balancer

In the AWS Management Console, go to:

```text
EC2 → Target Groups
```

Select the backend Target Group.

Check the following:

- Target type is `IP`.
- Port is `3000`.
- The ECS task is registered.
- The target status is `Healthy`.
- Health check path is `/`.
- The health check returns status code `200`.

When the target is not healthy, check:

- Security Groups.
- Container port.
- Target Group port.
- Health check path.
- Route Table.
- Whether the backend listens on `0.0.0.0:3000`.

### 11. Test CloudWatch Logs

In the AWS Management Console, go to:

```text
CloudWatch → Logs → Log groups → /ecs/bravelsport-backend
```

Open the log stream for the running ECS task.

Check for:

- NestJS startup logs.
- API logs.
- Scheduled-task logs.
- MongoDB connection logs.
- Socket.IO logs, when recorded.
- No secrets, tokens, or connection strings in the logs.

<!-- INSERT IMAGE HERE:
Capture CloudWatch Logs during testing.

The screenshot should show:
- The ECS Log Group or Log Stream
- Logs from backend-container
- Log events being recorded
- Timestamp and message

Mask:
- JWT
- MongoDB URI
- Password
- User email
- Account ID
- Task ID when unnecessary
-->

![CloudWatch Logs during testing](/images/5-Workshop/5.7-Testing/cloudwatch-test-logs.jpg)


## Results matrix

| Item | Expected result | Evidence | Result |
|---|---|---|---|
| DNS | Domain points to CloudFront | Route 53 Hosted Zone and DNS test | PASS |
| HTTPS | Certificate is valid | Access to `https://bravelsport.com` | PASS |
| Frontend | Page loads and route reload succeeds | `end-to-end-browser-test.jpg` | PASS |
| CloudFront | Static frontend and API pass through the distribution | DevTools Network and response headers | PASS |
| S3 Frontend | Bucket is private and direct access is denied | S3 Permissions and OAC Bucket Policy | PASS |
| WAF | Valid requests are not blocked | Web ACL and Sampled Requests | PASS |
| Login/JWT | Login and protected API work | DevTools Network | PASS |
| REST API | `/api/*` returns the correct response | `end-to-end-browser-test.jpg` | PASS |
| Socket.IO | `/socket.io/*` and namespace `/chat` work | Network Socket and chat test | PASS |
| MongoDB Atlas | Backend reads and writes data successfully | Atlas and API response | PASS |
| Media | Media upload and display succeed | S3 bucket `bravel-uploads` and the interface | PASS |
| ALB | ECS target is Healthy | Target Group | PASS |
| ECS Service | Desired task and running task both equal 1 | ECS Service Overview | PASS |
| CloudWatch | Backend writes logs without exposing secrets | `cloudwatch-test-logs.jpg` | PASS |

## Verification

The testing chapter is complete when all of the following conditions are met:

- The domain works through Route 53 and CloudFront.
- HTTPS is valid.
- The frontend loads successfully.
- Reloading a React Router route does not fail.
- The REST API returns the correct responses.
- Authentication and JWT work.
- Socket.IO connects successfully.
- MongoDB Atlas reads and writes data normally.
- Media is stored in Amazon S3.
- The ECS Service has a running task.
- The Target Group is Healthy.
- CloudWatch Logs records backend logs.
- Screenshots and logs do not expose secrets or sensitive information.
- No unresolved critical errors remain.

## Common issues

| Issue | Cause | Resolution |
|---|---|---|
| CloudFront returns `502 Bad Gateway` | The ALB cannot connect to ECS or the origin protocol is incorrect | Check the ALB, Target Group, Security Groups, and origin protocol |
| CloudFront returns `403` | OAC, the Bucket Policy, or WAF is misconfigured | Check the S3 Bucket Policy, CloudFront OAC, and Sampled Requests |
| React Router reload returns `403/404` | SPA fallback is not configured | Configure a Custom Error Response or CloudFront Function |
| The API returns `403` from WAF | A Managed Rule creates a false positive | Review Sampled Requests and change the rule to `Count` for analysis |
| The API returns `401` | The JWT is missing, invalid, or expired | Check the Authorization Header and authentication logic |
| The JWT does not reach the backend | The Origin Request Policy does not forward the header | Forward `Authorization` to the ALB origin |
| Socket.IO polling fails | Caching is enabled or the query string is not forwarded | Use `CachingDisabled` and forward the query string |
| WebSocket upgrade fails | The CloudFront behavior or backend configuration is incorrect | Check `/socket.io/*`, the ALB, and the Socket.IO server |
| The namespace does not connect | The namespace is confused with the transport path | Keep path `/socket.io` and namespace `/chat` |
| MongoDB times out | The Atlas IP Access List or NAT Gateway is incorrect | Check the NAT EIP, Route Table, and Atlas Network Access |
| Upload fails | The ECS Task Role lacks S3 permissions | Check `BackendUploadS3Policy` and the Task Role |
| Media is lost after redeployment | The application still stores files in local `/uploads` | Migrate completely to the S3 Media Bucket |
| Target Group is Unhealthy | The health path, port, or Security Group is incorrect | Check `/`, port `3000`, and SG-ALB → SG-ECS |
| No logs appear in CloudWatch | The Log Group is incorrect or the Execution Role lacks permissions | Check `awslogs` and `AmazonECSTaskExecutionRolePolicy` |
| Logs contain secrets | The application logs environment variables or a sensitive error object | Remove sensitive logging, rotate the secret, and deploy a new revision |

## Summary

The end-to-end testing process verifies the complete BravelSport flow:

```text
User
  → Route 53
  → CloudFront
  → AWS WAF
  → S3 Frontend
```

For the API and Socket.IO:

```text
User
  → Route 53
  → CloudFront
  → AWS WAF
  → Application Load Balancer
  → ECS Fargate
  → MongoDB Atlas
```

For media:

```text
ECS Fargate
  → Amazon S3 Media Bucket
```

CloudWatch Logs records the activity of the backend container. API requests in the browser return successful status codes, demonstrating that the frontend, backend, and AWS services work together according to the target architecture.


- Previous section: [5.6. Database and media](../5.6-Data-and-media/)
- Next section: [5.8. Resource cleanup](../5.8-Cleanup/)
