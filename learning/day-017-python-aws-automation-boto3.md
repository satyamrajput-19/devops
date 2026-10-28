# Day 17: Python for AWS Automation with Boto3

## Learning goals
- Explain Boto3 sessions, clients, and resources.
- List EC2 instances and S3 buckets safely.
- Create EBS snapshots and understand pagination.
- Handle AWS API errors.
- Produce a basic Cost Explorer report.
- Follow least privilege and clean up lab resources.

## 1. Setup and credentials

Install Boto3 in a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate       # Windows: .venv\\Scripts\\activate
python -m pip install --upgrade pip boto3
python -m pip freeze > requirements.txt
```

Configure a named AWS CLI profile using `aws configure --profile dev-lab`, or use an attached IAM role in an approved AWS environment. Do not put access keys in source code, commit them, or paste them into chat. Prefer short-lived credentials and least privilege.

Check the active identity:

```bash
aws sts get-caller-identity --profile dev-lab
```

## 2. Sessions, clients, and resources

- **Session**: holds configuration such as credentials, region, and profile; useful for creating multiple service objects consistently.
- **Client**: low-level API wrapper closely aligned with AWS API operations. It returns dictionaries and exposes paginators for many list operations.
- **Resource**: higher-level object-oriented interface for supported services. It is available for some services, including EC2 and S3, but not every AWS service/API operation.

```python
import boto3

session = boto3.Session(profile_name="dev-lab", region_name="ap-south-1")
ec2_client = session.client("ec2")
s3_client = session.client("s3")
ec2_resource = session.resource("ec2")
s3_resource = session.resource("s3")
```

Change the region/profile to the values approved for your lab. If using an IAM role or environment credentials, omit `profile_name`.

## 3. List EC2 instances (read-only)

Create `scripts/list_ec2.py`:

```python
import boto3
from botocore.exceptions import BotoCoreError, ClientError

session = boto3.Session(profile_name="dev-lab", region_name="ap-south-1")
ec2 = session.client("ec2")

try:
    paginator = ec2.get_paginator("describe_instances")
    for page in paginator.paginate():
        for reservation in page.get("Reservations", []):
            for instance in reservation.get("Instances", []):
                print({
                    "id": instance["InstanceId"],
                    "state": instance["State"]["Name"],
                    "type": instance["InstanceType"],
                    "az": instance["Placement"]["AvailabilityZone"],
                })
except (ClientError, BotoCoreError) as exc:
    print(f"AWS request failed: {exc}")
    raise
```

Try it and compare the output with the EC2 console. The paginator handles results spread across multiple API pages.

## 4. List S3 buckets

Create `scripts/list_s3_buckets.py`:

```python
import boto3
from botocore.exceptions import BotoCoreError, ClientError

s3 = boto3.Session(profile_name="dev-lab").client("s3")

try:
    response = s3.list_buckets()
    for bucket in response.get("Buckets", []):
        print(bucket["Name"], bucket["CreationDate"])
except (ClientError, BotoCoreError) as exc:
    print(f"Could not list buckets: {exc}")
    raise
```

Note: S3 bucket listing is account-level; the credentials need permission to list buckets. Do not print object contents or sensitive metadata unnecessarily.

## 5. Create EBS snapshots carefully

A snapshot can incur storage charges. Use a volume ID you own and are authorized to back up. This script requires an explicit volume ID and adds a description:

```python
import argparse
import boto3
from botocore.exceptions import BotoCoreError, ClientError

parser = argparse.ArgumentParser()
parser.add_argument("--volume-id", required=True, help="EBS volume ID you own")
args = parser.parse_args()

ec2 = boto3.Session(profile_name="dev-lab", region_name="ap-south-1").client("ec2")

try:
    result = ec2.create_snapshot(
        VolumeId=args.volume_id,
        Description="Day 17 Boto3 learning snapshot"
    )
    print("Snapshot requested:", result["SnapshotId"], result["State"])
except (ClientError, BotoCoreError) as exc:
    print(f"Snapshot request failed: {exc}")
    raise
