# AWS-IAM-Access-Analyzer
Identifying resources shared with an external entity

## Write a script of a scenario where IAM Access Analyzer provides the following: 

* A) How it identifies resources In an organization and account that are shared with external entities for example S3 buckets, IAM roles shared with external entities.
* B) How it validates IAM Policies against policy grammar and best practices.
* C) How it generates IAM Policies based on access activities in AWS Cloudtrail logs.
* D) How it analyzes various services like:
    * S3
    * IAM
    * KMS
    * Lambda functions
    * SQS
    * Secrets Manager Secrets
    
* E) Provide a Readme.md file with diagram on how it works/ functions with notes

### A) How it identifies resources In an organization and account that are shared with external entities for example S3 buckets, IAM roles shared with external entities.

Access Analyzer helps you identify the resources in your organization and accounts, such as Amazon S3 buckets or IAM roles, shared with an external entity. This lets you identify unintended access to your resources and data, which is a security risk. Access Analyzer identifies resources shared with external principals by using logic-based reasoning to analyze the resource-based policies in your AWS environment. For each instance of a resource shared outside of your account, Access Analyzer generates a finding. Findings include information about the access and the external principal granted to it. You can review findings to determine whether the access is intended and safe, or the access is unintended and a security risk. In addition to helping you identify resources shared with an external entity, you can use Access Analyzer findings to preview how your policy affects public and cross-account access to your resource before deploying resource permissions.  

An external entity can be another AWS account, a root user, an IAM user or role, a federated
user, an AWS service, an anonymous user, or other entity that you can use to create a filter. When you enable Access Analyzer, you create an analyzer for your entire organization or your account. The organization or account you choose is known as the zone of trust for the analyzer. The analyzer monitors all of the supported resources within your zone of trust. Any access to resources by principals within your zone of trust is considered trusted. Once enabled, Access Analyzer analyzes the policies applied to all of the supported resources in your zone of trust. After the first analysis, Access Analyzer analyzes these policies periodically. If you add a new policy , or change an existing policy, Access Analyzer analyzes the new or updated policy within about 30 minutes.  

When analyzing the policies, if Access Analyzer identifies one that grants access to an external principal that isn't within your zone of trust, it generates a finding. Each finding includes details about the resource, the external entity with access to it, and the permissions granted so that you can take appropriate action. You can view the details included in the finding to determine whether the resource access is intentional or a potential risk that you should resolve. When you add a policy to a resource, or update an existing policy, Access Analyzer analyzes the policy. Access Analyzer also analyzes all resource based policies periodically. On rare occasions under certain conditions, Access Analyzer does not receive notification of an added or updated policy. 

Access Analyzer can take up to 6 hours to generate or resolve findings if you create or delete a multi-region access point associated with an S3 bucket, or update the policy for the multiregion access point. Also, if there is a delivery issue with AWS CloudTrail log delivery, the policy change does not trigger a rescan of the resource reported in the finding. When this happens, Access Analyzer analyzes the new or updated policy during the next periodic scan, which is within 24 hours. If you want to confirm a change you make to a policy resolves an access issue reported in a finding, you can rescan the resource reported in a finding by using the Rescan link in the Findings details page, or by using the StartResourceScan operation of the Access Analyzer API.  

Access Analyzer analyzes only policies applied to resources in the same AWS Region where it's enabled.  
***To monitor all resources in your AWS environment, you must create an analyzer to enable Access Analyzer in each Region where you're using supported AWS resources***

### B) How it validates IAM Policies against policy grammar and best practices.

You can validate your policies using Access Analyzer policy checks. You can create or edit a policy using the AWS CLI, AWS API, or JSON policy editor in the IAM console. Access Analyzer validates your policy against IAM policy grammar and best practices. You can view policy validation check findings that include security warnings, errors, general warnings, and suggestions for your policy. These findings provide actionable recommendations that help you author policies that are functional and conform to security best practices.  

### C) How it generates IAM Policies based on access activities in AWS Cloudtrail logs.

Access Analyzer analyzes your AWS CloudTrail logs to identify actions and services that have been used by an IAM entity (user or role) within your specified date range. It then generates an IAM policy that is based on that access activity. You can use the generated policy to refine an entity's permissions by attaching it to an IAM user or role.   

### D) How it analyzes various services like:

#### *S3* :

When Access Analyzer analyzes Amazon S3 buckets, it generates a finding when an Amazon S3 bucket policy, ACL, or access point, including a multi-region access point, applied to a bucket grants access to an external entity. An external entity is a principal or other entity you can use to create a filter that isn't within your zone of trust. For example, if a bucket policy grants access to another account or allows public access, Access Analyzer generates a finding. However, if you enable Block public access on your bucket, you can block access at the account level or the bucket level.  

Amazon S3 block public access settings override the bucket policies applied to the bucket. The settings also override the access point policies applied to the bucket’s access points. Access Analyzer analyzes block public access settings at the bucket level whenever a policy changes. However, it evaluates the block public access settings at the account level only once every 6 hours. This means that Access Analyzer might not generate or resolve a finding for public access to a bucket for up to 6 hours. For example, if you have a bucket policy that allows public access, Access Analyzer generates a finding for that access. If you then enable block public access to block all public access to the bucket at the account level, Access Analyzer doesn't resolve the finding for the bucket policy for up to 6 hours, even though all public access to the bucket is blocked. For a multi-region access point, Access Analyzer uses an established policy for generating findings. Access Analyzer evaluates changes to multi-region access points once every 6 hours. This means Access Analyzer doesn’t generate or resolve a finding for up to 6 hours, even if you create or delete a multi region access point, or update the policy for it.

