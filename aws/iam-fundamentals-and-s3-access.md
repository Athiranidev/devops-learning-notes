AWS IAM Fundamentals and S3 Access — Quick Revision Notes
1. IAM: the basic idea
AWS Identity and Access Management (IAM) controls who can access AWS and what they can do.
- IAM user: an identity for a person or workload that needs to authenticate to AWS.
- IAM group: a collection of IAM users. Policies attached to the group apply to its users.
- IAM role: an identity with permissions that a trusted user, application, or AWS service can assume.
- IAM policy: a set of rules describing allowed or denied actions on resources.
2. IAM users and groups
- A group helps manage shared permissions centrally.
- One IAM user can belong to multiple groups.
- A group cannot contain another group or an IAM role.
- A user can also have policies attached directly to the user.
- Applicable Allow permissions can combine, but explicit Deny and other restrictions still apply.
Example:
- Developers group: S3 read access (s3:GetObject).
- Backup-Uploaders group: upload access (s3:PutObject) to an approved backup path.
- A developer in both groups receives the applicable permissions from both groups.
- Removing that developer from Backup-Uploaders removes the permissions supplied by that group, but does not remove permissions from Developers.
3. IAM user vs. IAM role
Human developer accessing AWS
A person authenticates using an assigned identity and receives only the permissions granted to that identity.
Application running on EC2 accessing AWS
A common secure pattern is:
1. Create an IAM role.
2. Configure its trust policy to allow the EC2 service to assume it.
3. Attach the required permissions policies to the role.
4. Attach the role to the EC2 instance (AWS uses an instance profile for this association).
5. The application or AWS SDK obtains temporary role credentials from the instance environment and uses them to call AWS services.
Do not hardcode long-lived AWS access keys in application source code when an EC2 instance role is appropriate.
4. Trust policy vs. permissions policy
- Trust policy: who or what is allowed to assume the role?
- Permissions policy: after the role is assumed, what AWS actions can it perform, and on which resources?
Both are required for the EC2-role pattern:
- If EC2 is not trusted to assume the role, the instance cannot obtain credentials through that role.
- If the role can be assumed but lacks an action, the application cannot perform that action.
5. Common S3 permissions
Action	Meaning
s3:GetObject	Read/download an object
s3:PutObject	Upload/create an object
s3:ListBucket	List objects in a bucket, when required


Reading an object does not automatically permit uploading one. These are separate permissions.
6. ARN: identify the exact resource
ARN means Amazon Resource Name. It identifies an AWS resource in a policy.

Example object ARN:

arn:aws:s3:::my-app-bucket/backups/*

Breakdown:
- arn: resource identifier format
- aws: AWS partition
- s3: service
- my-app-bucket: bucket name
- backups/*: objects under the backups prefix
Important distinction:
- Bucket-level actions such as s3:ListBucket normally use the bucket ARN: arn:aws:s3:::my-app-bucket.
- Object-level actions such as s3:GetObject and s3:PutObject use object ARNs, such as arn:aws:s3:::my-app-bucket/backups/*.
Always check that the policy resource matches the actual bucket and object key the application uses.
7. S3 encryption: SSE-S3 vs. SSE-KMS
- SSE-S3: S3 manages the encryption keys. New S3 objects are encrypted at rest by default with SSE-S3 unless a different encryption configuration is selected.
- SSE-KMS: S3 uses an AWS KMS key. The caller may need the relevant KMS permissions, and the KMS key policy must also permit the required operation.
For an SSE-KMS upload, kms:GenerateDataKey is generally required. Other operations, such as multipart upload, may require additional KMS permissions.

Do not investigate KMS permissions automatically for every S3 AccessDenied error. First establish which encryption method the bucket/object actually uses.
8. Allow, implicit Deny, and explicit Deny
- Allow: an applicable policy grants an action, subject to other policy controls.
- Implicit Deny: no applicable policy grants the requested action.
- Explicit Deny: an applicable policy explicitly blocks the action.
An applicable explicit Deny overrides an Allow. Adding another Allow does not cancel an explicit Deny. Find and understand the reason for the Deny before changing it.
9. S3 AccessDenied troubleshooting checklist
When an EC2 application gets AccessDenied while uploading to S3, investigate in this order:
1. Confirm the identity: is the application using the expected EC2 role, rather than unexpected credentials?
2. Confirm role attachment and trust: is the role attached to the EC2 instance, and does its trust policy allow EC2 to assume it?
3. Check the action: does the permissions policy allow s3:PutObject (not only s3:GetObject)?
4. Check the resource ARN: does the policy cover the exact bucket and object prefix used by the application?
5. Check the bucket policy: is there an explicit Deny or a condition that blocks the request?
6. Check other restrictions when relevant: permissions boundary, AWS Organizations SCP, VPC endpoint policy, or other applicable controls.
7. Check encryption only when relevant: if SSE-KMS is used, verify required KMS permissions and the key policy. If SSE-S3 is used, a customer-managed KMS permission is not required just for that default encryption.
8. Retest and verify: make the smallest justified policy change, then retry the upload and review the resulting error/event details.
Do not solve an authorization error by creating another long-lived access key without first identifying the missing permission or restriction.
10. Concepts to remember
- IAM user = individual identity.
- IAM group = collection of users with shared policies.
- IAM role = assumable identity for a trusted principal or service.
- Trust policy = who can assume a role.
- Permissions policy = what the identity can do.
- s3:GetObject = read an object.
- s3:PutObject = upload an object.
- ARN = exact AWS resource identifier.
- SSE-S3 = S3-managed encryption.
- SSE-KMS = encryption using a KMS key; relevant KMS permissions may be needed.
- Explicit Deny overrides Allow.
11. Current learning checkpoint
We have covered IAM users, groups, roles, trust policies, permissions policies, S3 read/write permissions, ARNs, S3 encryption choices, and explicit Deny.

Next topic to continue: EC2 role attachment. If a role has the correct trust and S3 permissions but is not attached to the EC2 instance, the application will not receive that role's credentials through the instance profile. We will examine that scenario next.
