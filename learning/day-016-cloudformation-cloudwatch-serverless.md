# Day 16 — CloudFormation, CloudWatch, and Serverless

## Learning objectives
- Explain Infrastructure as Code (IaC) and how CloudFormation manages AWS resources.
- Distinguish CloudWatch metrics, logs, alarms, and dashboards.
- Explain the roles of Amazon SNS and Amazon SQS.
- Describe AWS Lambda and Amazon API Gateway.
- Design a simple event-driven, serverless workflow and identify its security and cost considerations.

## 1. Infrastructure as Code (IaC)

Infrastructure as Code means describing infrastructure in version-controlled files instead of creating every resource manually through a console.

### AWS CloudFormation
CloudFormation uses templates to describe resources and provisions them as stacks.

Key concepts:
- **Template:** A YAML or JSON description of the desired resources.
- **Resource:** An AWS object declared in a template.
- **Parameter:** An input value supplied when creating or updating a stack.
- **Output:** A value exposed after deployment, such as a resource identifier.
- **Stack:** A managed collection of resources created from a template.
- **Change set:** A preview of proposed changes before applying an update.

Benefits include repeatability, reviewable changes, consistency, and easier teardown. Templates should be reviewed carefully: updates or deletion can affect real resources and data. Use least privilege and avoid hard-coding secrets.

### Safe IaC workflow
1. Write a small template.
2. Validate its syntax.
3. Review the resources and permissions it declares.
4. Create a stack in a sandbox account.
5. Inspect stack events and outputs.
6. Update the template and review a change set when applicable.
7. Delete the test stack only after checking whether retained resources or data need to be preserved.

## 2. Amazon CloudWatch

CloudWatch provides monitoring and observability capabilities for AWS resources and applications.

- **Metrics:** Time-series measurements such as CPU utilization, request count, errors, or custom application measurements.
- **Logs:** Log events organized into log groups and streams.
- **Alarms:** Evaluate metrics or other supported signals against conditions and take configured actions.
- **Dashboards:** Visualize metrics and monitoring information together.
- **Logs Insights:** Query and analyze log data using the supported query language.

### Monitoring example
For an application on EC2 or Lambda:
1. Identify the important signals: errors, latency, request volume, and resource utilization.
2. Ensure the required metrics and logs are being published.
3. Create an alarm with a meaningful threshold and evaluation period.
4. Configure an action, such as publishing a notification to SNS.
5. Test the alert path and document how to investigate the underlying issue.

An alarm is not useful unless someone or something can respond to it. Choose thresholds based on workload behavior and avoid noisy alerts.

## 3. Amazon SNS vs. Amazon SQS

### Amazon SNS — Simple Notification Service
SNS is a publish/subscribe messaging service. A publisher sends a message to a topic, which can deliver notifications to subscribed endpoints.

Typical use: send an alert to email or fan out an event to multiple subscribers.

### Amazon SQS — Simple Queue Service
SQS is a managed message queue. Producers send messages to a queue and consumers poll and process them.

Typical use: decouple an application from background work so temporary consumer failures do not require the producer to do all processing synchronously.

| Feature | SNS | SQS |
|---|---|---|
| Main pattern | Publish/subscribe | Queue and consumer |
| Delivery model | Push to subscriptions | Consumers poll the queue |
| Common use | Notifications and fan-out | Buffering and asynchronous processing |
| Important concern | Subscription and delivery configuration | Visibility timeout, retries, and dead-letter queues |

SNS and SQS can be combined: SNS fans an event out to several SQS queues, allowing separate consumers to process it independently.

## 4. AWS Lambda

Lambda runs supported code in response to events without requiring you to manage the underlying servers.

Important concepts:
- **Function:** Your deployed code and configuration.
- **Trigger/event source:** Something that invokes the function, such as an API Gateway request, an SQS message, or an S3 event.
- **Execution role:** IAM permissions the function uses to access other AWS services.
- **Timeout and memory:** Runtime limits and resource configuration.
- **Logs and metrics:** Use CloudWatch to observe executions and failures.
- **Retries and idempotency:** Event sources have different retry and delivery behavior; design handlers to safely handle repeated events when needed.

Lambda is not automatically cheaper or better for every workload. Consider execution duration, request volume, concurrency, startup characteristics, and service limits.

## 5. Amazon API Gateway

API Gateway provides managed API front doors for supported API types and integrations.

A common serverless pattern is:
1. A client sends an HTTPS request to API Gateway.
2. API Gateway applies configured routing, authorization, throttling, and other API controls.
3. API Gateway invokes a Lambda integration.
4. Lambda runs business logic and returns a response.
5. CloudWatch captures relevant logs and metrics.

Security considerations:
- Authenticate and authorize callers appropriately.
- Validate input and limit request rates where needed.
- Give Lambda only the IAM permissions it requires.
- Do not expose secret values in logs or responses.
- Configure CORS only when required by the client architecture; CORS is not authentication.

## 6. Hands-on labs

Use a sandbox AWS account. CloudFormation, Lambda, API Gateway, logs, alarms, SNS, and data transfer may incur charges depending on configuration and usage.

### Lab A — CloudFormation
- Create a minimal template with a safe resource or a simple resource such as an S3 bucket configured for private access.
- Validate the template and deploy a stack.
- Inspect stack events and outputs.
- Make a small reviewed change.
- Delete the test stack after checking for retained data and resources.

### Lab B — CloudWatch
- Inspect metrics for a resource you already own.
- Locate a log group and read recent events.
- Create a test alarm for a suitable metric if the lab permits.
- Route a notification to SNS if configured and needed.
- Verify the alert path and document how you would troubleshoot a false or missing alarm.

### Lab C — Serverless flow design
Draw this architecture: client → API Gateway → Lambda → a permitted downstream service. Add CloudWatch logs/metrics and an SNS notification path for operational alerts.

### Lab D — Queue-based processing
- Draw producer → SQS queue → consumer.
- Explain visibility timeout, retry behavior, duplicate processing, and a dead-letter queue.
- Explain how SNS can fan events out to multiple queues.

## 7. Interview questions
1. What is Infrastructure as Code?
2. What are a CloudFormation template and stack?
3. Why use a change set before updating infrastructure?
4. What is the difference between CloudWatch metrics and logs?
5. What does a CloudWatch alarm do?
6. How are SNS and SQS different?
7. When would you combine SNS with SQS?
8. What is AWS Lambda, and what triggers it?
9. Why does Lambda need an execution role?
10. What is idempotency, and why does it matter in event-driven systems?
11. What is API Gateway's role in a serverless architecture?
12. How would you troubleshoot a Lambda function that returns errors or times out?

## 8. Completion checklist
- Explain the IaC workflow and the role of CloudFormation stacks.
- Describe a useful metric, log, alarm, and alert path.
- Compare SNS and SQS with a real example.
- Draw and explain API Gateway → Lambda.
- Explain IAM execution roles, retries, idempotency, and basic cost controls.
- Document your labs, results, diagrams, troubleshooting notes, and cleanup.

## Cost and security reminders
- Check current AWS pricing and set a budget alert before running labs.
- Remove temporary resources and review logs, alarms, API stages, queues, and other resources afterward.
- Never commit credentials, tokens, secret values, or customer data to GitHub.
- Review CloudFormation changes before applying them; infrastructure changes can create charges or delete data.

## Next session
Day 17 — Python for AWS Automation with boto3: sessions, clients, resources, automating EC2/S3/snapshots, pagination, error handling, and cost reporting.
