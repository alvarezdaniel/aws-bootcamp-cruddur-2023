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

### Watch Ashish's Week 3 - Decentralized Authentication

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

### Implement Custom Signin Page

For implementing cognito in frontend app we are going to use AWS Amplify:

https://aws.amazon.com/amplify/

AWS Amplify is a complete solution that lets frontend web and mobile developers easily build, ship, and host full-stack applications on AWS, with the flexibility to leverage the breadth of AWS services as use cases evolve. No cloud expertise needed.

There is also a js library for cognito:

https://www.npmjs.com/package/amazon-cognito-identity-js

Amplify Dev Center
https://docs.amplify.aws/lib/auth/getting-started/q/platform/js/

The first thing to do is install amplify, inside `frontend-react-js` directory:

```sh
npm i aws-amplify --save
```

> --save: Saves dependency for development and runtime
> --save-dev: Saves dependency only for development

```
npm WARN deprecated resolve-url@0.2.1: https://github.com/lydell/resolve-url#deprecated
npm WARN deprecated source-map-url@0.4.1: See https://github.com/lydell/source-map-url#deprecated
npm WARN deprecated urix@0.1.0: Please see https://github.com/lydell/urix#deprecated
npm WARN deprecated source-map-resolve@0.5.3: See https://github.com/lydell/source-map-resolve#deprecated
npm WARN deprecated querystring@0.2.0: The querystring API is considered Legacy. new code should use the URLSearchParams API instead.
npm WARN deprecated uglify-es@3.3.9: support for ECMAScript is superseded by `uglify-js` as of v3.13.0
npm WARN deprecated uuid@3.4.0: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.
npm WARN deprecated uuid@3.4.0: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.
npm WARN deprecated uuid@3.4.0: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.
npm WARN deprecated @aws-amplify/xr@4.0.15: The Amazon Sumerian service is no longer accepting new customers. Existing customer scenes will not be available after February 21, 2023. The AWS Amplify XR features depend on the Amazon Sumerian service to function and as a result, will no longer be available.
npm WARN deprecated uuid@3.4.0: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.
npm WARN deprecated uuid@3.4.0: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.
npm WARN deprecated uuid@3.4.0: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.
npm WARN deprecated uuid@3.4.0: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.
npm WARN deprecated uuid@3.4.0: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.
npm WARN deprecated uuid@3.4.0: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.

added 1178 packages, and audited 2650 packages in 24s

243 packages are looking for funding
  run `npm fund` for details

8 high severity vulnerabilities

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.
```

After doing this, a new dependency will be added to `package.json` file

![](assets/week-3/12-add-amplify.png)

> AWS Amplify is the equivalent of gcp Firebase: https://firebase.google.com/firebase-and-gcp/

Next step is configuring Amplify (We need to hook up our cognito pool to our code in the `App.js`)

> We are not using Amplify UI: https://ui.docs.amplify.aws/, https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-app-integration.html

app.js

```js
import { Amplify } from 'aws-amplify';
```

The next step is configuring it (below import)

```js
Amplify.configure({
  "AWS_PROJECT_REGION": process.env.REACT_APP_AWS_PROJECT_REGION,
  "aws_cognito_region": process.env.REACT_APP_AWS_COGNITO_REGION,
  "aws_user_pools_id": process.env.REACT_APP_AWS_USER_POOLS_ID,
  "aws_user_pools_web_client_id": process.env.REACT_APP_CLIENT_ID,
  "oauth": {},
  Auth: {
    // We are not using an Identity Pool
    // identityPoolId: process.env.REACT_APP_IDENTITY_POOL_ID, // REQUIRED - Amazon Cognito Identity Pool ID
    region: process.env.REACT_APP_AWS_PROJECT_REGION,           // REQUIRED - Amazon Cognito Region
    userPoolId: process.env.REACT_APP_AWS_USER_POOLS_ID,         // OPTIONAL - Amazon Cognito User Pool ID
    userPoolWebClientId: process.env.REACT_APP_CLIENT_ID,   // OPTIONAL - Amazon Cognito Web Client ID (26-char alphanumeric string)
  }
});
```

