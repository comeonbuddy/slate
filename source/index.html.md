---
title: Instant Access Partner - API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript: JavaScript
  - javascript--react: React
  - php: PHP

toc_footers:
  - <a href='http://www.instantaccess.io/partner'>Partner Sign Up</a>

search: true
---

# IA Documentation

<img src="images/headerImg.svg" width="100%"/>

Before you start, you will need 2 keys<br/>
- Client ID Key<br/>
- Client Secret Key<br/><br/>
Contact us at [yo@instantaccess.io](email:yo@instantaccess.io) if you do not have these. You will need to provide us with a callback url to receive user info from our server.


# IA Partner Integration

Once you are ready, simply add one of the following `code snippets` where you want IA button to appear.

<aside class="notice">
Please note: IA button will only work under the domain name you registered with us. If your dev environment is under a different domain, please contact us via our <a href="http://bit.ly/2yhKXXs" target="_blank">dev slack channel</a>
</aside>

<aside class="notice">
You must replace <code>IA_PARTNER_CLIENT_ID_KEY</code> with your IA partner client id key.
</aside>

<aside class="notice">
You must replace <code>IA_PARTNER_CLIENT_SECRET_KEY</code> with your IA partner client secret key.
</aside>

You can integrate the IA button for different two scenarios:
1- For signing up / signing in.
2- For collecting payment from your clients.

## For signing up / signing in.

> To integrate IA on your website for signin up or signing in, add this code snippet to your website:

```javascript
  <!-- IA Authentication -->
  <script type="text/javascript">
    window.onload = function(e){
      var client_id = 'IA_PARTNER_CLIENT_ID_KEY';
      var client_secret = 'IA_PARTNER_CLIENT_SECRET_KEY';

      function successIA(usernameOrEmail) {
        // Handle successLogin in IA
        // Example: window.location = 'IACompleteRegisteration.php?iaUser='+usernameOrEmail
      }

      function deniedIA(message) {
        alert(message);
      }

      function timeoutIA(message) {
        alert(message);
      }


      document.getElementById('ia-framework-login').src = 'https://dashboard.instantaccess.io/api/partner/authorize/view?client_id='+client_id+'&client_secret='+client_secret;
      //listen to ia back
      window.addEventListener('message',function(event) {
        if(event.origin !== 'https://dashboard.instantaccess.io') return;
        if(event.data.action === 'alert')
          alert(event.data.message);
        else if(event.data.action === 'closeframe') {
          document.getElementById("ia-btn").style.display = "block";
          fade(document.getElementById("ia-framework-login"));
          //document.getElementById("ia-framework-login").style.display = "none";
        }
        else if(event.data.action === 'auth') {
          switch (event.data.status) {
            case 'success':
              successIA(event.data.usernameOrEmail);
              break;
            case 'denied':
              deniedIA(event.data.message);
              break;
            case 'timeout':
              timeoutIA(event.data.message);
              break;
          }
        }
      },false);
    }

    function iaButtonClicked() {
      unfade(document.getElementById("ia-framework-login"));
      //document.getElementById("ia-framework-login").style.display = "block";
      document.getElementById("ia-btn").style.display = "hidden";
    }

    function fade(element) {
      var op = 1;  // initial opacity
      var timer = setInterval(function () {
        if (op <= 0.1){
          clearInterval(timer);
          element.style.display = 'none';
        }
        element.style.opacity = op;
        element.style.filter = 'alpha(opacity=' + op * 100 + ")";
        op -= op * 0.1;
      }, 10);
    }

    function unfade(element) {
      var op = 0.1;  // initial opacity
      element.style.display = 'block';
      var timer = setInterval(function () {
        if (op >= 1){
          clearInterval(timer);
        }
        element.style.opacity = op;
        element.style.filter = 'alpha(opacity=' + op * 100 + ")";
        op += op * 0.1;
      }, 5);
    }
  </script>
  <div style="display: inline-block; position: relative;">
    <button type="button" onclick="iaButtonClicked()" id="ia-btn" style="background-color: #4A4A4A; border-width: 0; padding: 0; width: 56px; height: 56px; border-radius:10px;">
      <img src="https://dashboard.instantaccess.io/upload/image/whitelogo.svg"/>
    </button>
    <iframe id="ia-framework-login" src="" width="170" height="215" frameBorder="0" style="display:none; border-radius:10px; z-index: 1000; position: absolute; left: 0px; bottom: 0px;"></iframe>
  </div>
  <style>
    #ia-btn:focus {
      outline-width: 0;
    }
    #ia-btn:hover {
      opacity: 0.9;
      cursor: pointer;
    }
  </style>
  <!-- End of login with IA -->
```