```

Run only after verifying the account, region, volume ID, permissions, and cost. A snapshot is asynchronous; its initial state may be `pending`. Use the console or `describe_snapshots` to track it. Delete the lab snapshot after confirming it is no longer needed. Do not automate deletion of production backups.

## 6. Error handling patterns

- Catch `ClientError` for AWS service/API errors and inspect `exc.response["Error"]["Code"]`.
- Catch `BotoCoreError` for lower-level SDK issues.
- Handle expected cases explicitly, such as access denied, missing resources, and throttling.
- Do not swallow exceptions silently. Log enough context to troubleshoot without logging secrets.
- Use retries with backoff for retryable throttling/transient errors; Boto3 standard retry configuration can help.

Example:

```python
from botocore.exceptions import ClientError

try:
    response = s3.head_bucket(Bucket="your-approved-bucket-name")
except ClientError as exc:
    code = exc.response.get("Error", {}).get("Code", "Unknown")
    print(f"Bucket check failed ({code})")
    raise
```

Replace the example bucket name with one you are authorized to inspect.

## 7. Cost reporting with Cost Explorer

Cost Explorer must be enabled for the account and the identity needs appropriate Cost Explorer permissions. Cost data can be delayed; this is not a real-time bill. This example requests daily unblended cost for the last 7 complete days:

```python
from datetime import date, timedelta
from decimal import Decimal
import boto3
from botocore.exceptions import BotoCoreError, ClientError

today = date.today()
end = today                         # End is exclusive
start = today - timedelta(days=7)

ce = boto3.Session(profile_name="dev-lab", region_name="us-east-1").client("ce")

try:
    result = ce.get_cost_and_usage(
        TimePeriod={"Start": start.isoformat(), "End": end.isoformat()},
        Granularity="DAILY",
        Metrics=["UnblendedCost"],
    )
    for day in result.get("ResultsByTime", []):
        amount = Decimal(day["Total"]["UnblendedCost"]["Amount"])
        unit = day["Total"]["UnblendedCost"]["Unit"]
        print(day["TimePeriod"]["Start"], f"{amount:.2f}", unit)
except (ClientError, BotoCoreError) as exc:
    print(f"Cost report failed: {exc}")
    raise
```

Cost Explorer is a global billing service commonly accessed through `us-east-1`; use the endpoint/region supported by your account. The returned currency is determined by the account's billing configuration. This sample reports account-level daily cost; it does not automatically attribute every charge to a specific EC2 instance. For detailed allocation, explore cost allocation tags and Cost Explorer groupings.

## 8. Suggested project structure

```text
day-17-boto3-automation/
├── requirements.txt
├── README.md
└── scripts/
    ├── list_ec2.py
    ├── list_s3_buckets.py
    ├── create_snapshot.py
    └── cost_report.py
```

Keep the examples separate initially, then refactor shared session/configuration and logging once each works.

## 9. Hands-on checklist

- [ ] Create a virtual environment and install Boto3.
- [ ] Verify identity with STS.
- [ ] List EC2 instances using a paginator.
- [ ] List S3 buckets.
- [ ] Create and monitor one approved test-volume snapshot, or document why you skipped this chargeable task.
- [ ] Run the Cost Explorer report if access is available; otherwise document the missing permission/service setup.
- [ ] Test an expected error safely (for example, a nonexistent bucket with `head_bucket`).
- [ ] Add screenshots or sanitized output to the README. Never include credentials, account secrets, or sensitive resource details.
- [ ] Delete the lab snapshot when no longer needed and check for other billable resources.

## 10. Interview questions

1. What is the difference between a Boto3 session, client, and resource?
2. Why use a paginator?
3. What is the difference between `ClientError` and `BotoCoreError`?
4. Why should AWS credentials not be hardcoded?
5. What is an EBS snapshot, and why is its state initially sometimes `pending`?
6. Why can Cost Explorer data differ from current resource usage?
7. How would you make an automation script safe to rerun?
8. How would you handle throttling and transient API failures?

## Day 17 completion criteria

You can explain each script, run the read-only scripts, describe the permissions required, and document the outcome and any errors. Do not claim a lab succeeded unless you ran it and observed the result.

**Next:** Day 18 — Docker fundamentals and containerizing an application.