#### *IAM* :

For IAM roles, Access Analyzer analyzes trust policies. In a role trust policy, you define the principals that you trust to assume the role. A role trust policy is a required resource-based policy that is attached to a role in IAM. Access Analyzer generates findings for roles within the zone of trust that can be accessed by an external entity that is outside your zone of trust.  

*An IAM role is a global resource. If a role trust policy grants access to an external entity, Access Analyzer generates a finding in each enabled Region.*

#### *KMS* :

For AWS KMS keys, Access Analyzer analyzes the key policies and grants applied to a key. Access Analyzer generates a finding if a key policy or grant allows an external entity to access the key. For example, if you use the kms:CallerAccount condition key in a policy statement to allow access to all users in a specific AWS account, and you specify an account other than the current account (the zone of trust for the current analyzer), Access Analyzer generates a finding. When Access Analyzer analyzes a KMS key it reads key metadata, such as the key policy and list of grants. If the key policy doesn't allow the Access Analyzer role to read the key metadata, an Access Denied error finding is generated. For example, if the following example policy statement is the only policy applied to a key, it results in an Access Denied error finding in Access Analyzer:

> {  
>  "Sid": "Allow access for Key Administrators",  
>  "Effect": "Allow",  
>  "Principal": {  
>  "AWS": "arn:aws:iam::111122223333:role/Admin"  
>  },  
>  "Action": "kms:*",  
>  "Resource": "*"  
> }  

Because this statement allows only the role named Admin from the AWS account 111122223333 to access the key, an Access Denied error finding is generated because Access Analyzer isn't able to fully analyze the key. An error finding is displayed in red text in the Findings table. The finding looks similar to the following:

> {  
>  "error": "ACCESS_DENIED",  
>  "id": "12345678-1234-abcd-dcba-111122223333",  
>  "analyzedAt": "2019-09-16T14:24:33.352Z",  
>  "resource": "arn:aws:kms:uswest-2:1234567890:key/1a2b3c4d-5e6f-7a8b-9c0d-1a2b3c4d5e6f7g8a",  
>  "resourceType": "AWS::KMS::Key",  
>  "status": "ACTIVE",  
>  "updatedAt": "2019-09-16T14:24:33.352Z"  
> }  

When you create a KMS key, the permissions granted to access the key depend on how you create the key. If you receive an Access Denied error finding for a key resource, apply the following policy statement to the key resource to grant Access Analyzer permission to access the key.

> {  
>  "Sid": "Allow Access Analyzer access to key metadata",  
>  "Effect": "Allow",  
>  "Principal": {  
>  "AWS": "arn:aws:iam::111122223333:role/aws-service-role/accessanalyzer.amazonaws.com/AWSServiceRoleForAccessAnalyzer"  
>  },  
>  "Action": [  
>  "kms:DescribeKey",  
>  "kms:GetKeyPolicy",  
>  "kms:List*"  
>  ],  
>  "Resource": "*"  
> }  

After you receive an Access Denied finding for a KMS key resource, and then resolve the finding by updating the key policy, the finding is updated to a status of Resolved. If there are policy statements or key grants that grant permission to the key to an external entity, you might see additional findings for the key resource.  

#### *Lambda functions*

For AWS Lambda functions, Access Analyzer analyzes policies, including condition statements in a policy, that grants access to the function to an external entity. Access Analyzer also analyzes permissions granted when using the *AddPermission* operation of the AWS Lambda API with an EventSourceToken.  

#### *SQS*

For Amazon SQS queues, Access Analyzer analyzes policies, including condition statements in a policy, that allow an external entity access to a *queue*.  

#### *Secrets Manager Secrets*

For AWS Secrets Manager secrets, Access Analyzer analyzes policies, including condition statements in a policy, that allow an external entity to access a *secret*.  

### E) Provide a Readme.md file with diagram on how it works/ functions with notes

AWS IAM Access Analyzer is built on Zelkova, which translates IAM policies into equivalent logical statements, and runs a suite of general-purpose and specialized logical solvers (satisfiability modulo theories) against the problem. Access Analyzer applies Zelkova repeatedly to a policy with increasingly specific queries to characterize classes of behaviors the policy allows, based on the content of the policy. Access Analyzer does not examine access logs to determine whether an external entity accessed a resource within your zone of trust. It generates a finding when a resource-based policy allows access to a resource, even if the resource was not accessed by the external entity. 

Access Analyzer also does not consider the state of any external accounts when making its determination. That is, if it indicates that account 11112222333 can access your S3 bucket, it knows nothing about the state of users, roles, service control policies (SCP), and other relevant configurations in that account. This is for customer privacy – Access Analyzer doesn't consider who owns the other account. It is also for security – if the account is not owned by the Access Analyzer customer, it is still important to know that an external entity could gain access to their resources even if there are currently no principals in the account that could access the resources. Access Analyzer considers only certain IAM condition keys that external users cannot directly influence, or that are otherwise impactful to authorization. 

Access Analyzer does not currently report findings from AWS service principals or internal service accounts. In rare cases where Access Analyzer isn't able to fully determine whether a policy statement grants access to an external entity, it errs on the side of declaring a false positive finding. Access Analyzer is designed to provide a comprehensive view of the resource sharing in your account, and strives to minimize false negatives.  
