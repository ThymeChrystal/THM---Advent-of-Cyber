# Day 7: Oh, no! I'M SPEAKING IN CLOUDTRAIL!

## Background story
> The below is copied from the day 7 description.

*As SOC-mas approached, so did the need,*<br>
*To provide those without, with something to read.*<br>
*Care4Wares tried, they made it their mission,*<br>
*A gift for all wares, a SOC-mas tradition.*

*Although they had some, they still needed more,*<br>
*To pick up some books, they’d head to the store.*<br>
*The town’s favourite books, would no doubt make them jolly,*<br>
*They ticked off the list, as they filled up the trolley.*

*With the last book ticked off, the shopping was done,*<br>
*When asked for their card, the ware handed them one.*<br>
*“I’m sorry” he said, as the shop clerk reclined,*<br>
*“I can’t sell you these books, as your card has declined.”*<br>

*The ware put them back, as they walked in confusion,*<br>
*How could this be? An attack? An intrusion?*<br>
*And when they logged on, the ware got a scare,*<br>
*To find the donations, they just weren’t there!*

## Monitoring in an AWS environment
The challenge page provides a brief overview of AWS and CloudWatch and CloudTrail. I'm not going to repeat the same information here. You can refer to the challenge page for more details.

## Intro to JQ
The challenge page provides a brief overview of JQ and how it can be used to filter and manipulate JSON data. I'm not going to repeat the same information here. You can refer to the challenge page for more details.

## The Peculiar Case of Care4Wares' Dry Funds
> This section provides some information about the scenario. It is copied from the challenge page.
The responsible ware for the Care4Wares charity drive gave us the following info regarding this incident:

*We sent out a link on the 28th of November to everyone in our network that points to a flyer with the details of our charity. The details include the account number to receive donations. We received many donations the first day after sending out the link, but there were none from the second day on. I talked to multiple people who claimed to have donated a respectable sum. One showed his transaction, and I noticed the account number was wrong. I checked the link, and it was still the same. I opened the link, and the digital flyer was the same except for the account number.*

McSkidy recalls putting the digital flyer, wareville-bank-account-qr.png, in an Amazon AWS S3 bucket named wareville-care4wares. Let’s assist McSkidy and start by finding out more about that link. Before that, let’s first review the information that we currently have to start the investigation:

* The day after the link was sent out, several donations were received.
* Since the second day after sending the link, no more donations have been received.
* A donator has shown proof of his transaction. It was made 3 days after he received the link. The account number in the transaction was not correct.
* McSkidy put the digital flyer in the AWS S3 object named `wareville-bank-account-qr.png` under the bucket `wareville-care4wares`.
* The link has not been altered.

## Connection details
There is a button on the challenge page to start a machine. This opens in a split screen mode, so you can use the machine in the web browser. There are no other connection details provided, so we need to use this machine in the browser to complete the challenge.

> Note: By clicking on the two arrows in the bottom left corner of the machine, you can open it in its own page. This makes it easier to work with the machine and read the challenge text. Once opened in it's own page, you can click the `-` on the tool bar at the botton to exit split screen.

## Glitch Did It
The challenge text provides a log for the `wareville-care4wares` bucket. The log is in JSON format and contains the following information:
```json
{
  "eventVersion": "1.10",
  "userIdentity": {
    "type": "IAMUser",
    "principalId": "AIDAXRMKYT5O5Y2GLD4ZG",
    "arn": "arn:aws:iam::518371450717:user/wareville_collector",
    "accountId": "518371450717",
    "accessKeyId": "AKIAXRMKYT5OZCZPGNZ7",
    "userName": "wareville_collector"
  },
  "eventTime": "2024-10-21T22:13:24Z",
  "eventSource": "s3.amazonaws.com",
  "eventName": "ListObjects",
  "awsRegion": "ap-southeast-1",
  "sourceIPAddress": "34.247.218.56",
  "userAgent": "[aws-sdk-go/0.24.0 (go1.22.6; linux; amd64)]",
  "requestParameters": {
    "bucketName": "aoc-cloudtrail-wareville",
    "Host": "aoc-cloudtrail-wareville.s3.ap-southeast-1.amazonaws.com",
    "prefix": ""
  },
  "responseElements": null,
  "additionalEventData": {
    "SignatureVersion": "SigV4",
    "CipherSuite": "TLS_AES_128_GCM_SHA256",
    "bytesTransferredIn": 0,
    "AuthenticationMethod": "AuthHeader",
    "x-amz-id-2": "yqniVtqBrL0jNyGlvnYeR3BvJJPlXdgxvjAwwWhTt9dLMbhgZugkhlH8H21Oo5kNLiq8vg5vLoj3BNl9LPEAqN5iHpKpZ1hVynQi7qrIDk0=",
    "bytesTransferredOut": 236375
  },
  "requestID": "YKEKJP7QX32B4NZB",
  "eventID": "fd80529f-d0af-4f44-8034-743d8d92bdcf",
  "readOnly": true,
  "resources": [
    {
      "type": "AWS::S3::Object",
      "ARNPrefix": "arn:aws:s3:::aoc-cloudtrail-wareville/"
    },
    {
      "accountId": "518371450717",
      "type": "AWS::S3::Bucket",
      "ARN": "arn:aws:s3:::aoc-cloudtrail-wareville"
    }
  ],
  "eventType": "AwsApiCall",
  "managementEvent": false,
  "recipientAccountId": "518371450717",
  "eventCategory": "Data",
  "tlsDetails": {
    "tlsVersion": "TLSv1.3",
    "cipherSuite": "TLS_AES_128_GCM_SHA256",
    "clientProvidedHostHeader": "aoc-cloudtrail-wareville.s3.ap-southeast-1.amazonaws.com"
  }
}
```
The challenge provides and explanation of the log fields to focus on:
- `userIdentity`
- `eventTime`
- `eventType`
- `eventSource`
- `eventName`
- `sourceIPAddress`
- `userAgent`
- `requestParameters`