```javascript--react
> Step 1: IAButton React component source code - save this file as IAButton.js in your project

import React from "react";
import PropTypes from 'prop-types';

export default class IAButton extends React.Component {

    constructor (props) {
        super(props);
        this.state = {
            iaWhiteLogoSource: props.iaServer+'/upload/image/whitelogo.svg',
            iaFrameSource: props.iaServer+'/api/partner/authorize/view?client_id='+props.clientId+'&client_secret='+props.clientSecret+(props.payment?('&payment='+props.payment):''),
            iaButtonHovered: false,
            iaButtonFocused: false,
            iaButtonHidden: false,
            iaFrameDisplayed: false,
            iaFrameOpacity: 0.1,
            iaFrameFilter: 'alpha(opacity=0)'
        }
    }

    componentDidMount () {
        window.addEventListener("message", this.handleIAMessage);
    }

    componentWillUnmount () {
        window.removeEventListener("message", this.handleIAMessage);
    }

    handleIAMessage = (event) => {
        var eventOrigin = event.origin;
        if(!this.props.iaServer.toLowerCase().startsWith("http")) {
            if(eventOrigin.toLowerCase().startsWith("http:")) {
                eventOrigin = eventOrigin.substr(5);
            } else if(eventOrigin.toLowerCase().startsWith("https:")) {
                eventOrigin = eventOrigin.substr(6);
            }
        }
        if(eventOrigin !== this.props.iaServer) return;
        if(event.data.action === 'alert') {
            alert(event.data.message);
        } else if(event.data.action === 'closeframe') {
            this.setState({
                iaBtnHidden: false
            });
            this.fadeIAFrame();
        }
        else if(event.data.action === 'auth') {
            switch (event.data.status) {
                case 'success':
                    this.successIA(event.data.usernameOrEmail);
                    break;
                case 'denied':
                    this.deniedIA(event.data.message);
                    break;
                case 'timeout':
                    this.timeoutIA(event.data.message);
                    break;
            }
        }
    }

    successIA (usernameOrEmail) {
        // Handle successLogin in IA
        this.props.successIA(usernameOrEmail);
    }

    deniedIA (message) {
        if(this.props.notifyUser) {
            alert(message);
        }
        this.props.deniedIA(message);
    }

    timeoutIA (message) {
        if(this.props.notifyUser) {
            alert(message);
        }
        this.props.timeoutIA(message);
    }

    iaButtonClicked () {
        this.unfadeIAFrame();
        this.setState({
            iaBtnHidden: true
        });
    }

    fadeIAFrame() {
        var op = 1;  // initial opacity
        var timer = setInterval(() => {
            if (op <= 0.1){
                clearInterval(timer);
                this.setState({iaFrameDisplayed: false});
            }
            this.setState({
                iaFrameOpacity: op,
                iaFrameFilter: 'alpha(opacity=' + op * 100 + ')'
            });
            op -= op * 0.1;
        }, 10);
    }

    unfadeIAFrame () {
        var op = 0.1;  // initial opacity
        this.setState({iaFrameDisplayed: true});
        var timer = setInterval(() => {
            if (op >= 1){
                clearInterval(timer);
            }
            this.setState({
                iaFrameOpacity: op,
                iaFrameFilter: 'alpha(opacity=' + op * 100 + ')'
            });
            op += op * 0.1;
        }, 5);
    }

    render() {
        return (
            <div style={styles.mainDiv}>
                <button type="button"
                        onClick={() => this.iaButtonClicked()}
                        onMouseEnter={() => this.setState({iaButtonHovered: true})}
                        onMouseLeave={() => this.setState({iaButtonHovered: false})}
                        onFocus={() => this.setState({iaButtonFocused: true})}
                        onBlur={() => this.setState({iaButtonFocused: false})}
                        style={{...styles.iaBtn, ...{opacity: this.state.iaButtonHovered ? 0.9 : 1, outlineWidth: this.state.iaButtonFocused ? 0 : 'inherit', display: this.state.iaBtnHidden ? 'hidden' : 'block'}}}>
                    <img src={this.state.iaWhiteLogoSource} />
                </button>
                <iframe src={this.state.iaFrameSource} width="170" height="215" frameBorder="0" style={{...styles.iaFrame, ...{opacity: this.state.iaFrameOpacity, filter: this.state.iaFrameFilter, display: this.state.iaFrameDisplayed ? 'block' : 'none'}}}/>
            </div>
        )
    }
};

IAButton.propTypes = {
    iaServer: PropTypes.string,
    clientId: PropTypes.string.isRequired,
    clientSecret: PropTypes.string.isRequired,
    payment: PropTypes.string,
    notifyUser: PropTypes.bool,
    successIA: PropTypes.func.isRequired,
    deniedIA: PropTypes.func.isRequired,
    timeoutIA: PropTypes.func.isRequired
};

IAButton.defaultProps = {
    notifyUser: true,
    iaServer: '//dashboard.instantaccess.io'
};

const styles = {
    mainDiv: {
        display: 'inline-block',
        position: 'relative'
    },
    iaBtn: {
        backgroundColor: '#4A4A4A',
        borderWidth: 0,
        padding: 0,
        width: 56,
        height: 56,
        borderRadius:10,
        cursor: 'pointer'
    },
    iaFrame: {
        display:'none',
        borderRadius:10,
        zIndex: 1000,
        position: 'absolute',
        left: 0,
        bottom: 0
    }
};

> End of Step 1


> Step 2: Add IAButton component where you need

  <IAButton clientId="IA_PARTNER_CLIENT_ID_KEY"
            clientSecret="IA_PARTNER_CLIENT_SECRET_KEY"
            notifyUser={false}
            successIA={(usernameOrEmail) => IA_SUCCESS_HANDLER}
            deniedIA={(message) => OPTIONAL_IA_DENIED_HANDLER}
            timeoutIA={(message) => OPTIONAL_IA_TIMEOUT_HANDLER} />

> Note: In IA_SUCCESS_HANDLER you should handle user authentication and make actions, applicable to your website

> Note: If you don't want to handle success, denied, or timeout, replace *_HANDLER with {}

> End of Step 2
```


