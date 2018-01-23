---
title: Instant Access Partner - API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript: JavaScript
  - javascript--react: React
  - objective_c: Objective-C
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

* To integrate IA on your website for signin up or signing in, you will need to add the JavaScript or React code snippet to your website.

* To integrate IA in your iOS App for signin up or signing in, you will need to add the Objective-C units to your project.

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

```objective_c

* Step 1: add the following units to your project:

//
//  IALogin.h
//  https://WhatTuDu.com
//
//  Created by Christophe Delhaze - http://pikorua.it (chris@pikorua.it) on 7/12/17.
//  Copyright © 2017-2018 Christophe Delhaze. All rights reserved.
//

#import <Foundation/Foundation.h>

typedef void(^CompletionBlock)(BOOL success, NSError *error,NSDictionary *userData);

//Parameters required by IA
const static NSString *client_id = @"client_id"; // Insert your client_id here
const static NSString *client_secret = @"client_secret"; // Insert your client_secret here
const static NSString *partnerDomainURL = @"partnerDomainURL"; // Enter the domain you registered with IA
const static NSString *iaAuthorizeURL = @"https://dashboard.instantaccess.io/api/partner/authorize"; // To request authorization
const static NSString *iaAuthorizeStatusURL = @"https://dashboard.instantaccess.io/api/partner/status"; // To check the request status
const static NSString *payment = @""; // Not implemented in this version - Sign up / Login only.
const static int statusCallInterval = 3; // in seconds
const static int statusCheckingMaximumTime = 600; // in seconds

//Additional Parameters to fetch the user data from your server (IA can only do call back to your server so your app will need to fetch the data from there.)
//For this example we used php script on the server to fetch the data from a mySQL database. 
//You will need to secure your call to your server to prevent unauthorized access to your user data. 
//In this demo code, we will use basic security and do direct calls through https using a key/password conbination.
const static NSString *domainKey = @"your_server_secret_key";
const static NSString *domainPassword = @"your_server_secret_password";
const static NSString *userDataURLFormat = @"https://yourwebsite.com/get/userData/%@/%@/%@";
const static NSString *userIDDataURLFormat = @"https://yourwebsite.com/get/userIDData/%@/%@/%@";
const static NSString *userTokenDataURLFormat = @"https://yourwebsite.com/get/userTokenData/%@/%@/%@";

@interface IALogin : NSObject

@property (strong, nonatomic) NSString *userName; //Entered by the user on your app login screen
@property (assign, nonatomic) int statusCounter; 
@property (strong, nonatomic) NSTimer *connectionTimer; //Timer used to schedule a status check call then a user data call
@property (copy, nonatomic) CompletionBlock completionBlock; //Block called on success or failure of the IA login

+(instancetype)IALoginWithUsername:(NSString*) userName; //Create an instance of the IALogin Class

//Perform login. We pass the completion block, a caller (view) that will be displaying a HUD while the login is in process and will also allow to cancel the login process.
-(void)login:(CompletionBlock)completionBlock caller:(id)caller displayHUD:(SEL)displayHUD;

//Cancel login process
-(void)cancel;

@end


//
//  IALogin.m
//  https://WhatTuDu.com
//
//  Created by Christophe Delhaze - http://pikorua.it (chris@pikorua.it) on 7/12/17.
//  Copyright © 2017-2018 Christophe Delhaze. All rights reserved.
//

#import "IALogin.h"

#import <UserNotifications/UserNotifications.h>

@implementation IALogin

/* Create and instance of IALogin Class and set the userName */
+(instancetype)IALoginWithUsername:(NSString*) userName{
    IALogin *iaLogin = [IALogin new];
    iaLogin.userName = userName;
    return iaLogin;
}

/* Cancel the login process*/
-(void)cancel{
    [self.connectionTimer invalidate];
    self.connectionTimer = nil;
    if (self.completionBlock) {
        NSError *error = [NSError errorWithDomain:@"com.IA.error" code:2 userInfo:@{@"message":@"User Cancelled IA Login."}];
        self.completionBlock(NO, error, nil);
    }
}

/* Start the login process*/
/* In a perfect world, Your app would be able to open IA App once the login start and once the user is done with the login process, IA App would open back your app to complete the login */
/* At the present stage, IA App doesn't have an URL Scheme and doesn't allow to call your app URL Scheme. */
/* As a work around, IA server will send a push notification to proceed with the login. Your app will be using Local Notifications to bring back the user into your app once the login is complete*/
-(void)login:(CompletionBlock)completionBlock caller:(id)caller displayHUD:(SEL)displayHUD{
    UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
    [center getNotificationSettingsWithCompletionHandler:^(UNNotificationSettings * _Nonnull settings) {
        if (settings.authorizationStatus == UNAuthorizationStatusNotDetermined) {
            UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"IA Works better with Local Notifications" message:@"Please allow Local Notifications to Log In with IA." preferredStyle:UIAlertControllerStyleAlert];
            [alert addAction:[UIAlertAction actionWithTitle:@"Nope :(" style:UIAlertActionStyleDestructive handler:^(UIAlertAction * _Nonnull action) {
                [self proceedWithLogin:completionBlock];
            }]];
            [alert addAction:[UIAlertAction actionWithTitle:@"Sounds Great :D" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
                [center requestAuthorizationWithOptions:(UNAuthorizationOptionAlert + UNAuthorizationOptionSound + UNAuthorizationOptionBadge)
                                      completionHandler:^(BOOL granted, NSError * _Nullable error) {
                                          [caller performSelectorOnMainThread:displayHUD withObject:nil waitUntilDone:YES];
                                          [self proceedWithLogin:completionBlock];
                                      }];
            }]];
            [[[[UIApplication sharedApplication] keyWindow] rootViewController] presentViewController:alert animated:YES completion:^{
                
            }];
        } else {
            [caller performSelectorOnMainThread:displayHUD withObject:nil waitUntilDone:YES];
            [self proceedWithLogin:completionBlock];
        }
    }];
}

/* Proceed with the actual login. We will call IA server to request authorization*/
-(void)proceedWithLogin:(CompletionBlock)completionBlock{
    self.completionBlock = completionBlock;
    NSError *error;
    NSString *urlString = [NSString stringWithFormat:@"%@?client_id=%@&client_secret=%@&user_identifier=%@", iaAuthorizeURL, client_id, client_secret, self.userName];
    NSString *JSONString = [NSString stringWithContentsOfURL:[NSURL URLWithString:urlString] encoding:NSUTF8StringEncoding error:&error];
    if (error) {
        self.completionBlock(NO, error, nil);
        return;
    }
    NSDictionary *loginAuthenticationResponse;
    if (JSONString) {
        NSData* jsonEventsData = [JSONString dataUsingEncoding:NSUTF8StringEncoding];
        if (jsonEventsData) {
            error = nil;
            loginAuthenticationResponse = [NSJSONSerialization
                                           JSONObjectWithData:jsonEventsData
                                           options:0
                                           error:&error];
            if (error) {
              self.completionBlock(NO, error, nil);
              return;
            }
        }
    }
    if ([loginAuthenticationResponse[@"success"]boolValue]) {
        dispatch_async(dispatch_get_main_queue(), ^{
            self.connectionTimer = [NSTimer scheduledTimerWithTimeInterval:statusCallInterval repeats:NO block:^(NSTimer * _Nonnull timer) {
                [self checkStatus];
            }];
        });
    }
}

-(void)checkStatus {
    NSError *error;
    NSString *urlString = [NSString stringWithFormat:@"%@?client_id=%@&client_secret=%@&user_identifier=%@", iaAuthorizeStatusURL, client_id, client_secret, self.userName];
    NSString *JSONString = [NSString stringWithContentsOfURL:[NSURL URLWithString:urlString] encoding:NSUTF8StringEncoding error:&error];
    NSDictionary *loginAuthenticationResponse;
    if (JSONString) {
        NSData* jsonEventsData = [JSONString dataUsingEncoding:NSUTF8StringEncoding];
        if (jsonEventsData) {
            error = nil;
            loginAuthenticationResponse = [NSJSONSerialization
                                           JSONObjectWithData:jsonEventsData
                                           options:0
                                           error:&error];
        }
    }
    if( ([loginAuthenticationResponse[@"success"] boolValue] == false) && [loginAuthenticationResponse[@"message"] isEqualToString: @"User has previously authorized this partner" ]) {
        [self authSuccess];
    } else {
        [self updateStatus:loginAuthenticationResponse[@"data"]];
    }
}

-(void)updateStatus:(NSString*)current {
    NSDictionary *status = @{@"approve":@1,@"deny":@2};
    switch ([status[current] intValue]) {
        case 1:
             [self authSuccess];
            break;
        case 2:
            [self authDenied];
            break;
        default:
            if (self.statusCounter * statusCallInterval > statusCheckingMaximumTime) {
                [self authTimeout];
                break;
            }
            self.statusCounter += 1;
            dispatch_async(dispatch_get_main_queue(), ^{
                self.connectionTimer = [NSTimer scheduledTimerWithTimeInterval:statusCallInterval repeats:NO block:^(NSTimer * _Nonnull timer) {
                    [self checkStatus];
                }];
            });
            break;
    }
}

-(void)authSuccess{
    dispatch_async(dispatch_get_main_queue(), ^{
        self.connectionTimer = [NSTimer scheduledTimerWithTimeInterval:statusCallInterval repeats:NO block:^(NSTimer * _Nonnull timer) {
            [self getUserData];
        }];
    });
}

-(void)authDenied{
    NSError *error = [NSError errorWithDomain:@"com.IA.error" code:1 userInfo:@{@"message":@"Authentication Denied"}];
    self.completionBlock(NO, error, nil);
}

-(void)authTimeout{
    NSError *error = [NSError errorWithDomain:@"com.IA.error" code:0 userInfo:@{@"message":@"Authentication Timed Out"}];
    self.completionBlock(NO, error, nil);
}

-(void)getUserData {
    NSError *error;
    NSString *urlString = [NSString stringWithFormat:userDataURLFormat, self.userName, domainKey, domainPassword];
    NSString *JSONString = [NSString stringWithContentsOfURL:[NSURL URLWithString:urlString] encoding:NSUTF8StringEncoding error:&error];
    NSDictionary *userDataResponse;
    error = nil;
    if (JSONString) {
        NSData* jsonEventsData = [JSONString dataUsingEncoding:NSUTF8StringEncoding];
        if (jsonEventsData) {
            userDataResponse = [NSJSONSerialization
                                           JSONObjectWithData:jsonEventsData
                                           options:0
                                           error:&error];
        }
    }
    
    if ((error == nil)&&(JSONString)) {
        self.completionBlock(YES, error, userDataResponse);
    } else {
        NSError *error = [NSError errorWithDomain:@"com.IA.error" code:2 userInfo:@{@"message":@"Invalid User Data Returned"}];
        self.completionBlock(NO, error, nil);
    }
}

@end


* Step 2: Add The UI elements On you login page:

You will need 2 UIButton's and 1 UITextField.

1 Button (IALoginButton) will be used to make the other button and text field visible. 
By default that button will use the IA Logo. Alternatively as I did in WhatTuDu, you can create a custom button that integrate nicely in your UI.
See Resources on the left.

The text field (IALabel) is used by the user to input his user name and the 2nd button (IAButton) is used to proceed with the login.

```
<span style="padding: 10px; width: 100%; background: grey; display: block;margin-left: -28px;padding-left: 28px;padding-right: 28px;">
<strong>iOS Resources:</strong><br/><br/>
* IA Button:<br/><img style="padding: 10px;" src="/images/IAButton.png"/><br/>
* IA Logo for custom buttons:<br/><img style="padding: 10px;" src="/images/whitehand.png"/><br/>
* Custom Button Example:<br/><img style="padding: 10px;" width="350" src="/images/WhatTuDu.png"/><br/>
</span>

