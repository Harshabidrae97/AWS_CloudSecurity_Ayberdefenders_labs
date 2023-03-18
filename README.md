# AWS Bucket - Cloud Security ☁️ 🪣 🔐


## Category
- Cloud Security, Blue team, Incident Response

## Challenge Details

* Welcome, Defender! As an incident responder, we’re granting you access to the AWS account called “Security” as an IAM user. This account contains a copy of the logs during the time period of the incident and has the ability to assume the “Security” role in the target account so you can look around to spot the misconfigurations that allowed for this attack to happen.

## Credentials

* Your IAM credentials for the Security account:

* Login: https://flaws2-security.signin.aws.amazon.com/console
* Account ID: 322079859186
* Username: Security
* Password: password
* Access Key: AKIAIUFNQ2WCOPTEITJQ
* Secret Key: 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

Environment The credentials above give you access to the Security account, which can assume the role of “security” in the Target account. You also have access to an S3 bucket, named flaws2_logs, in the Security account, that contains the CloudTrail logs recorded during a successful compromise

# Solution

1. **What is the full AWS CLI command used to configure credentials?**
- aws configure

2. **What is the ‘creation’ date of the bucket ‘flaws2-logs’?**

- aws s3api list-buckets

```json
{
    "Buckets": [
        {
            "Name": "flaws2-logs",
            "CreationDate": "2018-11-19T20:54:31+00:00"
        }
    ],
    "Owner": {
        "DisplayName": "scott+flaws2_security",
        "ID": "0ff467deaf461e549934997a2df02d29c8010173b1464262782d522bce63bf46"
    }
}
```

    - The answer to the question is as below date-time format. 2018-11-19 20:54:31 UTC

3. **What is the name of the first generated event -according to time?**

```
aws s3 cp s3://flaws2-logs/AWSLogs/653711331788/CloudTrail/us-east-1/2018/11/28/653711331788_CloudTrail_us-east-1_20181128T2235Z_cR9ra7OH1rytWyXY.json.gz . --no-sign-request

cat 653711331788_CloudTrail_us-east-1_20181128T2235Z_cR9ra7OH1rytWyXY.json | jq '.Records[0].eventName'
"AssumeRole"
```
4. **What source IP address generated the event dated 2018-11-28 at 23:03:20 UTC?**

```
cat 653711331788_CloudTrail_us-east-1_20181128T2305Z_83VTWZ8Z0kiEC7Lq.json | jq '.Records[0].sourceIPAddress'
"34.234.236.212"
```

5. **Which IP address does not belong to Amazon AWS infrastructure?**

```
cat 653711331788_CloudTrail_us-east-1_20181128T2305Z_zKlMhON7EpHala9u.json | jq '.Records[0]'
```

```sh
{
  "eventVersion": "1.05",
  "userIdentity": {
    "type": "AWSAccount",
    "principalId": "",
    "accountId": "ANONYMOUS_PRINCIPAL"
  },
  "eventTime": "2018-11-28T23:02:56Z",
  "eventSource": "s3.amazonaws.com",
  "eventName": "GetObject",
  "awsRegion": "us-east-1",
  "sourceIPAddress": "104.102.221.250",
  "userAgent": "[Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36]",
  "requestParameters": {
    "bucketName": "flaws2.cloud",
    "key": "index.htm"
  },
  "responseElements": null,
  "additionalEventData": {
    "x-amz-id-2": "E/0Pm6UrRvpvNEYDjXhDnCsudplsm91YXPM5ShG/caAJ8H/adUjxO4fTIWRNKLwkVB9G6zEszOI="
  },
  "requestID": "D9DB3DE51407FFEC",
  "eventID": "38993e7c-b9e9-46eb-9f93-82e710147d39",
  "readOnly": true,
  "resources": [
    {
      "type": "AWS::S3::Object",
      "ARN": "arn:aws:s3:::flaws2.cloud/index.htm"
    },
    {
      "accountId": "653711331788",
      "type": "AWS::S3::Bucket",
      "ARN": "arn:aws:s3:::flaws2.cloud"
    }
  ],
  "eventType": "AwsApiCall",
  "recipientAccountId": "653711331788",
  "sharedEventID": "9deca33a-d705-42ef-b425-cafe63c5011c"
}
```

- **Narrowing down the answer further as below.**

```
cat 653711331788_CloudTrail_us-east-1_20181128T2305Z_zKlMhON7EpHala9u.json | jq '.Records[0].sourceIPAddress'
"104.102.221.250"
```

6. **Which user issued the ‘ListBuckets’ request?**