The above log entry can therefore be interpreted as:
- The user `wareville_collector` made a request to list objects in the `aoc-cloudtrail-wareville` bucket.
- The request was made on the 21st of October, 2024 at 22:13:24 UTC.
- The request was made from the IP address `34.247.218.56`.
- The request was made using the `aws-sdk-go/0.24.0 (go1.22.6; linux; amd64)` user agent (AWS SDK tool for go).

Now we can use JQ to filter the log entries to find the information we need.

On the machine, start a terminal and use the following commands:

![Show the files](images/show-files-c4w-logs.png)


We're going to focus on the `cloudtrail_log.json` file.

We can use the `jq` command to filter the log entries:
```bash
jq -r '.Records[] | select(.eventSource == "s3.amazonaws.com" and .requestParameters.bucketName=="wareville-care4wares")' cloudtrail_log.json
```
This will list quite a lot of entries, but they will specifically be for `s3.amazon.com` queries, and the `wareville-care4wares` bucket.

The `Record` part at the start of the query is used as that's the top level key in the JSON file, as you can see in the image below:

![JSON file structure](images/first-characters-json.png)

We can continue to further filter the results by adding more pipes to select specific fields. For example, to get the `eventTime`, `eventName`, `sourceIPAddress`, plus `bucketName` and `key` from the `requestParameters`, we can use the following command:
```bash
 jq -r '.Records[] | select(.eventSource == "s3.amazonaws.com" and .requestParameters.bucketName=="wareville-care4wares") | [.eventTime, .eventName, .userIdentity.userName // "N/A",.requestParameters.bucketName // "N/A", .requestParameters.key // "N/A", .sourceIPAddress // "N/A"]' cloudtrail_log.json
 ```
 Here the `// "N/A"` is used to provide a default value if the field is not present in the log entry.

 The output from this command is:
 ```plaintext
 [
  "2024-11-28T15:22:23Z",
  "ListObjects",
  "glitch",
  "wareville-care4wares",
  "N/A",
  "53.94.201.69"
]
[
  "2024-11-28T15:22:25Z",
  "ListObjects",
  "glitch",
  "wareville-care4wares",
  "N/A",
  "53.94.201.69"
]
[
  "2024-11-28T15:22:39Z",
  "PutObject",
  "glitch",
  "wareville-care4wares",
  "bank-details/wareville-bank-account-qr.png",
  "53.94.201.69"
]
[
  "2024-11-28T15:22:39Z",
  "PreflightRequest",
  "N/A",
  "wareville-care4wares",
  "bank-details/wareville-bank-account-qr.png",
  "53.94.201.69"
]
[
  "2024-11-28T15:22:44Z",
  "ListObjects",
  "glitch",
  "wareville-care4wares",
  "N/A",
  "53.94.201.69"
]
```
This can still be a bit hard to read, so we can use the `column` command to format the output. Note we also need the `@tsv` option in the `jq` command to output the data in tab-separated format:
```bash
jq -r '.Records[] | select(.eventSource == "s3.amazonaws.com" and .requestParameters.bucketName=="wareville-care4wares") | [.eventTime, .eventName, .userIdentity.userName // "N/A",.requestParameters.bucketName // "N/A", .requestParameters.key // "N/A", .sourceIPAddress // "N/A"] | @tsv' cloudtrail_log.json | column -t -s $'\t'
```
This will give us a nicely formatted output:
```plaintext
2024-11-28T15:22:23Z  ListObjects       glitch  wareville-care4wares  N/A                                         53.94.201.69
2024-11-28T15:22:25Z  ListObjects       glitch  wareville-care4wares  N/A                                         53.94.201.69
2024-11-28T15:22:39Z  PutObject         glitch  wareville-care4wares  bank-details/wareville-bank-account-qr.png  53.94.201.69
2024-11-28T15:22:39Z  PreflightRequest  N/A     wareville-care4wares  bank-details/wareville-bank-account-qr.png  53.94.201.69
2024-11-28T15:22:44Z  ListObjects       glitch  wareville-care4wares  N/A                                         53.94.201.69
```