## For collecting payment from your clients.

<aside class="notice">
You must replace <code>YOUR_PAYMENT_AMOUNT_WITH_THE_REGISTERED_CURRENCY_FOR_YOU_ON_IA_PLATFORM</code> with your payment that you want to collect from your user.
</aside>

<aside class="notice">
The Payment currency is the currency that you requested when you registered with the IA Platform.
</aside>

> To integrate IA on your website collecting payment from your clients, add this code snippet to your website:

```javascript
  <!-- IA Authentication -->
  <script type="text/javascript">
    window.onload = function(e){
      var client_id = 'IA_PARTNER_CLIENT_ID_KEY';
      var client_secret = 'IA_PARTNER_CLIENT_SECRET_KEY';
      var payment = 'YOUR_PAYMENT_AMOUNT_WITH_THE_REGISTERED_CURRENCY_FOR_YOU_ON_IA_PLATFORM';

      function successIA(usernameOrEmail) {
        // Handle successLogin in IA
        // Example: window.location = 'IACompleteRegisteration.php?iaUser='+usernameOrEmail
      }

      function deniedIA(message) {
        alert(message);
      }

      function timeoutIA(message) {
        alert(message);
      }


      document.getElementById('ia-framework-login').src = 'https://dashboard.instantaccess.io/api/partner/authorize/view?client_id='+client_id+'&client_secret='+client_secret+'&payment='+payment;
      //listen to ia back
      window.addEventListener('message',function(event) {
        if(event.origin !== 'https://dashboard.instantaccess.io') return;
        if(event.data.action === 'alert')
          alert(event.data.message);
        else if(event.data.action === 'closeframe') {
          document.getElementById("ia-btn").style.display = "block";
          fade(document.getElementById("ia-framework-login"));
          //document.getElementById("ia-framework-login").style.display = "none";
        }
        else if(event.data.action === 'auth') {
          switch (event.data.status) {
            case 'success':
              successIA(event.data.usernameOrEmail);
              break;
            case 'denied':
              deniedIA(event.data.message);
              break;
            case 'timeout':
              timeoutIA(event.data.message);
              break;
          }
        }
      },false);
    }

    function iaButtonClicked() {
      unfade(document.getElementById("ia-framework-login"));
      //document.getElementById("ia-framework-login").style.display = "block";
      document.getElementById("ia-btn").style.display = "hidden";
    }

    function fade(element) {
      var op = 1;  // initial opacity
      var timer = setInterval(function () {
        if (op <= 0.1){
          clearInterval(timer);
          element.style.display = 'none';
        }
        element.style.opacity = op;
        element.style.filter = 'alpha(opacity=' + op * 100 + ")";
        op -= op * 0.1;
      }, 10);
    }

    function unfade(element) {
      var op = 0.1;  // initial opacity
      element.style.display = 'block';
      var timer = setInterval(function () {
        if (op >= 1){
          clearInterval(timer);
        }
        element.style.opacity = op;
        element.style.filter = 'alpha(opacity=' + op * 100 + ")";
        op += op * 0.1;
      }, 5);
    }
  </script>
  <div style="display: inline-block; position: relative;">
    <button type="button" onclick="iaButtonClicked()" id="ia-btn" style="background-color: #4A4A4A; border-width: 0; padding: 0; width: 56px; height: 56px; border-radius:10px;">
      <img src="https://dashboard.instantaccess.io/upload/image/whitelogo.svg"/>
    </button>
    <iframe id="ia-framework-login" src="" width="170" height="215" frameBorder="0" style="display:none; border-radius:10px; z-index: 1000; position: absolute; left: 0px; bottom: 0px;"></iframe>
  </div>
  <style>
    #ia-btn:focus {
      outline-width: 0;
    }
    #ia-btn:hover {
      opacity: 0.9;
      cursor: pointer;
    }
  </style>
  <!-- End of login with IA -->
```

