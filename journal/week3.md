# Week 3 — Decentralized Authentication

## Week 3 instructors

- Andrew Brown [@andrewbrown](https://twitter.com/andrewbrown)
- Shala Warner [@GiftedLane](https://twitter.com/GiftedLane)

## Class Summary

- Provision via ClickOps a Amazon Cognito User Pool
- Install and configure Amplify client-side library for Amazon Congito
- Implement API calls to Amazon Coginto for custom login, signup, recovery and forgot password page
- Show conditional elements and data based on logged in or logged out
- Verify JWT Token server side to serve authenticated API endpoints in Flask Application

## Todo Checklist

### Watch Ashish's Week 3 - Decenteralized Authentication

https://www.youtube.com/watch?v=tEJIeII66pY&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=39

### Watch Chirag Week 3 - Spending Considerations



### Setup Cognito User Pool

https://www.youtube.com/watch?v=9obl7rVgzJw&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=41

For creating the cognito user pool, go to AWS console and open Amazon Cognito, selecting the corresponding region (in this case ca-central-1)

https://ca-central-1.console.aws.amazon.com/cognito/v2/home?region=ca-central-1#

And then click on "Create user pool" button

![](assets/week-3/01-cognito.png)

Step 1: Configure sign-in experience

![](assets/week-3/02-step-1.png)

Step 2: Configure security requirements

![](assets/week-3/03-step-2-1.png)

![](assets/week-3/04-step-2-2.png)

Step 3: Configure sign-up experience

![](assets/week-3/05-step-3-1.png)

![](assets/week-3/06-step-3-2.png)

![](assets/week-3/07-step-3-3.png)

Step 4: Configure message delivery

![](assets/week-3/08-step-4.png)

Step 5: Integrate your app

![](assets/week-3/09-step-5-1.png)

![](assets/week-3/10-step-5-2.png)

Step 6: Review and create

![](assets/week-3/11-user-pool-created.png)