This output can give is a lot of useful information:
* The number of log entries for the `wareville-care4wares` bucket.
* The time of the requests.
* What actions were taken on the bucket.
* The user who intiated the requests.
* That it was our bucket that was accessed.
* Any specific files that were accessed or modified.
* The IP address of the user who made the requests.

We can see that the `glitch` user made a `PutObject` request to the `wareville-care4wares` bucket to upload the `wareville-bank-account-qr.png` file. This is the file that was modified to change the account number.

> The challenge adds some extra info from McSkidy, copied here.
McSkidy is sure there was no user glitch in the system before. There is no one in the city hall with that name, either. The only person that McSkidy knows with that name is the hacker who keeps to himself. McSkidy suggests that we look into this anomalous user.

## McSkidy Fooled Us?
We can see when the user `glitch` was created, and what activity they have carried out by looking at the CloudTrail logs. We can use the following command to list all activity by the `glitch` user:
```bash
jq -r '["Event_Time", "Event_Source", "Event_Name", "User_Name", "Source_IP"],(.Records[] | select(.userIdentity.userName == "glitch") | [.eventTime, .eventSource, .eventName, .userIdentity.userName // "N/A", .sourceIPAddress // "N/A"]) | @tsv' cloudtrail_log.json | column -t -s $'\t'
```

The first part of this command ([`Event_Time`, `Event_Source`, `Event_Name`, `User_Name`, `Source_IP`]) is used to provide column headers for the output. The rest of the command is similar to before, but the initial filter only selects entries where the `userName` is `glitch`. The output from this command is:
```plaintext
Event_Time            Event_Source                         Event_Name                           User_Name  Source_IP
2024-11-28T15:22:12Z  s3.amazonaws.com                     HeadBucket                           glitch     53.94.201.69
2024-11-28T15:22:23Z  s3.amazonaws.com                     ListObjects                          glitch     53.94.201.69
2024-11-28T15:22:25Z  s3.amazonaws.com                     ListObjects                          glitch     53.94.201.69
2024-11-28T15:22:39Z  s3.amazonaws.com                     PutObject                            glitch     53.94.201.69
2024-11-28T15:22:44Z  s3.amazonaws.com                     ListObjects                          glitch     53.94.201.69
2024-11-28T15:21:54Z  signin.amazonaws.com                 ConsoleLogin                         glitch     53.94.201.69
2024-11-28T15:21:57Z  ce.amazonaws.com                     GetCostAndUsage                      glitch     53.94.201.69
2024-11-28T15:21:57Z  cost-optimization-hub.amazonaws.com  ListEnrollmentStatuses               glitch     53.94.201.69
2024-11-28T15:21:57Z  health.amazonaws.com                 DescribeEventAggregates              glitch     53.94.201.69
2024-11-28T15:22:12Z  s3.amazonaws.com                     ListBuckets                          glitch     53.94.201.69
2024-11-28T15:22:14Z  s3.amazonaws.com                     GetStorageLensConfiguration          glitch     AWS Internal
2024-11-28T15:22:14Z  s3.amazonaws.com                     GetStorageLensDashboardDataInternal  glitch     AWS Internal
2024-11-28T15:22:13Z  s3.amazonaws.com                     GetStorageLensDashboardDataInternal  glitch     AWS Internal
2024-11-28T15:21:57Z  health.amazonaws.com                 DescribeEventAggregates              glitch     53.94.201.69
2024-11-28T15:21:57Z  ce.amazonaws.com                     GetCostAndUsage                      glitch     53.94.201.69
```

The `ConsoleLogin` event shows that the `glitch` user logged into the AWS console using a browser. 

We want information about the tools and OS used by the `glitch` user. We can use the following command to get this information:
```bash
jq -r '["Event_Time", "Event_Source", "Event_Name", "User_Name", "Source_IP", "User_Agent"],(.Records[] | select(.userIdentity.userName == "glitch") | [.eventTime, .eventSource, .eventName, .userIdentity.userName // "N/A", .sourceIPAddress // "N/A", .userAgent // "N/A"]) | @tsv' cloudtrail_log.json | column -t -s $'\t'
```

