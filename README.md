Welcome to Secrects-in-CF.

I appear to have a concept and it is described in below.

1. Description:

It is possible to have a plain text password and encrypt it with your AWS KMS key.

The resulting string (hash) can be decrypted by our AWS infrastructure and can be added 

to AWS Secrets Manager so that any AWS service available in our account can use the password string we provided.

2. Solution:

We will need some trivial things for starters:

* dedicated AWS IAM Policy for KMS functions

* AWS KMS Key

* AWS User with direct attachment to the IAM Policy for KMS

* AWS CLI v2 configured with the user credentials (aws configure)

Sample password string "123456"

echo "123456" > pass (create the file with our password string in plain text)

To encrypt the 123456 string from our pass file we use this command:

aws kms encrypt --key-id *key-id-here* --plaintext fileb://pass --output text --query CiphertextBlob | base64 --decode > encrypted (encrypted file cointains our pw in base64)

To decrypt our encrypted file we use:

aws kms decrypt --ciphertext-blob fileb://encrypted --key-id *key-id-here* --output text --query Plaintext | base64 --decode > decrypted

In fact, we don't need the decrypted output so we should rm pass decrypted while keeping encrypted safe.

Put encrypted file in an AWS S3 bucket for further manipulation:

Switch aws cli to a profile with S3 create access (let's say it is called owner) and do it:

aws s3api create-bucket --bucket unsecured-content --region eu-central-1 --create-bucket-configuration LocationConstraint=eu-central-1 --profile owner

Upload encrypted as an object to the new s3 bucket:

aws s3api put-object --bucket unsecured-content --key encrypted --body encrypted --profile owner

Now we have our encrypted password with this ARN:

arn:aws:s3:::unsecured-content/encrypted
