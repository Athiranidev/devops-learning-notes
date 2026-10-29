# AWS IAM and S3 Permissions — Quick Revision

## 1. IAM: Identity and Access Management

IAM controls who can access AWS resources and which actions they can perform.

Important concepts:
- IAM identity: represents a user, role, or other security principal.
- IAM role: an identity that a trusted principal can assume.
- IAM policy: defines permitted or denied actions on resources.
- Temporary credentials: allow an application to make AWS requests without storing long-lived access keys in its code.

## 2. IAM Role for an EC2 Application

Example:
An application running on EC2 needs to read a configuration file from S3.

Recommended approach:
1. Create an IAM role that EC2 is trusted to assume.
2. Attach an appropriate permissions policy to the role.
3. Attach the role to the EC2 instance.
4. Let the application use the role's temporary credentials.

Avoid hardcoding long-lived AWS access keys in application code.

## 3. Trust Policy vs. Permissions Policy

### Trust policy
Defines who is allowed to assume the role.

Example:
The EC2 service must be trusted to assume an IAM role intended for an EC2 instance.

### Permissions policy
Defines what the assumed role can do.

Example:
The role may be allowed to read objects from a particular S3 bucket.

Remember:
- Trust policy: Who can assume the role?
- Permissions policy: What can the role do?

A role with correct S3 permissions still cannot be used by EC2 if EC2 is not permitted to assume it.

## 4. S3 Permissions

Different actions require different permissions.

| Action | IAM permission | Purpose |
|---|---|---|
| Read an object | s3:GetObject | Download or read an object |
| Upload an object | s3:PutObject | Upload or create an object |
| List a bucket | s3:ListBucket | List objects in a bucket |

Being allowed to read an object does not automatically grant permission to upload one.

### Example scenario
The EC2 role can read a configuration file, but uploading a backup returns AccessDenied.

Possible cause:
The role allows s3:GetObject but lacks s3:PutObject.

Potential fix:
Add s3:PutObject for the required backup location, following least privilege.

## 5. ARN: Amazon Resource Name

An ARN identifies an AWS resource.

Example:
arn:aws:s3:::my-app-bucket/backups/*

Parts:
- arn: AWS resource identifier prefix.
- aws: AWS partition.
- s3: Amazon S3 service.
- my-app-bucket: bucket name.
- backups/*: objects under the backups/ path.

Why it matters:
A policy may allow an action but apply to the wrong bucket or object path.

Use the correct resource ARN for the intended operation:
- Bucket-level actions such as s3:ListBucket generally use the bucket ARN.
- Object-level actions such as s3:GetObject and s3:PutObject generally use object ARNs.

## 6. S3 Encryption: SSE-S3 vs. SSE-KMS

### SSE-S3
- S3 automatically encrypts new objects at rest by default using S3-managed encryption keys.
- A separate customer-managed KMS key is not required for SSE-S3.

### SSE-KMS
- S3 uses an AWS KMS key for encryption.
- Required IAM permissions and the KMS key policy must permit the needed operations.
- Uploads using SSE-KMS generally require kms:GenerateDataKey.
- Multipart uploads can require additional permissions, such as kms:Decrypt.

Remember:
- S3 permissions control the object operation.
- KMS permissions control use of the encryption key when SSE-KMS is involved.
- Don't investigate KMS permissions first if the bucket uses SSE-S3, unless other evidence points to KMS.

## 7. Troubleshooting S3 AccessDenied

Use evidence rather than immediately creating new credentials.

1. Confirm the application is using the intended IAM role.
2. Check that EC2 is trusted to assume the role.
3. Verify the required action, such as s3:PutObject.
4. Confirm the policy's resource ARN matches the intended bucket and object path.
5. Inspect the S3 bucket policy for explicit Deny rules or restrictive conditions.
6. If applicable, inspect permissions boundaries, organization policies, and VPC endpoint policies.
7. If SSE-KMS is used, check the relevant KMS permissions and key policy.
8. Retest the operation after correcting the identified cause.

## 8. Allow, Deny, and Default Deny

- Allow: grants an action when applicable policy evaluation permits it.
- Explicit Deny: overrides an applicable Allow.
- No applicable Allow: access is generally denied by default.

Example:
An IAM role allows s3:PutObject, but a bucket policy explicitly denies the upload.

Result:
The upload fails. Adding another Allow does not override the explicit Deny.

## 9. Least Privilege

Give an identity only the permissions needed for its task.

Example:
If an application uploads backups only to a designated backups/ path, restrict s3:PutObject to that path rather than granting unrestricted access to every bucket.

Benefits:
- Reduces unnecessary access.
- Limits the impact of compromised credentials.
- Makes permissions easier to review.

## 10. Interview Quick Review

Q: What is the difference between a trust policy and a permissions policy?
A: Trust controls who can assume a role; permissions control what that role can do.

Q: Why can an S3 read succeed while an upload fails?
A: s3:GetObject and s3:PutObject are different permissions.

Q: What is an ARN?
A: An identifier for an AWS resource.

Q: Does every S3 bucket require a customer-managed KMS key?
A: No. S3 automatically encrypts new objects at rest using SSE-S3 by default.

Q: Why might an upload fail even when s3:PutObject is allowed?
A: The resource ARN may be wrong, or another applicable policy or encryption restriction may block the operation.

Q: What happens when an applicable policy explicitly denies an action?
A: The explicit Deny overrides an Allow.

Core principle:
Identify the acting identity, verify the required action and resource, inspect applicable restrictions, then test again.