The references variables should be added also to `docker-compose.yml` file, using the User Pool ID and Client ID set in Cognito User Pool (using env variables set in Gitpod, Andrew give me a plus 25 as you've mentioned in the video ;)

```sh
export REACT_APP_AWS_USER_POOLS_ID="MY_USER_POOL_ID"
gp env REACT_APP_AWS_USER_POOLS_ID="MY_USER_POOL_ID"

export REACT_APP_CLIENT_ID="MY_CLIENT_ID"
gp env REACT_APP_CLIENT_ID="MY_CLIENT_ID"
```

```yml
version: "3.8"
services:
  frontend-react-js:
    environment:
      REACT_APP_AWS_PROJECT_REGION: "${AWS_DEFAULT_REGION}"
      REACT_APP_AWS_COGNITO_REGION: "${AWS_DEFAULT_REGION}"
      REACT_APP_AWS_USER_POOLS_ID: "${REACT_APP_AWS_USER_POOLS_ID}"
      REACT_APP_CLIENT_ID: "${REACT_APP_CLIENT_ID}"
```

After configuring Amplify, we are going to conditionally show components based on login status (logged in or logget out)

`HomeFeedPage.js`

Change from:

```js
// Authentication
//import Cookies from 'js-cookie'
import { Auth } from 'aws-amplify';
```

Set a state

```js
const [user, setUser] = React.useState(null);
```

It's already set, so no change in here

```js
export default function HomeFeedPage() {
  const [activities, setActivities] = React.useState([]);
  const [popped, setPopped] = React.useState(false);
  const [poppedReply, setPoppedReply] = React.useState(false);
  const [replyActivity, setReplyActivity] = React.useState({});
  const [user, setUser] = React.useState(null);
  const dataFetchedRef = React.useRef(false);
```

Change checkAuth function, from the current implementation using cookies to a new one using cognito

```js
  // Authentication: check if we are authenticated
  /*
  const checkAuth = async () => {
    console.log('checkAuth')
    // [TODO] Authenication
    if (Cookies.get('user.logged_in')) {
      setUser({
        display_name: Cookies.get('user.name'),
        handle: Cookies.get('user.username')
      })
    }
  };
  */
  const checkAuth = async () => {
    Auth.currentAuthenticatedUser({
      // Optional, By default is false. 
      // If set to true, this call will send a 
      // request to Cognito to get the latest user data
      bypassCache: false 
    })
    .then((user) => {
      console.log('user',user);
      return Auth.currentAuthenticatedUser()
    }).then((cognito_user) => {
        setUser({
          display_name: cognito_user.attributes.name,
          handle: cognito_user.attributes.preferred_username
        })
    })
    .catch((err) => console.log(err));
  };
```

Finally, check when the page loads if we are authenticated (already implemented)

```js
React.useEffect(()=>{
  loadData();
  checkAuth();
}, [])
```

As user information is in Home page, it must be passed to other components (DesktopNavigation and DesktopSideBar) (these changes are already implemented)

```js
<DesktopNavigation user={user} active={'home'} setPopped={setPopped} />
<DesktopSidebar user={user} />
```

In `DesktopNavigation.js` we need to conditionally show or hide the links in the left hand column according to user login status (this is already implemented)

```js
import './DesktopNavigation.css';
import {ReactComponent as Logo} from './svg/logo.svg';
import DesktopNavigationLink from '../components/DesktopNavigationLink';
import CrudButton from '../components/CrudButton';
import ProfileInfo from '../components/ProfileInfo';

export default function DesktopNavigation(props) {

  let button;
  let profile;
  let notificationsLink;
  let messagesLink;
  let profileLink;
  if (props.user) {
    button = <CrudButton setPopped={props.setPopped} />;
    profile = <ProfileInfo user={props.user} />;
    notificationsLink = <DesktopNavigationLink 
      url="/notifications" 
      name="Notifications" 
      handle="notifications" 
      active={props.active} />;
    messagesLink = <DesktopNavigationLink 
      url="/messages"
      name="Messages"
      handle="messages" 
      active={props.active} />
    profileLink = <DesktopNavigationLink 
      url="/@andrewbrown" 
      name="Profile"
      handle="profile"
      active={props.active} />
  }

  return (
    <nav>
      <Logo className='logo' />
      <DesktopNavigationLink url="/" 
        name="Home"
        handle="home"
        active={props.active} />
      {notificationsLink}
      {messagesLink}
      {profileLink}
      <DesktopNavigationLink url="/#" 
        name="More" 
        handle="more"
        active={props.active} />
      {button}
      {profile}
    </nav>
  );
}
```

The same must be implemented in 'DesktopSidebar.js` (also already implemented, just a small change in if conditions)

```js
import './DesktopSidebar.css';
import Search from '../components/Search';
import TrendingSection from '../components/TrendingsSection'
import SuggestedUsersSection from '../components/SuggestedUsersSection'
import JoinSection from '../components/JoinSection'

export default function DesktopSidebar(props) {
  const trendings = [
    {"hashtag": "100DaysOfCloud", "count": 2053 },
    {"hashtag": "CloudProject", "count": 8253 },
    {"hashtag": "AWS", "count": 9053 },
    {"hashtag": "FreeWillyReboot", "count": 7753 }
  ]

  const users = [
    {"display_name": "Andrew Brown", "handle": "andrewbrown"}
  ]

  let trending;
  let suggested;
  let join;
  if (props.user) {
    trending = <TrendingSection trendings={trendings} />
    suggested = <SuggestedUsersSection users={users} />
  } else {
    join = <JoinSection />
  }

  return (
    <section>
      <Search />
      {trending}
      {suggested}
      {join}
      <footer>
        <a href="#">About</a>
        <a href="#">Terms of Service</a>
        <a href="#">Privacy Policy</a>
      </footer>
    </section>
  );
}
```

We need to implement a change in `ProfileInfo.js`

```js
// Authentication
//import Cookies from 'js-cookie'
import { Auth } from 'aws-amplify';

/*
const signOut = async () => {
console.log('signOut')
// [TODO] Authenication
Cookies.remove('user.logged_in')
//Cookies.remove('user.name')
//Cookies.remove('user.username')
//Cookies.remove('user.email')
//Cookies.remove('user.password')
//Cookies.remove('user.confirmation_code')
window.location.href = "/"
}
*/
const signOut = async () => {
try {
    await Auth.signOut({ global: true });
    window.location.href = "/"
} catch (error) {
    console.log('error signing out: ', error);
}
}  
```

> global: true indicates logout from all sessions


The next thing to do is the sign in page, for changing current cookies implementation with the one with cognito

`SigninPage.js`

```js
import { Auth } from 'aws-amplify';

const [cognitoErrors, setCognitoErrors] = React.useState('');

const onsubmit = async (event) => {
  setCognitoErrors('')
  event.preventDefault();
  try {
    Auth.signIn(username, password)
      .then(user => {
        localStorage.setItem("access_token", user.signInUserSession.accessToken.jwtToken)
        window.location.href = "/"
      })
      .catch(err => { console.log('Error!', err) });
  } catch (error) {
    if (error.code == 'UserNotConfirmedException') {
      window.location.href = "/confirm"
    }
    setCognitoErrors(error.message)
  }
  return false
}

let errors;
if (cognitoErrors){
  errors = <div className='errors'>{cognitoErrors}</div>;
}

// just before submit component
{errors}
```

```js
// Authentication
//import Cookies from 'js-cookie'
import { Auth } from 'aws-amplify';

  // Authentication
  /*
  const onsubmit = async (event) => {
    event.preventDefault();
    setErrors('')
    console.log('onsubmit')
    if (Cookies.get('user.email') === email && Cookies.get('user.password') === password){
      Cookies.set('user.logged_in', true)
      window.location.href = "/"
    } else {
      setErrors("Email and password is incorrect or account doesn't exist")
    }
    return false
  }
  */
  const onsubmit = async (event) => {
    setErrors('')
    event.preventDefault();
    Auth.signIn(email, password)
    .then(user => {
      console.log('user', user) // added for debug
      localStorage.setItem("access_token", user.signInUserSession.accessToken.jwtToken)
      window.location.href = "/"
    })
    .catch(error => { 
      if (error.code == 'UserNotConfirmedException') {
        window.location.href = "/confirm"
      }
      setErrors(error.message)
    });
    return false
  }
```

> console.log('user', user) is useful for debugging purposes

For testing the changes, we are going to create manually a user inside the user pool

![](assets/week-3/13-create-user.png)

![](assets/week-3/14-create-user-2.png)

![](assets/week-3/15-user-created.png)

The user is created, verified, but requires a password change in the first login. To avoid that, we need to execute the following command:

```sh
aws cognito-idp admin-set-user-password --user-pool-id <value> --username <email-address> --password <value> --permanent
```

```sh
aws cognito-idp admin-set-user-password --user-pool-id ca-central-1_iGtdWLWJr --username alvarez.daniel@gmail.com --password Password123! --permanent
```

> https://stackoverflow.com/questions/64720348/create-new-users-in-aws-cognito-without-a-temp-password-email

> https://docs.aws.amazon.com/cli/latest/reference/cognito-idp/admin-set-user-password.html#description

Now the user is verified and confirmed

![](assets/week-3/16-user-verified-and-confirmed.png)

It's time to test the changes. Let's start Gitpod workspace and launch compose file to start the required services

![](assets/week-3/17-compose-started.png)

If I try to sign-in using the created user with the correct password, the page is refreshed and shown as logged-in:

![](assets/week-3/18-signin-page.png)

![](assets/week-3/19-home-page-logged-in.png)

Also, as the user is logged into the console, the information can be shown there:

![](assets/week-3/20-log-user-console.png)

Signout button is also available in home page

![](assets/week-3/21-signout-button.png)

If I try to log in using a non existing user, an error message is shown

![](assets/week-3/22-invalid-user.png)

I have the same message if I try to use the valid user but with an invalid password

![](assets/week-3/23-invalid-password.png)

Now, with the page loaded with a valid login, information for that user should be shown in screen, but it's not

![](assets/week-3/24-user.png)

So we need to edit the user in AWS console and add the required attributes

![](assets/week-3/25-add-attributes.png)

![](assets/week-3/26-user-modified.png)

By signing out and in again, now this information is shown in 

![](assets/week-3/27-user-handle.png)

### Implement Custom Signup Page / Custom Confirmation Page

For implementing user signup, first of all we are going to get rid of the manually created user, so delete it from AWS Cognito console

![](assets/week-3/28-delete-user.png)

We'll begin implementing changes for SignUp in `SignupPage.js`, replacing cookies auth with cognito

```js
// Authentication
//import Cookies from 'js-cookie'
import { Auth } from 'aws-amplify';
```

```js
  // Authentication
  /*
  const onsubmit = async (event) => {
    event.preventDefault();
    console.log('SignupPage.onsubmit')
    
    Cookies.set('user.name', name)
    Cookies.set('user.username', username)
    Cookies.set('user.email', email)
    Cookies.set('user.password', password)
    Cookies.set('user.confirmation_code',1234)
    window.location.href = `/confirm?email=${email}`
    return false
  }
  */
  const onsubmit = async (event) => {
    event.preventDefault();
    setErrors('')

    console.log('SignupPage.onsubmit')
    console.log('username',username)
    console.log('email',email)
    console.log('name',name)

    try {
      const { user } = await Auth.signUp({
        username: email,
        password: password,
        attributes: {
          name: name,
          email: email,
          preferred_username: username,
        },
        autoSignIn: { // optional - enables auto sign in after user is confirmed
          enabled: true,
        }
      });
      console.log(user);
      window.location.href = `/confirm?email=${email}`
    } catch (error) {
        console.log(error);
        setErrors(error.message)
    }

    return false
  }
```

```js
  let el_errors;
  if (errors){
    el_errors = <div className='errors'>{errors}</div>;
  }
```

Also we need to change confirmation page `ConfirmationPage.js`

```js
// Authentication
//import Cookies from 'js-cookie'
import { Auth } from 'aws-amplify';
```

```js
  // Authentication
  /*
  const resend_code = async (event) => {
    console.log('resend_code')
  }
  */
  const resend_code = async (event) => {
    console.log('resend_code')
    setErrors('')
    try {
      await Auth.resendSignUp(email);
      console.log('code resent successfully');
      setCodeSent(true)
    } catch (err) {
      // does not return a code
      // does cognito always return english
      // for this to be an okay match?
      console.log(err)
      if (err.message == 'Username cannot be empty'){
        setErrors("You need to provide an email in order to send Resend Activiation Code")   
      } else if (err.message == "Username/client id combination not found."){
        setErrors("Email is invalid or cannot be found.")   
      }
    }    
  }

  // Authentication
  /*
  const onsubmit = async (event) => {
    event.preventDefault();
    console.log('ConfirmationPage.onsubmit')
    if (Cookies.get('user.email') === undefined || Cookies.get('user.email') === '' || Cookies.get('user.email') === null){
      setErrors("You need to provide an email in order to send Resend Activiation Code")   
    } else {
      if (Cookies.get('user.email') === email){
        if (Cookies.get('user.confirmation_code') === code){
          Cookies.set('user.logged_in',true)
          window.location.href = "/"
        } else {
          setErrors("Code is not valid")
        }
      } else {
        setErrors("Email is invalid or cannot be found.")   
      }
    }
    return false
  }
  */
  const onsubmit = async (event) => {
    event.preventDefault();
    console.log('ConfirmationPage.onsubmit')
    setErrors('')
    try {
      await Auth.confirmSignUp(email, code);
      window.location.href = "/"
    } catch (error) {
      setErrors(error.message)
    }
  }

  let el_errors;
  if (errors){
    el_errors = <div className='errors'>{errors}</div>;
  }
```

To test it, I'll run compose file and open frontend app, and then signup creating a new user with the same information that I created before manually

![](assets/week-3/29-signup-user.png)

After completing the new user registration, this user will appear in AWS Cognito console, but it will be Unconfirmed and Not verified.

![](assets/week-3/30-new-user.png)

An email should be received with the verification code to type in confirmation page

![](assets/week-3/31-verification-code.png)

![](assets/week-3/32-confirmation-page.png)

After confirming with the received code, the user will be Confirmed and Verified in AWS Cognito console

![](assets/week-3/33-user-confirmed.png)

And now it can be possible to log in to the app using the new created user

![](assets/week-3/34-signin-new-user.png)

![](assets/week-3/35-logged-in.png)

### Implement Custom Recovery Page

The last page to implement is the recovery page, used to generate a new password in case it is forgotten

`RecoverPage.js`

```js
// Authentication
import { Auth } from 'aws-amplify';
```

```js
  // Authentication
  /*
  const onsubmit_send_code = async (event) => {
    event.preventDefault();
    console.log('onsubmit_send_code')
    return false
  }
  */
  const onsubmit_send_code = async (event) => {
    event.preventDefault();
    setErrors('')
    Auth.forgotPassword(username)
    .then((data) => setFormState('confirm_code') )
    .catch((err) => setErrors(err.message) );
    return false
  }

  /*
  const onsubmit_confirm_code = async (event) => {
    event.preventDefault();
    console.log('onsubmit_confirm_code')
    return false
  }
  */
  const onsubmit_confirm_code = async (event) => {
    event.preventDefault();
    setErrors('')
    if (password == passwordAgain){
      Auth.forgotPasswordSubmit(username, code, password)
      .then((data) => setFormState('success'))
      .catch((err) => setErrors(err.message) );
    } else {
      setErrors('Passwords do not match')
    }
    return false
  }
```

For testing the change, we need to use the Forgot Password link in the signin page

![](assets/week-3/36-recover-password.png)

When doing that, an email should be received with a code for generating a new password

![](assets/week-3/37-password-reset-code.png)

![](assets/week-3/38-recover-password.png)

On successful operation, a message should indicate that the password has been reset and it is ready to be used to log in

![](assets/week-3/39-password-reset.png)


When signing in the system, an access token is retrieved and then saved into local storage to be used later in backend requests.

`SigninPage.js`

```js
  const onsubmit = async (event) => {
    setErrors('')
    event.preventDefault();
    Auth.signIn(email, password)
    .then(user => {
      console.log('user', user) // added for debug
      localStorage.setItem("access_token", user.signInUserSession.accessToken.jwtToken)
      window.location.href = "/"
    })
    .catch(error => { 
      if (error.code == 'UserNotConfirmedException') {
        window.location.href = "/confirm"
      }
      setErrors(error.message)
    });
    return false
  }
```

However, then signing out of the system, this access token is not cleared, so it should be implemented

`ProfileInfo.js`

```js
  const signOut = async () => {
    try {
        await Auth.signOut({ global: true });
        localStorage.removeItem("access_token") // clear access token
        window.location.href = "/"
    } catch (error) {
        console.log('error signing out: ', error);
    }
```

For implementing token verification in backend, first we need to pass the obtained access token to the backend calls in the request header

This should be done in `HomeFeedPage.js`

```js
  const loadData = async () => {
    try {
      const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/activities/home`
      const res = await fetch(backend_url, {
        // Authentication
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`
        },        
        method: "GET"
```

About changes in backend, we are going to use python module `Flask-AWSCognito`

> https://pypi.org/project/Flask-AWSCognito/

> https://flask-awscognito.readthedocs.io/en/latest/index.html

> https://github.com/cgauge/Flask-AWSCognito

We need to add it to `requirements.txt` file and install it

```txt
Flask-AWSCognito
```

```sh
pip install -r requirements.txt
```

We are going to copy code from https://github.com/cgauge/Flask-AWSCognito/blob/master/flask_awscognito/services/token_service.py into a new class in backend-flask app

`lib/cognito_jwt_token.py`

> We are renaming class name, from TokenService to CognitoJwtToken, and making some other changes

Original file

```js
import time
import requests
from jose import jwk, jwt
from jose.exceptions import JOSEError
from jose.utils import base64url_decode
from flask_awscognito.exceptions import FlaskAWSCognitoError, TokenVerifyError


class TokenService:
    def __init__(self, user_pool_id, user_pool_client_id, region, request_client=None):
        self.region = region
        if not self.region:
            raise FlaskAWSCognitoError("No AWS region provided")
        self.user_pool_id = user_pool_id
        self.user_pool_client_id = user_pool_client_id
        self.claims = None
        if not request_client:
            self.request_client = requests.get
        else:
            self.request_client = request_client
        self._load_jwk_keys()

    def _load_jwk_keys(self):
        keys_url = f"https://cognito-idp.{self.region}.amazonaws.com/{self.user_pool_id}/.well-known/jwks.json"
        try:
            response = self.request_client(keys_url)
            self.jwk_keys = response.json()["keys"]
        except requests.exceptions.RequestException as e:
            raise FlaskAWSCognitoError(str(e)) from e

    @staticmethod
    def _extract_headers(token):
        try:
            headers = jwt.get_unverified_headers(token)
            return headers
        except JOSEError as e:
            raise TokenVerifyError(str(e)) from e

    def _find_pkey(self, headers):
        kid = headers["kid"]
        # search for the kid in the downloaded public keys
        key_index = -1
        for i in range(len(self.jwk_keys)):
            if kid == self.jwk_keys[i]["kid"]:
                key_index = i
                break
        if key_index == -1:
            raise TokenVerifyError("Public key not found in jwks.json")
        return self.jwk_keys[key_index]

    @staticmethod
    def _verify_signature(token, pkey_data):
        try:
            # construct the public key
            public_key = jwk.construct(pkey_data)
        except JOSEError as e:
            raise TokenVerifyError(str(e)) from e
        # get the last two sections of the token,
        # message and signature (encoded in base64)
        message, encoded_signature = str(token).rsplit(".", 1)
        # decode the signature
        decoded_signature = base64url_decode(encoded_signature.encode("utf-8"))
        # verify the signature
        if not public_key.verify(message.encode("utf8"), decoded_signature):
            raise TokenVerifyError("Signature verification failed")

    @staticmethod
    def _extract_claims(token):
        try:
            claims = jwt.get_unverified_claims(token)
            return claims
        except JOSEError as e:
            raise TokenVerifyError(str(e)) from e

    @staticmethod
    def _check_expiration(claims, current_time):
        if not current_time:
            current_time = time.time()
        if current_time > claims["exp"]:
            raise TokenVerifyError("Token is expired")  # probably another exception

    def _check_audience(self, claims):
        # and the Audience  (use claims['client_id'] if verifying an access token)
        audience = claims["aud"] if "aud" in claims else claims["client_id"]
        if audience != self.user_pool_client_id:
            raise TokenVerifyError("Token was not issued for this audience")

    def verify(self, token, current_time=None):
        """ https://github.com/awslabs/aws-support-tools/blob/master/Cognito/decode-verify-jwt/decode-verify-jwt.py """
        if not token:
            raise TokenVerifyError("No token provided")

        headers = self._extract_headers(token)
        pkey_data = self._find_pkey(headers)
        self._verify_signature(token, pkey_data)

        claims = self._extract_claims(token)
        self._check_expiration(claims, current_time)
        self._check_audience(claims)

        self.claims = claims
```

Modified file

```js
import time
import requests
from jose import jwk, jwt
from jose.exceptions import JOSEError
from jose.utils import base64url_decode

class FlaskAWSCognitoError(Exception):
  pass

class TokenVerifyError(Exception):
  pass

def extract_access_token(request_headers):
    access_token = None
    auth_header = request_headers.get("Authorization")
    if auth_header and " " in auth_header:
        _, access_token = auth_header.split()
    return access_token

class CognitoJwtToken:
    def __init__(self, user_pool_id, user_pool_client_id, region, request_client=None):
        self.region = region
        if not self.region:
            raise FlaskAWSCognitoError("No AWS region provided")
        self.user_pool_id = user_pool_id
        self.user_pool_client_id = user_pool_client_id
        self.claims = None
        if not request_client:
            self.request_client = requests.get
        else:
            self.request_client = request_client
        self._load_jwk_keys()


    def _load_jwk_keys(self):
        keys_url = f"https://cognito-idp.{self.region}.amazonaws.com/{self.user_pool_id}/.well-known/jwks.json"
        try:
            response = self.request_client(keys_url)
            self.jwk_keys = response.json()["keys"]
        except requests.exceptions.RequestException as e:
            raise FlaskAWSCognitoError(str(e)) from e

    @staticmethod
    def _extract_headers(token):
        try:
            headers = jwt.get_unverified_headers(token)
            return headers
        except JOSEError as e:
            raise TokenVerifyError(str(e)) from e

    def _find_pkey(self, headers):
        kid = headers["kid"]
        # search for the kid in the downloaded public keys
        key_index = -1
        for i in range(len(self.jwk_keys)):
            if kid == self.jwk_keys[i]["kid"]:
                key_index = i
                break
        if key_index == -1:
            raise TokenVerifyError("Public key not found in jwks.json")
        return self.jwk_keys[key_index]

    @staticmethod
    def _verify_signature(token, pkey_data):
        try:
            # construct the public key
            public_key = jwk.construct(pkey_data)
        except JOSEError as e:
            raise TokenVerifyError(str(e)) from e
        # get the last two sections of the token,
        # message and signature (encoded in base64)
        message, encoded_signature = str(token).rsplit(".", 1)
        # decode the signature
        decoded_signature = base64url_decode(encoded_signature.encode("utf-8"))
        # verify the signature
        if not public_key.verify(message.encode("utf8"), decoded_signature):
            raise TokenVerifyError("Signature verification failed")

    @staticmethod
    def _extract_claims(token):
        try:
            claims = jwt.get_unverified_claims(token)
            return claims
        except JOSEError as e:
            raise TokenVerifyError(str(e)) from e

    @staticmethod
    def _check_expiration(claims, current_time):
        if not current_time:
            current_time = time.time()
        if current_time > claims["exp"]:
            raise TokenVerifyError("Token is expired")  # probably another exception

    def _check_audience(self, claims):
        # and the Audience  (use claims['client_id'] if verifying an access token)
        audience = claims["aud"] if "aud" in claims else claims["client_id"]
        if audience != self.user_pool_client_id:
            raise TokenVerifyError("Token was not issued for this audience")

    def verify(self, token, current_time=None):
        """ https://github.com/awslabs/aws-support-tools/blob/master/Cognito/decode-verify-jwt/decode-verify-jwt.py """
        if not token:
            raise TokenVerifyError("No token provided")

        headers = self._extract_headers(token)
        pkey_data = self._find_pkey(headers)
        self._verify_signature(token, pkey_data)

        claims = self._extract_claims(token)
        self._check_expiration(claims, current_time)
        self._check_audience(claims)

        self.claims = claims 
        return claims
```

In `app.py` we are going to use this class, by importing it (we need to import also sys module)

```py
# Authentication
import sys
```

```py
# Authentication
from lib.cognito_jwt_token import CognitoJwtToken, extract_access_token, TokenVerifyError
```

Then we need to configure the library by passing several new environment variables

```py
# Authentication
cognito_jwt_token = CognitoJwtToken(
  user_pool_id=os.getenv("AWS_COGNITO_USER_POOL_ID"), 
  user_pool_client_id=os.getenv("AWS_COGNITO_USER_POOL_CLIENT_ID"),
  region=os.getenv("AWS_DEFAULT_REGION")
)
```

For this configuration to work, we need to add the environment variables to compose file (the ones that are missing, because AWS_DEFAULT_REGION is already set)

```yml
version: "3.8"
services:
  backend-flask:
    environment:
      AWS_COGNITO_USER_POOL_ID: "${REACT_APP_AWS_USER_POOLS_ID}"
      AWS_COGNITO_USER_POOL_CLIENT_ID: "${REACT_APP_CLIENT_ID}"     
```

Then we need to change CORs configuration in `app.py` for not receiving cors errors while calling api

```py
cors = CORS(
  app, 
  resources={r"/api/*": {"origins": origins}},
  #expose_headers="location,link",
  #allow_headers="content-type,if-modified-since",
  headers=['Content-Type', 'Authorization'], 
  expose_headers='Authorization',  
  methods="OPTIONS,GET,HEAD,POST"
)
```

> Allow Authorization header

Finally, we are going to change /api/activities/home endpoint so as to verify the token in `app.py`

```py
# Authentication
#@app.route("/api/activities/home", methods=['GET'])
#@xray_recorder.capture('activities_home')
#def data_home():
#  data = HomeActivities.run(logger=LOGGER)
#  return data, 200
@app.route("/api/activities/home", methods=['GET'])
@xray_recorder.capture('activities_home')
def data_home():
  #app.logger.debug("AUTH TOKEN")
  #app.logger.debug(request.headers.get("Authorization"))
  access_token = extract_access_token(request.headers)
  try:
    claims = cognito_jwt_token.verify(access_token)
    # authenticated request
    app.logger.debug("authenticated")
    app.logger.debug(claims)
    app.logger.debug(claims['username'])
    data = HomeActivities.run(logger=LOGGER, cognito_user_id=claims['username'])
  except TokenVerifyError as e:
    # unauthenticated request
    app.logger.debug(e)
    app.logger.debug("unauthenticated")
    data = HomeActivities.run(logger=LOGGER)  
  return data, 200
```

As we are passing the authenticated user to HomeActivities, we need to change it as well (`services/home_activities.py`)

```py
# Authentication
class HomeActivities:
  #def run(logger):
  def run(logger, cognito_user_id=None):
    #logger.info('Hello Cloudwatch! from /api/activities/home')
```

And in case the user is authenticated, we will return an extra crud for verification

```py
      # Authentication
      if cognito_user_id != None:
        extra_crud = {
          'uuid': '248959df-3079-4947-b847-9e0892d1bab4',
          'handle':  'Lore',
          'message': 'My dear brother, it the humans that are the problem',
          'created_at': (now - timedelta(hours=1)).isoformat(),
          'expires_at': (now + timedelta(hours=12)).isoformat(),
          'likes': 1042,
          'replies': []
        }
        
        results.insert(0,extra_crud)

      span.set_attribute("app.result_length", len(results))
      
      return results
```

For testing this change, we'll start compose file and browse the application

When running unauthenticated, only 4 cruds will appear

![](assets/week-3/40-not-authenticated.png)

And the message `The user is not authenticated` will appear in console

![](assets/week-3/41-user-not-authenticated.png)

And a message will be shown in the log output

![](assets/week-3/45-log-error.png)

However, when sigin in using valid credentials, an extra crud will appear

![](assets/week-3/42-authenticated.png)

And the information for the logged in user will be written to the console

![](assets/week-3/43-user-authenticated.png)

Also, the obtained claims and username will be written to the log output

![](assets/week-3/44-log-claims.png)

### Watch about different approaches to verifying JWTs

https://www.youtube.com/watch?v=nJjbI4BbasU&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=43

Andrew mentioned in the video that we could try exploring other options for JWT token validation

This is the code ChatGPT is proposing to me

`write python code that will verify an access token from cognito and return claims`

Sure! Here's some sample code that you can use to verify an access token from Cognito and return its claims:

```py
import requests
import jwt

def verify_cognito_token(token, user_pool_id, region):
    """
    Verifies a Cognito access token and returns its claims.
    
    :param token: The access token to verify.
    :param user_pool_id: The ID of the user pool that issued the token.
    :param region: The region that the user pool is in.
    :return: The claims of the verified token if valid, False otherwise.
    """
    headers = {'Content-type': 'application/x-amz-json-1.1', 'Authorization': token}
    url = f'https://cognito-idp.{region}.amazonaws.com/{user_pool_id}/.well-known/jwks.json'
    r = requests.get(url, headers=headers)
    keys = r.json()['keys']

    # Verify token signature with each key
    for key in keys:
        kid = key['kid']
        kty = key['kty']
        alg = key['alg']
        n = key['n']
        e = key['e']

        rsa_key = {
            'alg': alg,
            'kty': kty,
            'n': n,
            'e': e,
            'kid': kid
        }

        try:
            payload = jwt.decode(token, rsa_key, algorithms=[alg])
            return payload
        except:
            pass

    return False
```

To use this function, simply call verify_cognito_token with the access token you want to verify, the ID of the user pool that issued it, and the region that the pool is in. The function will return the claims of the token if it is valid, or False if it is not.

Note that this code requires the requests and jwt Python packages to be installed. You can install them with pip install requests and pip install pyjwt, respectively.

### Improving UI contrast

Watching Andrew's video: https://www.youtube.com/watch?v=m9V4SmJWoJU&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=44, I've implemented the suggested changes to styles to improve UI contrast

CSS Frameworks

- Tailwind CSS (https://tailwindcss.com/)

After implementing the changes by Andrew (exctracted from https://github.com/omenking/aws-bootcamp-cruddur-2023/commit/d79a2370d142d68a51788a7a5b093b55ea8b4d7c#diff-d35c60b4ef7994ee8b4b4d81b4dff86eae0e85270aff4fc08baaa3c13b33d322), this is the result

![](assets/week-3/46-contrast-improved.png)

### Useful links

- https://www.linuxtek.ca/2023/03/07/aws-cloud-project-bootcamp-week-3-unofficial-homework-guide/

Additional Resources

Videos:

- [YouTube – ExamPro – Introduction to Cognito](https://www.youtube.com/watch?v=pp9ilsKWZho&feature=youtu.be)
- [YouTube – ExamPro – Cognito User Pools](https://www.youtube.com/watch?v=g2s_hJRI70U&feature=youtu.be)
- [YouTube – ExamPro – Cognito Follow-Along](https://www.youtube.com/watch?v=X-7dkSIYUrE&feature=youtu.be)
- [YouTube – ExamPro – Cognito Cheat Sheet](https://www.youtube.com/watch?v=PPdz1JI5iS0&feature=youtu.be)

> Note: These videos are about 3 years old, so some of the information and user interfaces may be out of date.

Documentation:

- [AWS Cognito Documentation](https://docs.aws.amazon.com/cognito/index.html)
- [OmenKing Github Notes for Week 3](https://github.com/omenking/aws-bootcamp-cruddur-2023/blob/week-3/journal/week3.md)
- [Frontend Code from Amplify JavaScript](https://docs.amplify.aws/lib/auth/emailpassword/q/platform/js/)
- [Amplify UI Documentation](https://ui.docs.amplify.aws/)

## Knowledge Challenges

### Security Quiz

Which additional AWS service should be enabled and monitored alongside Cognito to help detect malicious Cognito user behavior?

- RDS
- Cloud Trail
- Cloud Watch (X)
- DynamoDB

When it comes to single-sign-on, what does the acronym SAML stand for?

- Security Assertion Markup Language (X)
- Security Assumed Middleware Language
- Security Assertion Middleware Language
- Security Assumed Markup Language

Which AWS service should be used with Cognito to enable rate limiting, and setup allow/deny rule lists?

- Cloud Watch
- EC2 Load balancer
- WAF (X)
- Cloud Trail

Which of the following is NOT an industry standard for authentication and authorization?

- YAML (X)
- OAuth 2.0
- Open ID Connect
- SAML

Your Cognito deployment should only be in the AWS region which you are legally allowed to hold user data in

- TRUE (X)
- FALSE