The output from this command is:
```plaintext
Event_Time            Event_Source                         Event_Name                           User_Name  Source_IP     User_Agent
2024-11-28T15:22:12Z  s3.amazonaws.com                     HeadBucket                           glitch     53.94.201.69  [S3Console/0.4, aws-internal/3 aws-sdk-java/1.12.750 Linux/5.10.226-192.879.amzn2int.x86_64 OpenJDK_64-Bit_Server_VM/25.412-b09 java/1.8.0_412 vendor/Oracle_Corporation cfg/retry-mode/standard]
2024-11-28T15:22:23Z  s3.amazonaws.com                     ListObjects                          glitch     53.94.201.69  [S3Console/0.4, aws-internal/3 aws-sdk-java/1.12.750 Linux/5.10.226-192.879.amzn2int.x86_64 OpenJDK_64-Bit_Server_VM/25.412-b09 java/1.8.0_412 vendor/Oracle_Corporation cfg/retry-mode/standard]
2024-11-28T15:22:25Z  s3.amazonaws.com                     ListObjects                          glitch     53.94.201.69  [S3Console/0.4, aws-internal/3 aws-sdk-java/1.12.750 Linux/5.10.226-192.879.amzn2int.x86_64 OpenJDK_64-Bit_Server_VM/25.412-b09 java/1.8.0_412 vendor/Oracle_Corporation cfg/retry-mode/standard]
2024-11-28T15:22:39Z  s3.amazonaws.com                     PutObject                            glitch     53.94.201.69  [Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36]
2024-11-28T15:22:44Z  s3.amazonaws.com                     ListObjects                          glitch     53.94.201.69  [S3Console/0.4, aws-internal/3 aws-sdk-java/1.12.750 Linux/5.10.226-193.880.amzn2int.x86_64 OpenJDK_64-Bit_Server_VM/25.412-b09 java/1.8.0_412 vendor/Oracle_Corporation cfg/retry-mode/standard]
2024-11-28T15:21:54Z  signin.amazonaws.com                 ConsoleLogin                         glitch     53.94.201.69  Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36
2024-11-28T15:21:57Z  ce.amazonaws.com                     GetCostAndUsage                      glitch     53.94.201.69  Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36
2024-11-28T15:21:57Z  cost-optimization-hub.amazonaws.com  ListEnrollmentStatuses               glitch     53.94.201.69  Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36
2024-11-28T15:21:57Z  health.amazonaws.com                 DescribeEventAggregates              glitch     53.94.201.69  Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36
2024-11-28T15:22:12Z  s3.amazonaws.com                     ListBuckets                          glitch     53.94.201.69  [S3Console/0.4, aws-internal/3 aws-sdk-java/1.12.750 Linux/5.10.226-193.880.amzn2int.x86_64 OpenJDK_64-Bit_Server_VM/25.412-b09 java/1.8.0_412 vendor/Oracle_Corporation cfg/retry-mode/standard]
2024-11-28T15:22:14Z  s3.amazonaws.com                     GetStorageLensConfiguration          glitch     AWS Internal  AWS Internal
2024-11-28T15:22:14Z  s3.amazonaws.com                     GetStorageLensDashboardDataInternal  glitch     AWS Internal  AWS Internal
2024-11-28T15:22:13Z  s3.amazonaws.com                     GetStorageLensDashboardDataInternal  glitch     AWS Internal  AWS Internal
2024-11-28T15:21:57Z  health.amazonaws.com                 DescribeEventAggregates              glitch     53.94.201.69  Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36
2024-11-28T15:21:57Z  ce.amazonaws.com                     GetCostAndUsage                      glitch     53.94.201.69  Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36
```

This shows 2 different user agents used by the `glitch` user:
- The `S3Console` user agent is used for the `HeadBucket`, `ListObjects`, and `ListBuckets` events. This is internal AWS tooling.
- The `Mozilla/5.0` user agent is used for the `PutObject` and `ConsoleLogin` events. This is a browser user agent. It is a Google Chrome user agent on a Mac OS X system.

This may or may not be useful information, as it can be spoofed, but we can't dismiss it.

Next we want to find out who created the `glitch` user. We can use the following command to get this information:
```bash
jq -r '["Event_Time", "Event_Source", "Event_Name", "User_Name", "Source_IP"], (.Records[] | select(.eventSource == "iam.amazonaws.com") | [.eventTime, .eventSource, .eventName, .userIdentity.userName // "N/A", .sourceIPAddress // "N/A"]) | @tsv' cloudtrail_log.json | column -t -s $'\t'
```

The output from this command is:
```plaintext
Event_Time            Event_Source       Event_Name          User_Name  Source_IP
2024-11-28T15:21:26Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:29Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:30Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:30Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:30Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:30Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:25Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:33Z  iam.amazonaws.com  GetPolicy           mcskidy    53.94.201.69
2024-11-28T15:21:33Z  iam.amazonaws.com  GetPolicy           mcskidy    53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:33Z  iam.amazonaws.com  GetPolicy           mcskidy    53.94.201.69
2024-11-28T15:21:33Z  iam.amazonaws.com  GetPolicy           mcskidy    53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:36Z  iam.amazonaws.com  CreateLoginProfile  mcskidy    53.94.201.69
2024-11-28T15:21:36Z  iam.amazonaws.com  AttachUserPolicy    mcskidy    53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com  ListPolicies        mcskidy    53.94.201.69
2024-11-28T15:21:44Z  iam.amazonaws.com  ListUsers           mcskidy    53.94.201.69
2024-11-28T15:21:35Z  iam.amazonaws.com  CreateUser          mcskidy    53.94.201.69
```

This shows that there is a `CreateUser` event, but we don't know the user name created. We can also see an `AttachUserPolicy` event.

The IP address used by the `mcskidy` user is the same as the `glitch` user.

We can get the full record for the `CreateUser` event to see the user name created:
```bash
jq -r '.Records[] | select(.eventSource == "iam.amazonaws.com" and .eventName == "CreateUser")' cloudtrail_log.json
```

