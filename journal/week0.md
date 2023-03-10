# Week 0 — Billing and Architecture

## Prerequisite Technologies

According to bootcamp outline, I've registered accounts in the following services:

- GitHub (I already had an account on this, only created the new repo using the template Andrew provided)
- Gitpod (I've created a free Gitpod account, and installed Chrome extension so I can use the Gitpod button from Github repo)
- Github Codespaces (I've created a free account so as to use if Gitpod is not enough)
- AWS (I already had an AWS account, but it was not completely registered, so I entered my credit card and I had to call for support to finish registration process because the automated phone call was not working for me)
- Momento (I've created a free tier Momento account)
- Custom domain name (I've already had a domain free for using in this bootcamp, so my idea is to use it)
- Lucid Charts (I've already had an account opened in Lucid Charts, so I'm using it for the bootcamp)
- HoneyComb.io (I've created a free-tier account)
- Rollbar (I've created a free-tier account)


## Watched videos:

Week 0 - Live Streamed Video - [Free AWS Cloud Project Bootcamp - Week 0 - Billing and Architecture](https://www.youtube.com/watch?v=SG8blanhAOg)
- Andrew Brown [@andrewbrown](https://twitter.com/andrewbrown)
- Chris Williams [@mistwire](https://twitter.com/mistwire)
- Shala Warner [@GiftedLane](https://twitter.com/GiftedLane)
- Margaret Valtierra [@MargaretValtie](https://twitter.com/MargaretValtie)

Chirag's Week 0 - Spend Considerations - [AWS Bootcamp Week 0 - Pricing Basics and Free tier](https://www.youtube.com/watch?v=OVw3RrlP-sI)
- Chirag Nayyar [@chiragnayyar](https://twitter.com/chiragnayyar)

Ashish's Week 0 - Security Considerations - [AWS Organizations & AWS IAM Tutorial For Beginners - Cloud BootCamp - Week 0](https://www.youtube.com/watch?v=4EMWBYVggQI)
- Ashish Rajan [@hashishrajan](https://twitter.com/hashishrajan)

Week 0 – Generate Credentials, AWS CLI, Budget and Billing Alarm via CLI - [Week 0 - Generate Credentials, AWS CLI, Budget and Billing Alarm via CLI](https://www.youtube.com/watch?v=OdUnNuKylHg)
- Andrew Brown [@andrewbrown](https://twitter.com/andrewbrown)


## Required Homework

### Recreate Conceptual Diagram in Lucid Charts or on a Napkin:

- [Cruddur Conceptual Design (link LucidChart)](https://lucid.app/lucidchart/6c79322a-15d5-45f1-b85e-171b0a30c4f3/edit?viewport_loc=160%2C284%2C1664%2C841%2C0_0&invitationId=inv_f43dd37f-191f-4e80-b444-1eadccd61383)

![Cruddur Conceptual Design](assets/week-0/Cruddur%20Conceptual%20Design.png)

### Recreate Logical Architectual Diagram in Lucid Charts:

- [Cruddur Logical Design (link LucidChart)](https://lucid.app/lucidchart/4b0ac743-a6a2-40e0-8863-0a8696174374/edit?viewport_loc=-444%2C91%2C2219%2C1121%2C0_0&invitationId=inv_ccf4c6ac-ff3a-4afd-9685-c37b2a2c7f07)

![Cruddur Logical Design](assets/week-0/Cruddur%20Logical%20Design.png)

### Create an Admin User
### Generate AWS Credentials

I've created another account for not using the management account in AWS.

![Management Account](assets/week-0/Management%20Account.png)

I've created an IAM admin user for not using the account root user.

- [IAM Console](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/home)
- Create new user alvarezdaniel inside my account alias
- Console access enabled
- Assign user group called Admin with PolicyName=AdministratorAccess

![IAM Dashboard](assets/week-0/IAM%20Dashboard.png)

![Admin User](assets/week-0/Admin%20User.png)

I've configured an access key for using from AWS CLI.

![Access Key](assets/week-0/Access%20Key.png)

### Use CloudShell

I've opened AWS Cloudshell, configured cli auto prompt and retrieved current identity

![Cloud Shell](assets/week-0/Cloud%20Shell.png)

### Installed AWS CLI

I've installed Chrome Gitpod extension, and the Gitpod button is showing in my github repo:
https://chrome.google.com/webstore/detail/gitpod-always-ready-to-co/dodmmooeoklaejobgleioelladacbeki

![Gitpod button](assets/week-0/Gitpod%20button.png)

I've opened Gitpod terminal and checked that AWS CLI is not installed:

![AWS not found](assets/week-0/AWS%20not%20found.png)

I've configured **.gitpod.yml** file so when it starts, AWS CLI is installed automatically. Also, partial autoprompt mode is set for using autocomplete in AWS CLI commands.

```
tasks:
  - name: aws-cli
    env:
      AWS_CLI_AUTO_PROMPT: on-partial
    init: |
      cd /workspace
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
      cd $THEIA_WORKSPACE_ROOT
```

I've opened gitpod terminal again and checked that AWS CLI was installed, and I can use it:

![AWS installed](assets/week-0/AWS%20installed.png)

I've checked that I don't have AWS access configured in gitpod, by running:

![AWS not configured](assets/week-0/AWS%20not%20configured.png)

I've configured AWS CLI environment variables in gitpod so it can connect to AWS:

![AWS configure](assets/week-0/AWS%20configure.png)

I've added gitpod environment variables so they are configured in each workspace:

![Gitpod env vars](assets/week-0/Gitpod%20env%20vars.png)

I've launched a new Gitpod workspace to verify if AWS CLI credentials are available, but AWS CLI is not configured successfully (must investigate and ask question in Discord):

![AWS not configured 2](assets/week-0/AWS%20not%20configured%202.png)

I've found that I've set AWS_ACCESS_KEY_ID env var with a wrong name (AWS_ACCESS_KEY instead of AWS_ACCESS_KEY_ID). I've changed it and now it's working:

https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html

![Gitpod env vars 2](assets/week-0/Gitpod%20env%20vars%202.png)

![AWS configured](assets/week-0/AWS%20configured.png)

### Create a Billing Alarm

In my root account, I've enabled Billing alerts so as to receive alerts in my email.

![Receive Billing Alerts](assets/week-0/Receive%20Billing%20Alerts.png)

Before creating the alarm, an SNS topic is needed:

https://docs.aws.amazon.com/cli/latest/reference/sns/create-topic.html

```bash
aws sns create-topic --name billing-alarm
```

![Create SNS Topic](assets/week-0/Create%20SNS%20Topic.png)

Then create a subscription, using the returned TopicARN and my email to receive the notification there:

```bash
aws sns subscribe \
    --topic-arn TOPIC_ARN \
    --protocol email \
    --notification-endpoint EMAIL
```

![Create Subscription](assets/week-0/Create%20Subscription.png)

The result is "Pending confirmation", so I've checked my email and confirmed it so as to complete the process:

![Confirm subscription](assets/week-0/Confirm%20subscription.png)

![Subscription confirmed](assets/week-0/Subcription%20confirmed.png)

I can see the topic and subscription in AWS console:

![SNS Topic](assets/week-0/SNS%20Topic.png)

Now I'm able to create the billing alarm, by using AWS CLI and a configuration json file. I'm replacing with the previously created topic name.

```json
{
  "AlarmName": "DailyEstimatedCharges",
  "AlarmDescription": "This alarm would be triggered if the daily estimated charges exceeds 1$",
  "ActionsEnabled": true,
  "AlarmActions": [
      "arn:aws:sns:<REGION>:<ACCOUNT_ID>:<SNS_TOPIC_NAME>"
  ],
  "EvaluationPeriods": 1,
  "DatapointsToAlarm": 1,
  "Threshold": 1,
  "ComparisonOperator": "GreaterThanOrEqualToThreshold",
  "TreatMissingData": "breaching",
  "Metrics": [{
      "Id": "m1",
      "MetricStat": {
          "Metric": {
              "Namespace": "AWS/Billing",
              "MetricName": "EstimatedCharges",
              "Dimensions": [{
                  "Name": "Currency",
                  "Value": "USD"
              }]
          },
          "Period": 86400,
          "Stat": "Maximum"
      },
      "ReturnData": false
  },
  {
      "Id": "e1",
      "Expression": "IF(RATE(m1)>0,RATE(m1)*86400,0)",
      "Label": "DailyEstimatedCharges",
      "ReturnData": true
  }]
}
```

Some info:
https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html
https://aws.amazon.com/premiumsupport/knowledge-center/cloudwatch-estimatedcharges-alarm/


Run I run this command to create the alarm:

```bash
aws cloudwatch put-metric-alarm --cli-input-json file://journal/aws/json/alarm_config.json
```

![Create alarm](assets/week-0/Create%20alarm.png)

The alarm was created, so I can check it in AWS console, in CloudWatch:

![Created alarm](assets/week-0/Created%20alarm.png)

### Create a Budget

Info:
https://docs.aws.amazon.com/cli/latest/reference/budgets/create-budget.html

To create a budget, the Account ID must be retrieved:

```
aws sts get-caller-identity --query Account --output text
```

![Get Account ID](assets/week-0/Get%20Account%20ID.png)

After that, AWS CLI is called passing some json files that should be configured with required parameters for creating the budget:

budget.json
```json
{
  "BudgetLimit": {
      "Amount": "100",
      "Unit": "USD"
  },
  "BudgetName": "Example Tag Budget",
  "BudgetType": "COST",
  "CostFilters": {
      "TagKeyValue": [
          "user:Key$value1",
          "user:Key$value2"
      ]
  },
  "CostTypes": {
      "IncludeCredit": true,
      "IncludeDiscount": true,
      "IncludeOtherSubscription": true,
      "IncludeRecurring": true,
      "IncludeRefund": true,
      "IncludeSubscription": true,
      "IncludeSupport": true,
      "IncludeTax": true,
      "IncludeUpfront": true,
      "UseBlended": false
  },
  "TimePeriod": {
      "Start": 1477958399,
      "End": 3706473600
  },
  "TimeUnit": "MONTHLY"
}
```

budget-notifications-with-subscribers.json
```json
[
  {
      "Notification": {
          "ComparisonOperator": "GREATER_THAN",
          "NotificationType": "ACTUAL",
          "Threshold": 80,
          "ThresholdType": "PERCENTAGE"
      },
      "Subscribers": [
          {
              "Address": "example@example.com",
              "SubscriptionType": "EMAIL"
          }
      ]
  }
]
```

I've executed the following command to create the budget, after replacing the required params in json file for my email and required amount for the budget:

```bash
aws budgets create-budget \
    --account-id AccountID \
    --budget file://journal/aws/json/budget.json \
    --notifications-with-subscribers file://journal/aws/json/budget-notifications-with-subscribers.json
```

![Create budget](assets/week-0/Create%20budget.png)

Then, the budget can be inspected from AWS console:

![Budget](assets/week-0/Budget.png)

