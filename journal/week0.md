# Week 0 — Billing and Architecture

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

![Cruddur Conceptual Design](assets/Cruddur%20Conceptual%20Design.png)

### Recreate Logical Architectual Diagram in Lucid Charts:

- [Cruddur Logical Design (link LucidChart)](https://lucid.app/lucidchart/4b0ac743-a6a2-40e0-8863-0a8696174374/edit?viewport_loc=-444%2C91%2C2219%2C1121%2C0_0&invitationId=inv_ccf4c6ac-ff3a-4afd-9685-c37b2a2c7f07)

![Cruddur Logical Design](assets/Cruddur%20Logical%20Design.png)

### Create an Admin User
### Generate AWS Credentials

I've created another account for not using the management account in AWS.

![Management Account](assets/Management%20Account.png)

I've created an IAM admin user for not using the account root user.

- [IAM Console](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/home)
- Create new user alvarezdaniel inside my account alias
- Console access enabled
- Assign user group called Admin with PolicyName=AdministratorAccess

![IAM Dashboard](assets/IAM%20Dashboard.png)

![Admin User](assets/Admin%20User.png)

I've configured an access key for using from AWS CLI.

![Access Key](assets/Access%20Key.png)

### Use CloudShell

I've opened AWS Cloudshell, configured cli auto prompt and retrieved current identity

![Cloud Shell](assets/Cloud%20Shell.png)

### Installed AWS CLI

I've installed Chrome Gitpod extension, and the Gitpod button is showing in my github repo:
https://chrome.google.com/webstore/detail/gitpod-always-ready-to-co/dodmmooeoklaejobgleioelladacbeki

![Gitpod button](assets/Gitpod%20button.png)

I've opened Gitpod terminal and checked that AWS CLI is not installed:

![AWS not found](assets/AWS%20not%20found.png)

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

![AWS installed](assets/AWS%20installed.png)

I've checked that I don't have AWS access configured in gitpod, by running:

![AWS not configured](assets/AWS%20not%20configured.png)

I've configured AWS CLI environment variables in gitpod so it can connect to AWS:

![AWS configure](assets/AWS%20configure.png)

I've added gitpod environment variables so they are configured in each workspace:

![Gitpod env vars](assets/Gitpod%20env%20vars.png)

I've launched a new Gitpod workspace to verify if AWS CLI credentials are available, but AWS CLI is not configured successfully (must investigate and ask question in Discord):

![AWS not configured 2](assets/AWS%20not%20configured%202.png)

I've found that I've set AWS_ACCESS_KEY_ID env var with a wrong name (AWS_ACCESS_KEY instead of AWS_ACCESS_KEY_ID). I've changed it and now it's working:

https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html

![Gitpod env vars 2](assets/Gitpod%20env%20vars%202.png)

![AWS configured](assets/AWS%20configured.png)

### Create a Billing Alarm

In my root account, I've enabled Billing alerts so as to receive alerts in my email.

![Receive Billing Alerts](assets/Receive%20Billing%20Alerts.png)

Before creating the alarm, an SNS topic is needed:

https://docs.aws.amazon.com/cli/latest/reference/sns/create-topic.html

```bash
aws sns create-topic --name billing-alarm
```

![Create SNS Topic](assets/Create%20SNS%20Topic.png)

Then create a subscription, using the returned TopicARN and my email to receive the notification there:

```bash
aws sns subscribe \
    --topic-arn TOPIC_ARN \
    --protocol email \
    --notification-endpoint EMAIL
```

![Create Subscription](assets/Create%20Subscription.png)

The result is "Pending confirmation", so I've checked my email and confirmed it so as to complete the process:

![Confirm subscription](assets/Confirm%20subscription.png)

![Subscription confirmed](assets/Subcription%20confirmed.png)