The record is as follows:
```plaintext
{
  "eventVersion": "1.10",
  "userIdentity": {
    "type": "IAMUser",
    "principalId": "AIDAXRMKYT5O6Z6AZBXU6",
    "arn": "arn:aws:iam::518371450717:user/mcskidy",
    "accountId": "518371450717",
    "accessKeyId": "ASIAXRMKYT5OVOMUJU3P",
    "userName": "mcskidy",
    "sessionContext": {
      "attributes": {
        "creationDate": "2024-11-28T15:20:54Z",
        "mfaAuthenticated": "false"
      }
    }
  },
  "eventTime": "2024-11-28T15:21:35Z",
  "eventSource": "iam.amazonaws.com",
  "eventName": "CreateUser",
  "awsRegion": "ap-southeast-1",
  "sourceIPAddress": "53.94.201.69",
  "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36",
  "requestParameters": {
    "userName": "glitch"
  },
  "responseElements": {
    "user": {
      "path": "/",
      "userName": "glitch",
      "userId": "AIDAXRMKYT5O7SKYSEJBQ",
      "arn": "arn:aws:iam::518371450717:user/glitch",
      "createDate": "Oct 22, 2024 3:21:35 PM"
    }
  },
  "requestID": "415e0a96-f1b6-429a-9cac-1c921c0b85f5",
  "eventID": "64dd59fc-c1b1-4f2d-b15c-b005911f1de4",
  "readOnly": false,
  "eventType": "AwsApiCall",
  "managementEvent": true,
  "recipientAccountId": "518371450717",
  "eventCategory": "Management",
  "tlsDetails": {
    "tlsVersion": "TLSv1.3",
    "cipherSuite": "TLS_AES_128_GCM_SHA256",
    "clientProvidedHostHeader": "iam.amazonaws.com"
  },
  "sessionCredentialFromConsole": "true"
}
```

We can see that this event created the `glitch` user, and that user was created by the `mcskidy` user.

Now we can see what policies were attached to the `glitch` user:
```bash
jq -r '.Records[] | select(.eventSource == "iam.amazonaws.com" and .eventName == "AttachUserPolicy")' cloudtrail_log.json
```

The record for the `AttachUserPolicy` event is:
```plaintext
{
  "eventVersion": "1.10",
  "userIdentity": {
    "type": "IAMUser",
    "principalId": "AIDAXRMKYT5O6Z6AZBXU6",
    "arn": "arn:aws:iam::518371450717:user/mcskidy",
    "accountId": "518371450717",
    "accessKeyId": "ASIAXRMKYT5OVOMUJU3P",
    "userName": "mcskidy",
    "sessionContext": {
      "attributes": {
        "creationDate": "2024-11-28T15:20:54Z",
        "mfaAuthenticated": "false"
      }
    }
  },
  "eventTime": "2024-11-28T15:21:36Z",
  "eventSource": "iam.amazonaws.com",
  "eventName": "AttachUserPolicy",
  "awsRegion": "ap-southeast-1",
  "sourceIPAddress": "53.94.201.69",
  "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36",
  "requestParameters": {
    "userName": "glitch",
    "policyArn": "arn:aws:iam::aws:policy/AdministratorAccess"
  },
  "responseElements": null,
  "requestID": "975d9d18-27d5-48a8-a882-a8b58b3a7173",
  "eventID": "08799cc5-535f-4d58-8373-d0b2cbb73a52",
  "readOnly": false,
  "eventType": "AwsApiCall",
  "managementEvent": true,
  "recipientAccountId": "518371450717",
  "eventCategory": "Management",
  "tlsDetails": {
    "tlsVersion": "TLSv1.3",
    "cipherSuite": "TLS_AES_128_GCM_SHA256",
    "clientProvidedHostHeader": "iam.amazonaws.com"
  },
  "sessionCredentialFromConsole": "true"
}
```

This shows that the `glitch` user was attached to the `AdministratorAccess` policy.

> The challenge adds some extra info from McSkidy, copied here.
McSkidy is baffled by these results. She knows that she did not create the anomalous user and did not assign the privileged access. She also doesn’t recognise the IP address involved in the events and does not use a Mac OS; she only uses a Windows machine. All this information is different to the typical IP address and machine used by McSkidy, so she wants to prove her innocence and asks to continue the investigation.

## Logs Don't Lie
We can use the logs to prove McSkidy's innocence. We can use the following command to get all the logs for the suspicious IP address:
```bash
jq -r '["Event_Time", "Event_Source", "Event_Name", "User_Name", "Source_IP"], (.Records[] | select(.sourceIPAddress == "53.94.201.69") | [.eventTime, .eventSource, .eventName, .userIdentity.userName // "N/A", .sourceIPAddress // "N/A"]) | @tsv' cloudtrail_log.json | column -t -s $'\t'
```