```javascript--react
> Step 1: IAButton React component source code - save this file as IAButton.js in your project

import React from "react";
import PropTypes from 'prop-types';

export default class IAButton extends React.Component {

    constructor (props) {
        super(props);
        this.state = {
            iaWhiteLogoSource: props.iaServer+'/upload/image/whitelogo.svg',
            iaFrameSource: props.iaServer+'/api/partner/authorize/view?client_id='+props.clientId+'&client_secret='+props.clientSecret+(props.payment?('&payment='+props.payment):''),
            iaButtonHovered: false,
            iaButtonFocused: false,
            iaButtonHidden: false,
            iaFrameDisplayed: false,
            iaFrameOpacity: 0.1,
            iaFrameFilter: 'alpha(opacity=0)'
        }
    }

    componentDidMount () {
        window.addEventListener("message", this.handleIAMessage);
    }

    componentWillUnmount () {
        window.removeEventListener("message", this.handleIAMessage);
    }

    handleIAMessage = (event) => {
        var eventOrigin = event.origin;
        if(!this.props.iaServer.toLowerCase().startsWith("http")) {
            if(eventOrigin.toLowerCase().startsWith("http:")) {
                eventOrigin = eventOrigin.substr(5);
            } else if(eventOrigin.toLowerCase().startsWith("https:")) {
                eventOrigin = eventOrigin.substr(6);
            }
        }
        if(eventOrigin !== this.props.iaServer) return;
        if(event.data.action === 'alert') {
            alert(event.data.message);
        } else if(event.data.action === 'closeframe') {
            this.setState({
                iaBtnHidden: false
            });
            this.fadeIAFrame();
        }
        else if(event.data.action === 'auth') {
            switch (event.data.status) {
                case 'success':
                    this.successIA(event.data.usernameOrEmail);
                    break;
                case 'denied':
                    this.deniedIA(event.data.message);
                    break;
                case 'timeout':
                    this.timeoutIA(event.data.message);
                    break;
            }
        }
    }

    successIA (usernameOrEmail) {
        // Handle successLogin in IA
        this.props.successIA(usernameOrEmail);
    }

    deniedIA (message) {
        if(this.props.notifyUser) {
            alert(message);
        }
        this.props.deniedIA(message);
    }

    timeoutIA (message) {
        if(this.props.notifyUser) {
            alert(message);
        }
        this.props.timeoutIA(message);
    }

    iaButtonClicked () {
        this.unfadeIAFrame();
        this.setState({
            iaBtnHidden: true
        });
    }

    fadeIAFrame() {
        var op = 1;  // initial opacity
        var timer = setInterval(() => {
            if (op <= 0.1){
                clearInterval(timer);
                this.setState({iaFrameDisplayed: false});
            }
            this.setState({
                iaFrameOpacity: op,
                iaFrameFilter: 'alpha(opacity=' + op * 100 + ')'
            });
            op -= op * 0.1;
        }, 10);
    }

    unfadeIAFrame () {
        var op = 0.1;  // initial opacity
        this.setState({iaFrameDisplayed: true});
        var timer = setInterval(() => {
            if (op >= 1){
                clearInterval(timer);
            }
            this.setState({
                iaFrameOpacity: op,
                iaFrameFilter: 'alpha(opacity=' + op * 100 + ')'
            });
            op += op * 0.1;
        }, 5);
    }

    render() {
        return (
            <div style={styles.mainDiv}>
                <button type="button"
                        onClick={() => this.iaButtonClicked()}
                        onMouseEnter={() => this.setState({iaButtonHovered: true})}
                        onMouseLeave={() => this.setState({iaButtonHovered: false})}
                        onFocus={() => this.setState({iaButtonFocused: true})}
                        onBlur={() => this.setState({iaButtonFocused: false})}
                        style={{...styles.iaBtn, ...{opacity: this.state.iaButtonHovered ? 0.9 : 1, outlineWidth: this.state.iaButtonFocused ? 0 : 'inherit', display: this.state.iaBtnHidden ? 'hidden' : 'block'}}}>
                    <img src={this.state.iaWhiteLogoSource} />
                </button>
                <iframe src={this.state.iaFrameSource} width="170" height="215" frameBorder="0" style={{...styles.iaFrame, ...{opacity: this.state.iaFrameOpacity, filter: this.state.iaFrameFilter, display: this.state.iaFrameDisplayed ? 'block' : 'none'}}}/>
            </div>
        )
    }
};

IAButton.propTypes = {
    iaServer: PropTypes.string,
    clientId: PropTypes.string.isRequired,
    clientSecret: PropTypes.string.isRequired,
    payment: PropTypes.string,
    notifyUser: PropTypes.bool,
    successIA: PropTypes.func.isRequired,
    deniedIA: PropTypes.func.isRequired,
    timeoutIA: PropTypes.func.isRequired
};

IAButton.defaultProps = {
    notifyUser: true,
    iaServer: '//dashboard.instantaccess.io'
};

const styles = {
    mainDiv: {
        display: 'inline-block',
        position: 'relative'
    },
    iaBtn: {
        backgroundColor: '#4A4A4A',
        borderWidth: 0,
        padding: 0,
        width: 56,
        height: 56,
        borderRadius:10,
        cursor: 'pointer'
    },
    iaFrame: {
        display:'none',
        borderRadius:10,
        zIndex: 1000,
        position: 'absolute',
        left: 0,
        bottom: 0
    }
};

> End of Step 1


> Step 2: Add IAButton component where you need

  <IAButton clientId="IA_PARTNER_CLIENT_ID_KEY"
            clientSecret="IA_PARTNER_CLIENT_SECRET_KEY"
            payment="YOUR_PAYMENT_AMOUNT_WITH_THE_REGISTERED_CURRENCY_FOR_YOU_ON_IA_PLATFORM";
            notifyUser={false}
            successIA={(usernameOrEmail) => IA_SUCCESS_HANDLER}
            deniedIA={(message) => OPTIONAL_IA_DENIED_HANDLER}
            timeoutIA={(message) => OPTIONAL_IA_TIMEOUT_HANDLER} />

> Note: In IA_SUCCESS_HANDLER you should handle user authentication and make actions, applicable to your website

> Note: If you don't want to handle success, denied, or timeout, replace *_HANDLER with {}

> End of Step 2
```


