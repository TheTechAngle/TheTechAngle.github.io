On AWS, authentication and authorization are primarily handled by Identity and Access Management (IAM).
Users or applications without AWS accounts can be authenticated and given temporary access to AWS resources using an external service such as Kerberos, Microsoft Active Directory, or the Lightweight Directory Access Protocol (LDAP).

Anything done in IAM is done in region 'Global'

## 1) IAM Identities
* Every AWS account has a default root user, with full rights to everything in the account
* Hence, root is the point of weakness
* New user - default level of access a newly created IAM User is granted is no access to anything
* Power User Access allows access to all AWS services except the management of groups and users within IAM.

### IAM Policies
* A policy might, for instance, allow (the effect) the creation of buckets (the action) within
S3 (the resource). 
* A policy can be attached to infinite identities, though a sinlge identity can have 10 policies
* Allow vs deny confilcts are resolved by denying 

### Protecting User and Root Accounts

* create at least one new user with enough
authority to get the necessary work done, give main admin user the AdministratorAccess policy
* Unlike root, AdministratorAccess policy lacks powers like including the ability to create or delete
account-wide budgets and enable MFA Delete on an S3 bucket.
* Delete any access keys associated with root.
* Assign a long and complex password and store it in a secure password vault.
* Enable multifactor authentication (MFA) for the root account.
* Wherever possible, don’t use root to perform administration operations.

Users can use the My Security Credentials page to view settings and 
* retrieving you Amazon ID
* Update a password
* Managing MFA
* Managing access keys
* Generating key pairs for authenticating signed URLs
* Generating X.509 certificates to encrypt Simple Object Access Protocol (SOAP)
requests to those AWS services that allow it 
NOTE: SOAP requests to S3 and Amazon Mechanical Turk are an exception to this
rule, as they use regular access keys rather than X.509 certificates.

### Access Keys
* You can make your local environment aware of the access key ID and secret access key.
* Deactivate unused keys
* Key rotation can be ensured in the password policy of IAM account, where keys are replaces every x number of days
* aws iam get-access-key-last-used --access-key-id ABCDEFGHIJKLMNOP can be used to determine whether any applications are still using an old key.
* Users assigned Access key id(username), secret access key(password) for programmatic access
* To access the console you use an account and password combination. To access AWS programmatically you use a Key and Secret Key combination
* New users have NO permissions when created

### Using IAM Roles with EC2

* Create role -> choose service that will be using it (EC2) -> administrative access
* Ec2 -> actions -> attach IAM role -> choose the role
* JSON form of the access policy will be available on aws
* No credentials stored on the ec2 instance
* ROLES ARE UNIVERSAL


-----
FAQ
There are three options  which can be used to secure access to files stored in S3. CloudFront Signed URLs and CloudFront Signed Cookies are different ways to ensure that users attempting access to files in an S3 bucket can be authorised.
An Origin Access Identity on the other hand, is a virtual user identity that is used to give the CloudFront distribution permission to fetch a private object from an S3 bucket. 