The output from this command is:
```plaintext
Event_Time            Event_Source                         Event_Name               User_Name      Source_IP
2024-11-28T15:20:38Z  s3.amazonaws.com                     HeadBucket               mayor_malware  53.94.201.69
2024-11-28T15:22:12Z  s3.amazonaws.com                     HeadBucket               glitch         53.94.201.69
2024-11-28T15:22:23Z  s3.amazonaws.com                     ListObjects              glitch         53.94.201.69
2024-11-28T15:22:25Z  s3.amazonaws.com                     ListObjects              glitch         53.94.201.69
2024-11-28T15:22:39Z  s3.amazonaws.com                     PutObject                glitch         53.94.201.69
2024-11-28T15:22:39Z  s3.amazonaws.com                     PreflightRequest         N/A            53.94.201.69
2024-11-28T15:22:44Z  s3.amazonaws.com                     ListObjects              glitch         53.94.201.69
2024-11-28T15:18:37Z  signin.amazonaws.com                 ConsoleLogin             mayor_malware  53.94.201.69
2024-11-28T15:20:54Z  signin.amazonaws.com                 ConsoleLogin             mcskidy        53.94.201.69
2024-11-28T15:21:54Z  signin.amazonaws.com                 ConsoleLogin             glitch         53.94.201.69
2024-11-28T15:21:26Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:29Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:30Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:30Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:30Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:30Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:25Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:33Z  iam.amazonaws.com                    GetPolicy                mcskidy        53.94.201.69
2024-11-28T15:21:33Z  iam.amazonaws.com                    GetPolicy                mcskidy        53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:33Z  iam.amazonaws.com                    GetPolicy                mcskidy        53.94.201.69
2024-11-28T15:21:33Z  iam.amazonaws.com                    GetPolicy                mcskidy        53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:36Z  iam.amazonaws.com                    CreateLoginProfile       mcskidy        53.94.201.69
2024-11-28T15:21:36Z  iam.amazonaws.com                    AttachUserPolicy         mcskidy        53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com                    ListPolicies             mcskidy        53.94.201.69
2024-11-28T15:21:44Z  iam.amazonaws.com                    ListUsers                mcskidy        53.94.201.69
2024-11-28T15:21:35Z  iam.amazonaws.com                    CreateUser               mcskidy        53.94.201.69
2024-11-28T15:21:45Z  organizations.amazonaws.com          DescribeOrganization     mcskidy        53.94.201.69
2024-11-28T15:21:57Z  ce.amazonaws.com                     GetCostAndUsage          glitch         53.94.201.69
2024-11-28T15:21:57Z  cost-optimization-hub.amazonaws.com  ListEnrollmentStatuses   glitch         53.94.201.69
2024-11-28T15:21:57Z  health.amazonaws.com                 DescribeEventAggregates  glitch         53.94.201.69
2024-11-28T15:22:12Z  s3.amazonaws.com                     ListBuckets              glitch         53.94.201.69
2024-11-28T15:21:57Z  health.amazonaws.com                 DescribeEventAggregates  glitch         53.94.201.69
2024-11-28T15:21:57Z  ce.amazonaws.com                     GetCostAndUsage          glitch         53.94.201.69
2024-11-22T11:08:03Z  signin.amazonaws.com                 ConsoleLogin             mayor_malware  53.94.201.69
2024-11-23T07:19:01Z  signin.amazonaws.com                 ConsoleLogin             mayor_malware  53.94.201.69
2024-11-24T02:28:17Z  signin.amazonaws.com                 ConsoleLogin             mayor_malware  53.94.201.69
2024-11-25T21:48:22Z  signin.amazonaws.com                 ConsoleLogin             mayor_malware  53.94.201.69
2024-11-26T22:55:51Z  signin.amazonaws.com                 ConsoleLogin             mayor_malware  53.94.201.69
```

This shows that three different users have logged in from the suspicious IP address: `mayor_malware`, `mcskidy`, and `glitch`.

Now we can look at each user and see if they always use the same IP address:
```bash
jq -r '["Event_Time", "Event_Source", "Event_Name", "User_Name", "Source_IP"], (.Records[] | select(.userIdentity.userName == "mayor_malware") | [.eventTime, .eventSource, .eventName, .userIdentity.userName // "N/A", .sourceIPAddress // "N/A"]) | @tsv' cloudtrail_log.json | column -t -s $'\t'
```

The output for the `mayor_malware` user is:
```plaintext
Event_Time            Event_Source          Event_Name    User_Name      Source_IP
2024-11-28T15:20:38Z  s3.amazonaws.com      HeadBucket    mayor_malware  53.94.201.69
2024-11-28T15:18:37Z  signin.amazonaws.com  ConsoleLogin  mayor_malware  53.94.201.69
2024-11-22T11:08:03Z  signin.amazonaws.com  ConsoleLogin  mayor_malware  53.94.201.69
2024-11-23T07:19:01Z  signin.amazonaws.com  ConsoleLogin  mayor_malware  53.94.201.69
2024-11-24T02:28:17Z  signin.amazonaws.com  ConsoleLogin  mayor_malware  53.94.201.69
2024-11-25T21:48:22Z  signin.amazonaws.com  ConsoleLogin  mayor_malware  53.94.201.69
2024-11-26T22:55:51Z  signin.amazonaws.com  ConsoleLogin  mayor_malware  53.94.201.69
```