# Handling IA Authentication

IA System will call one of this three methods when finish IA Authentication according to the status (success, denied or fail, timeout)

## Success

```javascript
    function successIA(usernameOrEmail) {
      // Handle successLogin in IA
      // Example: window.location = 'IACompleteRegisteration.php?iaUser='+usernameOrEmail
    }
```

```javascript--react
> See Step 2 code snippet of IA Partner Integration

    successIA={(usernameOrEmail) => IA_SUCCESS_HANDLER}

> replace IA_SUCCESS_HANDLER with your handler or use {}
```

This method will be called when user finished IA Authentication successfully and approved the request on their IA app.

Simply implement this method and continue your user flow, IA will provide all approved user info to your callback endpoint with the username and user email before calling this method. You can implement the callback to save these data for later use. Please refer to 'Callback Implementation Section'.

You can fetch user data using ajax or reditect to another page in this method and continue your user process. All authorized user info must be saved now on your system via your callback url.

### Query Parameters

Parameter | Description
--------- | -----------
usernameOrEmail | This is the username or email that the user used it to authenticate your service via IA System.

## Denied or Failed

```javascript
      function deniedIA(message) {
        alert(message);
      }
```

```javascript--react
> See Step 2 code snippet of IA Partner Integration

     deniedIA={(message) => OPTIONAL_IA_DENIED_HANDLER}

> replace OPTIONAL_IA_DENIED_HANDLER with your handler or use {}
```