```
cat 653711331788_CloudTrail_us-east-1_20181128T2310Z_jQajCuiobojD8I4y.json | jq '.Records[0]'
```
```sh
{
  "eventVersion": "1.05",
  "userIdentity": {
    "type": "AssumedRole",
    "principalId": "AROAJQMBDNUMIKLZKMF64:d190d14a-2404-45d6-9113-4eda22d7f2c7",
    "arn": "arn:aws:sts::653711331788:assumed-role/level3/d190d14a-2404-45d6-9113-4eda22d7f2c7",
    "accountId": "653711331788",
    "accessKeyId": "ASIAZQNB3KHGNXWXBSJS",
    "sessionContext": {
      "attributes": {
        "mfaAuthenticated": "false",
        "creationDate": "2018-11-28T22:31:59Z"
      },
      "sessionIssuer": {
        "type": "Role",
        "principalId": "AROAJQMBDNUMIKLZKMF64",
        "arn": "arn:aws:iam::653711331788:role/level3",
        "accountId": "653711331788",
        "userName": "level3"
      }
    }
  },
  "eventTime": "2018-11-28T23:09:28Z",
  "eventSource": "s3.amazonaws.com",
  "eventName": "ListBuckets",
  "awsRegion": "us-east-1",
  "sourceIPAddress": "104.102.221.250",
  "userAgent": "[aws-cli/1.16.19 Python/2.7.10 Darwin/17.7.0 botocore/1.12.9]",
  "requestParameters": null,
  "responseElements": null,
  "requestID": "4698593B9338B27F",
  "eventID": "65e111a0-83ae-4ba8-9673-16291a804873",
  "eventType": "AwsApiCall",
  "recipientAccountId": "653711331788"
}
```
- **Narrowing down the answer to username as below**

```
cat 653711331788_CloudTrail_us-east-1_20181128T2310Z_jQajCuiobojD8I4y.json | jq '.Records[0].userIdentity.sessionContext.sessionIssuer.userName'
"level3"
```

7. **What was the first request issued by the user ‘level1’?**

```
cat 653711331788_CloudTrail_us-east-1_20181128T2310Z_7J9NEIxrjJsrlXSd.json | jq '.Records[0].eventName'
"CreateLogStream"
```

# Common vulnerabilities and threats that S3 buckets may face:

```
1. Misconfiguration: S3 buckets can be accidentally or inadvertently misconfigured, making them publicly accessible or allowing unauthorized access to sensitive data.

2. Access Control: Improperly configured access controls can also lead to unauthorized access and data breaches.

3. Data Leaks: S3 buckets can also be vulnerable to data leaks caused by accidental or intentional exposure of sensitive information.

4. Malware Infection: S3 buckets can be infected with malware through infected files that are uploaded to the bucket.

5. Data Tampering: S3 buckets can be vulnerable to data tampering, where attackers modify or alter the data stored in the bucket, compromising its integrity.

6. Denial of Service (DoS) Attacks: S3 buckets can also be targeted with DoS attacks, where attackers flood the bucket with traffic, making it unavailable to legitimate users.

7. Exploits and Vulnerabilities: S3 buckets can also be vulnerable to exploits and vulnerabilities, such as cross-site scripting (XSS), SQL injection, and buffer overflow attacks.

To mitigate these vulnerabilities and threats, it's important to implement proper security controls and best practices, such as restricting access to S3 buckets, regularly monitoring and logging bucket activity, implementing encryption, and regularly reviewing and updating configurations and settings. It's also important to stay up to date with the latest security updates and patches, and to regularly train employees on security best practices.
```
# Here are some detailed steps you can take to secure your S3 bucket:

```
1. Secure access to your S3 bucket by setting up AWS Identity and Access Management (IAM) policies. IAM policies allow you to control who can access your S3 bucket and what actions they can perform.

2. Implement encryption to protect your data in transit and at rest. You can use AWS Key Management Service (KMS) to manage your encryption keys and encrypt data stored in your S3 bucket. You can also use SSL/TLS to encrypt data in transit.

3. Enable versioning to protect against accidental or malicious deletion of your data. Versioning allows you to recover deleted files and restore previous versions of your data.

4. Use bucket policies to restrict access to your S3 bucket based on IP address or other conditions. Bucket policies can also be used to enforce security best practices, such as requiring SSL/TLS encryption for all requests.

5. Implement logging and monitoring to detect and respond to security incidents. AWS provides various logging and monitoring tools, such as Amazon CloudWatch, that can be used to monitor S3 bucket activity and detect security threats.

6. Regularly review your S3 bucket settings and configurations to ensure that they are up to date and meet your security requirements. AWS provides various tools, such as AWS Trusted Advisor, that can be used to identify security vulnerabilities and best practices.

7. Implement access controls to prevent unauthorized access to your S3 bucket. You can use AWS Organizations to manage access across multiple AWS accounts and services, and also use Amazon S3 Access Points to control access to specific parts of your S3 bucket.

By following these steps, you can ensure that your S3 bucket is highly secure and your data is protected from unauthorized access and data breaches.
```