The output for the `mcskidy` user is:
```plaintext
Event_Time            Event_Source                 Event_Name            User_Name  Source_IP
2024-11-28T15:20:54Z  signin.amazonaws.com         ConsoleLogin          mcskidy    53.94.201.69
2024-11-28T15:21:26Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:29Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:30Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:30Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:30Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:30Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:25Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:33Z  iam.amazonaws.com            GetPolicy             mcskidy    53.94.201.69
2024-11-28T15:21:33Z  iam.amazonaws.com            GetPolicy             mcskidy    53.94.201.69
2024-11-28T15:21:31Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:33Z  iam.amazonaws.com            GetPolicy             mcskidy    53.94.201.69
2024-11-28T15:21:33Z  iam.amazonaws.com            GetPolicy             mcskidy    53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:36Z  iam.amazonaws.com            CreateLoginProfile    mcskidy    53.94.201.69
2024-11-28T15:21:36Z  iam.amazonaws.com            AttachUserPolicy      mcskidy    53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:32Z  iam.amazonaws.com            ListPolicies          mcskidy    53.94.201.69
2024-11-28T15:21:44Z  iam.amazonaws.com            ListUsers             mcskidy    53.94.201.69
2024-11-28T15:21:35Z  iam.amazonaws.com            CreateUser            mcskidy    53.94.201.69
2024-11-28T15:21:45Z  organizations.amazonaws.com  DescribeOrganization  mcskidy    53.94.201.69
2024-11-22T12:20:54Z  signin.amazonaws.com         ConsoleLogin          mcskidy    31.210.15.79
2024-11-23T07:15:54Z  signin.amazonaws.com         ConsoleLogin          mcskidy    31.210.15.79
2024-11-24T05:19:31Z  signin.amazonaws.com         ConsoleLogin          mcskidy    31.210.15.79
2024-11-25T01:11:32Z  signin.amazonaws.com         ConsoleLogin          mcskidy    31.210.15.79
2024-11-26T19:22:05Z  signin.amazonaws.com         ConsoleLogin          mcskidy    31.210.15.79
```

And the output for the `glitch` user is:
```plaintext
Event_Time            Event_Source                         Event_Name                           User_Name  Source_IP
2024-11-28T15:22:12Z  s3.amazonaws.com                     HeadBucket                           glitch     53.94.201.69
2024-11-28T15:22:23Z  s3.amazonaws.com                     ListObjects                          glitch     53.94.201.69
2024-11-28T15:22:25Z  s3.amazonaws.com                     ListObjects                          glitch     53.94.201.69
2024-11-28T15:22:39Z  s3.amazonaws.com                     PutObject                            glitch     53.94.201.69
2024-11-28T15:22:44Z  s3.amazonaws.com                     ListObjects                          glitch     53.94.201.69
2024-11-28T15:21:54Z  signin.amazonaws.com                 ConsoleLogin                         glitch     53.94.201.69
2024-11-28T15:21:57Z  ce.amazonaws.com                     GetCostAndUsage                      glitch     53.94.201.69
2024-11-28T15:21:57Z  cost-optimization-hub.amazonaws.com  ListEnrollmentStatuses               glitch     53.94.201.69
2024-11-28T15:21:57Z  health.amazonaws.com                 DescribeEventAggregates              glitch     53.94.201.69
2024-11-28T15:22:12Z  s3.amazonaws.com                     ListBuckets                          glitch     53.94.201.69
2024-11-28T15:22:14Z  s3.amazonaws.com                     GetStorageLensConfiguration          glitch     AWS Internal
2024-11-28T15:22:14Z  s3.amazonaws.com                     GetStorageLensDashboardDataInternal  glitch     AWS Internal
2024-11-28T15:22:13Z  s3.amazonaws.com                     GetStorageLensDashboardDataInternal  glitch     AWS Internal
2024-11-28T15:21:57Z  health.amazonaws.com                 DescribeEventAggregates              glitch     53.94.201.69
2024-11-28T15:21:57Z  ce.amazonaws.com                     GetCostAndUsage                      glitch     53.94.201.69
```

We can see that the `mayor_malware` and `glitch` users have always used the same IP address, but the `mcskidy` user has used a different IP address in the past. Because of this, we can conclude that McSkidy is innocent, and that the IP address used by the `mcskidy` user was spoofed by the `mayor_malware` user to create the `glitch` user and attach the `AdministratorAccess` policy.

We could also look at the user agents used by each user to help backup the IP address evidence. I did this, but won't include it here as this is getting long enough and the IP address is all we need to meet the challenge requirements.

## Definitive Evidence
The Wareville Bank have provided some logs to us from their Amazon Relational Database Service (RDS). These are collected from CloudWatch rather than CloudTrail, so are not in JSON format. The logs are in the `rds.log` file.

The challenge text tells us that we can use the following command to search for transactions within the logs:
```bash
grep "INSERT" rds.log
```

We can see by looking through the output from the above command, that the account receiving the funds changed from `Care4Wares Fund` to `Mayor Malware` on the 28th of November. This is the same day and time that the `glitch` user was created and used to change the account information:

```plaintext
...
024-11-28T15:22:17.728Z 2024-11-28T15:22:17.728648Z	  263 Query	INSERT INTO wareville_bank_transactions (account_number, account_owner, amount) VALUES ('8839 2219 1329 6917', 'Care4wares Fund', 342.80)
2024-11-28T15:22:18.569Z 2024-11-28T15:22:18.569279Z	  263 Query	INSERT INTO wareville_bank_transactions (account_number, account_owner, amount) VALUES ('8839 2219 1329 6917', 'Care4wares Fund', 929.57)
2024-11-28T15:23:02.605Z 2024-11-28T15:23:02.605700Z	  263 Query	INSERT INTO wareville_bank_transactions (account_number, account_owner, amount) VALUES ('2394 6912 7723 1294', 'Mayor Malware', 193.45)
2024-11-28T15:23:02.792Z 2024-11-28T15:23:02.792161Z	  263 Query	INSERT INTO wareville_bank_transactions (account_number, account_owner, amount) VALUES ('2394 6912 7723 1294', 'Mayor Malware', 998.13)
...
```

This is definitive evidence that Mayor Malware was involved in the fraudulent activity.

## Answes to the questions
> Note: This section contains the answers to the questions asked in the challenge. If you don't want spoilers, don't read this section.

### What is the other activity made by the user glitch?
The full question is: `What is the other activity made by the user glitch aside from the ListObject action?`

If we look at the result of the `jq` command for the intial query to get activity on the `wareville-care4wares` bucket, we can see the following line:
```plaintext
2024-11-28T15:22:39Z  PutObject         glitch  wareville-care4wares  bank-details/wareville-bank-account-qr.png  53.94.201.69
```

This is the only line in that initial query that wasn't a `ListObjects` action, and was taken by the `glitch` user. The answer to this question is: `PutObject`.

### What is the source IP?
The full question is: `What is the source IP related to the S3 bucket activities of the user glitch?`

If we look at the result of the `jq` command for the intial query we can see the IP address used for all actions was: `53.94.201.69`. That answers this question.

### What AWS service generates the ConsoleLogin event?
The full question is: `Based on the eventSource field, what AWS service generates the ConsoleLogin event?`

Our second `jq` query had the following line which showed the console login event:
```plaintext
2024-11-28T15:21:54Z  signin.amazonaws.com                 ConsoleLogin                         glitch     53.94.201.69
```

Here we can see the service used is `signin.amazonaws.com`.

### When did the anamalous user trigger the ConsoleLogin event?
The full question is: `When did the anomalous user trigger the ConsoleLogin event?`

If we loog at the line used to answer the previous question, we can see the date and time of the console login event was: `2024-11-28T15:21:54Z`.

### What was the name of the user created by the mcskidy user?
The full question is: `What was the name of the user created by the mcskidy user?`

If we look at the result of the `jq` command to get the full record for the `CreateUser` event, we can see the following entry:
```plaintext
"responseElements": {
    "user": {
      "path": "/",
      "userName": "glitch",
      "userId": "AIDAXRMKYT5O7SKYSEJBQ",
      "arn": "arn:aws:iam::518371450717:user/glitch",
      "createDate": "Oct 22, 2024 3:21:35 PM"
    }
```

This shows the user name being created was: `glitch`.

### What type of assigned to the anomalous user?
The full question is: `What type of policy was assigned to the anomalous user?`

If we look at the result of the `jq` command to get the full record for the `AttachUserPolicy` event, we can see the following entry:
```plaintext
"requestParameters": {
    "userName": "glitch",
    "policyArn": "arn:aws:iam::aws:policy/AdministratorAccess"
  }
```

This shows the policy assigned to the `glitch` user was: `AdministratorAccess`.

### Which IP does Mayor Malware typically use to log into AWS?
The full question is: `Which IP does Mayor Malware typically use to log into AWS?`

If we look at the result of the `jq` queries looking at the activity of each user in turn, we can see that the IP address used by the `mayor_malware` user was: `53.94.201.69`.

### What is McSkidy's actual IP address?
The full question is: `What is McSkidy's actual IP address?`

If we look at the result of the `jq` queries looking at the activity of each user in turn, we can see that the user `mcskidy`, before the anomalous activity, used the IP address: `31.210.15.79`.

### What is the bank account number owned by Mayor Malware?
The full question is: `What is the bank account number owned by Mayor Malware?`

If we look at the result of the `grep` command to search for `INSERT` transactions in the `rds.log` file, we can see the following entry:
```plaintext
2024-11-28T15:23:02.605Z 2024-11-28T15:23:02.605700Z	  263 Query	INSERT INTO wareville_bank_transactions (account_number, account_owner, amount) VALUES ('2394 6912 7723 1294', 'Mayor Malware', 193.45)
```

The account number is given in this line as: `2394 6912 7723 1294`. This is the answer to the question.

### Remaining questions
The remaining questions only require you to click `Complete` and have no specific answers to find.