```objective_c

* Step 3: In the LoginViewController.h add / change the following

@class IALogin;

@interface LoginViewController : UIViewController <UITextFieldDelegate,UIViewControllerTransitioningDelegate>
- (IBAction)loginWithIA:(id)sender;
- (IBAction)displayIAForm:(id)sender;

@property (strong, nonatomic) IBOutlet UIButton *IALoginButton;
@property (strong, nonatomic) IBOutlet UIButton *IAButton;
@property (strong, nonatomic) IBOutlet UITextField *IALabel;
@property (strong, nonatomic) IALogin *iaLogin;


* Step 4: In LoginViewController.m add the following

#import "IALogin.h"
#import <UserNotifications/UserNotifications.h>

then in the @implementation part:

- (void)viewDidLoad {
    self.IALabel.delegate = self;
    //keep whatever other code you had in there...
}


//Implement the cancelButtonTapped to your need. 
//We hide the HUD and cancel the login then nil the iaLogin object.
- (void)cancelButtonTapped:(UIButton*)sender{
    [self.hudButton removeFromSuperview];
    self.hudButton = nil;
    [self.iaLogin cancel]; //Required
    self.iaLogin = nil; // Required
    [ProgressHUD dismiss];
    [self hideIAForm:nil]; // Required
}


//Implement the displayHUD to your need. 
//You have an example of implementation using a modified version of ProgressHUD here...
-(void)displayHUD{
    [ProgressHUD show:@"Please login using IA App or Tap here to cancel." Interaction:NO];
    [[NSRunLoop currentRunLoop] runUntilDate:[NSDate dateWithTimeIntervalSinceNow:0.1]];
    [[NSRunLoop mainRunLoop] runUntilDate:[NSDate dateWithTimeIntervalSinceNow:0.1]];
    ProgressHUD *hud = [ProgressHUD shared];
    self.hudButton = [UIButton buttonWithType:UIButtonTypeCustom];
    [self.hudButton addTarget:self action:@selector(cancelButtonTapped:) forControlEvents:UIControlEventTouchUpInside];
    self.hudButton.frame = hud.frame;
    self.hudButton.layer.zPosition = CGFLOAT_MAX;
    [hud.backgroundView addSubview:self.hudButton];
    hud.backgroundView.layer.zPosition = 0;
}

- (IBAction)loginWithIA:(id)sender {
    if (![self.IALabel.text isEqualToString:@""]) { //we need a userName
        [self.IALabel resignFirstResponder];
        
        self.iaLogin = [IALogin IALoginWithUsername:self.IALabel.text];
        [self.iaLogin login:^(BOOL success, NSError *error, NSDictionary *userData) {
            UNNotificationCategory* loginCategory = [UNNotificationCategory
                                                     categoryWithIdentifier:@"LOGIN"
                                                     actions:@[]
                                                     intentIdentifiers:@[]
                                                     options:UNNotificationCategoryOptionCustomDismissAction];
            
            // Register the notification categories.
            UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
            [center setNotificationCategories:[NSSet setWithObjects:loginCategory, nil]];
            
            UNMutableNotificationContent* content = [[UNMutableNotificationContent alloc] init];
            content.title = [NSString localizedUserNotificationStringForKey:@"IA Login!" arguments:nil];
            content.body = [NSString localizedUserNotificationStringForKey:@"IA Login Completed!\nPlease go back to WhatTuDu Now!"
                                                                 arguments:nil];
            content.categoryIdentifier = @"LOGIN";
            content.sound = [UNNotificationSound defaultSound];
            
            // Configure the trigger
            NSDate *now = [NSDate dateWithTimeIntervalSinceNow:1];
            NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier: NSCalendarIdentifierGregorian];
            NSDateComponents *date = [calendar components:NSCalendarUnitYear|NSCalendarUnitMonth|NSCalendarUnitDay|NSCalendarUnitHour|NSCalendarUnitMinute|NSCalendarUnitSecond fromDate:now];
            UNCalendarNotificationTrigger* trigger = [UNCalendarNotificationTrigger triggerWithDateMatchingComponents:date repeats:NO];
            
            // Create the request object.
            UNNotificationRequest* request = [UNNotificationRequest
                                              requestWithIdentifier:@"IALogin" content:content trigger:trigger];

            [center addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
                if (error != nil) {
                    NSLog(@"%@", error.localizedDescription);
                }
            }];
            
            if (!error) {
                //Everything went as planned and we have the userData back from our server.
                [self loginSucceeded:userData];
            } else {
                if (error.code == 256) {
                    [[[UIAlertView alloc] initWithTitle:@"Error!" message:@"Please enter a valid username and try again." delegate:nil cancelButtonTitle:@"OK" otherButtonTitles: nil] show];
                } else {
                    [[[UIAlertView alloc] initWithTitle:@"Error!" message:error.userInfo[@"message"] delegate:nil cancelButtonTitle:@"OK" otherButtonTitles: nil] show];
                }
            }
            [self.hudButton removeFromSuperview];
            self.hudButton = nil;
            [ProgressHUD dismiss];
            self.iaLogin = nil;
            [self hideIAForm:nil];
        } caller:self displayHUD:@selector(displayHUD)];
    } else {
        [[[UIAlertView alloc] initWithTitle:@"Warning..." message:@"Please enter a username and try again." delegate:nil cancelButtonTitle:@"OK" otherButtonTitles: nil] show];
    }
}


-(void)loginSucceeded:(NSDictionary*)userData{
    
    NSError *error = nil;
    NSString *urlString = [NSString stringWithFormat:userIDDataURLFormat, self.IALabel.text, domainKey, DomainPassword];
    NSURL *url = [NSURL URLWithString:urlString];
    NSString *ia_id = [NSString stringWithContentsOfURL:url encoding:NSUTF8StringEncoding error:&error]; //Retrieve the user ID

    urlString = [NSString stringWithFormat:userTokenDataURLFormat, self.IALabel.text, domainKey, DomainPassword];
    url = [NSURL URLWithString:urlString];
    error = nil;
    NSString *access_token = [NSString stringWithContentsOfURL:url encoding:NSUTF8StringEncoding error:&error]; //Retrieve the access_token

    NSDictionary *authData = @{
                               @"access_token"        :   access_token,
                               @"id"                  :   ia_id,
                               @"username"            :   self.IALabel.text
                               };
    
    //Create or login to your app server 
    //You will need to implement this based on your server architecture and replace YourServerUserClass accordingly.
    [YourServerUserClass logInInBackgroundWithAuthData:authData completionBlock:^id _Nullable(YourUserClass * _Nonnull loggedInUser) {
        //we can now update the user with the user data...
        NSString *fn, *ln, *g, *pp, *dob, *em;
        for (NSDictionary *data in userData) {
            if ([data[@"attribute"] isEqualToString:@"First Name"]) {
                fn = data[@"value"];
            } else if ([data[@"attribute"] isEqualToString:@"Last Name"]) {
                ln = data[@"value"];
            } else if ([data[@"attribute"] isEqualToString:@"Emails"]) {
                if([data[@"value"] isKindOfClass:[NSArray class]]) {
                    em = data[@"value"][0]; //We pick the first email from the list since our system doesn't support multiple emails.
                } else {
                    em = data[@"value"];
                }
            } else if ([data[@"attribute"] isEqualToString:@"Gender"]) {
                g = data[@"value"];
            } else if ([data[@"attribute"] isEqualToString:@"Profile Picture"]) {
                pp = data[@"value"];
            } else if ([data[@"attribute"] isEqualToString:@"Date of Birth"]) {
                dob = data[@"value"];
            }
        }
        
        NSString *fullname = [[[@"" stringByAppendingString:((fn)?fn:@"")] stringByAppendingString:((((fn)?fn:@"") && ln)?@" ":@"")] stringByAppendingString:((ln)?ln:@"")];
        loggedInUser[@"first_name"] = fn;
        loggedInUser[@"last_name"] = ln;
        loggedInUser[@"fullname"] = fullname;
        loggedInUser[@"email"] = em;
        loggedInUser[@"gender"] = [g lowercaseString];
        loggedInUser[@"picture_url"] = pp;
        loggedInUser[@"dob"] = dob;
        loggedInUser[@"ia_id"] = ia_id;
        loggedInUser[@"ia_username"] = self.IALabel.text;
        [newUser saveInBackgroundWithBlock:^(BOOL succeeded, NSError * _Nullable error) {
            [NotificationCenter post:NOTIFICATION_USER_LOGGED_IN];
            [self gotToMainView];
        }];
        return  nil;
    }];
    
}

- (IBAction)displayIAForm:(id)sender {
    [UIView animateWithDuration:0.4 animations:^{
        self.IALabel.hidden = NO;
        self.IAButton.hidden = NO;
    } completion:^(BOOL finished) {
        [self.IALabel becomeFirstResponder];
    }];
}

- (IBAction)hideIAForm:(id)sender {
    if ([self.IALabel isFirstResponder]) {
        [self.IALabel resignFirstResponder];
    }
    [UIView animateWithDuration:0.4 animations:^{
        self.IALabel.hidden = YES;
        self.IAButton.hidden = YES;
    } completion:^(BOOL finished) {
        //
    }];
}

-(BOOL)textFieldShouldReturn:(UITextField *)textField{
    [self loginWithIA:self.IAButton];
    return NO;
}

You will probably need to implement 

-(void)onKeyboardFrameChange:(NSNotification *)notification
-(void)onKeyboardShow:(NSNotification *)notification
-(void)onKeyboardHide:(NSNotification *)notification

To make sure that the IALabel is visible when the keyboard is displayed...

Any questions, direct message @Chris on slack https://ia-app.slack.com/messages

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
  $user_access_token = $data["access_token"];
  $user__refresh_token = $data["refresh_token"];
  $user_access_token_expiry = $data["expires_in"];
}
```