This method will be called if your website user denied the IA Authentication request on the IA App or the request fail for any other reason.

By default, this method alerts the user with a failing message, however you can change this to suit your need.

### Query Parameters

Parameter | Description
--------- | -----------
message | It contains the reason of failure if the user denied the process, ...etc.

## Timeout

```javascript
      function timeoutIA(message) {
        alert(message);
      }
```

```javascript--react
> See Step 2 code snippet of IA Partner Integration

      timeoutIA={(message) => OPTIONAL_IA_TIMEOUT_HANDLER} />

> replace OPTIONAL_IA_TIMEOUT_HANDLER with your handler or use {}
```

This method will be called if user failed to respond to IA request over IA App.

By default, this method alerts the user with a timeout message.

### Query Parameters

Parameter | Description
--------- | -----------
message | It contains timeout message.


# Callback Implementation

The callback is requested from IA platform in two conditions:

1- When user approved the request over IA App, the Callback URL will be called with two parameters.
2- when the user update any information partner website has access to it. 

## Callback when user approve request over IA App

```php
$authorizationCode = $_GET['code'];
$state = $_GET['state'];
$stateArr = explode("@@", $state);
$userid = $stateArr[0];
$username = $stateArr[1];
$email = $stateArr[2];
```


### HTTP Request