> The above command returns JSON structured like this:

```json
{
  "access_token": "THE_USER_ACCESS_TOKEN",
  "refresh_token": "THE_USER_REFRESH_TOKEN",
  "expires_in": "THE_NUMBER_SECONDS_OF_UNTIL_THE_ACCESS_TOKEN_EXPIRES"
}
```

Please use the authorization code provided previously to fetch user access token and refresh token then save it in your users record. (database, files, ...etc)

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

## Refreshing Access Token for this user from IA System

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
                                                              'refresh_token'=>'USER_REFRESH_TOKEN',
                                                              'grant_type'=>'refresh_token',
                                                              'scope' => '')));

$result = curl_exec($curl);

curl_close($curl);

if($result !== FALSE) {
  $data = json_decode($result,true);
  $user_access_token = $data["access_token"];
  $user__refresh_token = $data["refresh_token"];
  $user_access_token_expiry = $data["expires_in"];
}
```

> The above command returns JSON structured like this:

```json
{
  "access_token": "THE_USER_ACCESS_TOKEN",
  "refresh_token": "THE_USER_REFRESH_TOKEN",
  "expires_in": "THE_NUMBER_OF_SECONDS_UNTIL_THE_ACCESS_TOKEN_EXPIRES"
}
```

Please use the refresh_token provided previously to fetch new user access token and refresh token then save it in your users record. (database, files, ...etc)

<aside class="success">
This token is required everytime your system requests user info from IA. 
</aside>

### HTTP Request

`POST https://dashboard.instantaccess.io/api/oauth/token`

Parameter | Description
--------- | -----------
client_id | your IA_PARTNER_CLIENT_ID_KEY
client_secret | your IA_PARTNER_CLIENT_SECRET_KEY
refresh_token | your Authorization Code that provided in the previous step
grant_type | always with string value 'refresh_token'
scope | send it with empty string => ''


## Fetch User Information

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