`GET http://yourwebsite.com/yourCallBackurl?code={AUTHORIZATION_CODE}&state={userid}@@{username}@@{email}`

Parameter | Description
--------- | -----------
code | the Authorization Code that used to fetch the user’s token from IA
state | this string contains the userid, username and email concatenated with '@@', you can split them and use them.


## Callback when user update any information partner website has access to it 

```php
$userId = $_POST['user_id'];
```


### HTTP Request

`POST http://yourwebsite.com/yourCallBackurl`

Parameter | Description
--------- | -----------
user_id | the user id for the user who updated any associated info.

## fetching token for this user from IA System

```php
//setup the request, you can also use CURLOPT_URL
$curl = curl_init('https://dashboard.instantaccess.io/api/oauth/token');

// SET API request type as post
curl_setopt($curl, CURLOPT_POST, 1);

// Returns the data/output as a string instead of raw data
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);
curl_setopt($curl, CURLOPT_POSTFIELDS, http_build_query(array('client_id'=>'IA_PARTNER_CLIENT_ID_KEY',
                                                              'client_secret'=>'IA_PARTNER_CLIENT_SECRET_KEY',
                                                              'code'=>'your Authorization Code that provided in the previous step',
                                                              'redirect_uri' => 'your callback url',
                                                              'grant_type'=>'authorization_code')));

$result = curl_exec($curl);

curl_close($curl);

if($result !== FALSE) {
  $data = json_decode($result,true);
  $user_token = $data["access_token"];
}
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "message" : "User data items retrieved successfully",
  "data":[
      "phone": "+01111111111111",
      "Email": "test@test.com",
  ],
}
```

Please use the authorization code provided previously to fetch user token and save it in your users record. (database, files, ...etc)

<aside class="success">
This token is required everytime your system requests user info from IA. 
</aside>

### HTTP Request

`POST https://dashboard.instantaccess.io/api/oauth/token`

Parameter | Description
--------- | -----------
client_id | your IA_PARTNER_CLIENT_ID_KEY
client_secret | your IA_PARTNER_CLIENT_SECRET_KEY
code | your Authorization Code that provided in the previous step
redirect_uri | your callback url
grant_type | always with string value 'authorization_code'


## Fetch User Token

```php
if($result !== FALSE) {
  $data = json_decode($result,true);
  $user_token = $data["access_token"];
}

//setup the request, you can also use CURLOPT_URL
$curl = curl_init('https://dashboard.instantaccess.io/api/public/user');

// Returns the data/output as a string instead of raw data
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);
//Set your auth headers
curl_setopt($curl, CURLOPT_HTTPHEADER, array(
    'Content-Type: application/json',
    'Authorization: Bearer ' . $user_token
    ));

$result = curl_exec($curl);

curl_close($curl);

if($result !== FALSE) {

  $json = json_decode($result);
  $data = $json->data

  // Save it in your DB is the best choice :)
  file_put_contents('ANY FILE PATH', json_encode($data));
}
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "message" : "User data items retrieved successfully",
  "data":[
      "phone": "+01111111111111",
      "Email": "test@test.com",
  ],
}
```

Please use the user token to fetch user info from IA system. You can process these info as you usually do (e.g. save in database).

<aside class="success">
You can fetch user info with the user token via this api call at anytime.
</aside>

### HTTP Request

`GET https://dashboard.instantaccess.io/api/public/user`

NO Parameters here just set the Headers as following

Header | Description
--------- | -----------
Content-Type | 'application/json'
Authorization | 'Bearer {access_token that provided from last step}'

The data will retrieved in json string as following
