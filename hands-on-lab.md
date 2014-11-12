Module XX: *Authenticate to AD with iOS*
==========================

##Overview

The lab lets students configure and run an iOS App which allows the user to authenticate to an Azure ActiveDirectory account using ADALiOS library.

##Objectives

- Learn how to authenticate with Azure AD from iOS using the **Azure Active Directory AuthenticationLibrary (ADAL) for iOS**
- Learn how to get a token and store it for further requests

##Prerequisites

- Apple Macintosh environment
- XCode 6 (from the AppStore - compatible with iOS8 sdk)
- XCode developer tools (it will install git integration from XCode and the terminal)
- You must have a Windows Azure subscription to complete this lab.
- You must have completed Module 04 and linked your Azure subscription with your O365 tenant.
- You must have Cocoapods dependencies manager already installed on the Mac. (cocoapods.org)

##Exercises

The hands-on lab includes the following exercises:

- [Create a New Project and add ADAiOS library](#exercise1)
- [Add login and clear buttons to get a token from Azure AD](#exercise2)

<a name="exercise1"></a>
##Exercise 1: Create a New Project and add ADAiOS library
In this exercise you will create a New Project, download the ADALiOS library from the git repository and add it to the project.
Then check everything is ok, running the empty app.

###Task 1 - New Project
Download XCode 6 with iOS8 support, sdk and XCode developer tools.

01. Launch XCode and create a new **Sigle View Application**

    ![](img/fig.1.png)

02. Fill the inputs with the application info
     
    ![](img/fig.2.png)

03. Using Finder, on the project folder, create a new text file called **Podfile** (without extension), and add the lines

    ```
    pod 'ADALiOS', '~>1.0.0'
    ```  

04. Now open a terminal and navigate to the root folder of the project. Then execute the following command:

    ```
    pod install
    ```

    ```
    This will retrieve a copy of the ADALiOS library, store it into the local repository and
    import into our project.
    Always remember that this command will take effect if executed in the same folder as
    the .xcodeproj file.
    ```

05. Close XCode and re-open the project, this time using the **.xcworkspace** file. This is to have two projects: the target one and the pod libraries.

06. Under Project Settings go to **Build Phases**

    ![](img/fig.7.png)

07. Add an Item under **Link Binary with Libraries** and select the ADALiOS packed library (the file with .a extension)

    ![](img/fig.9.png)

08. Build the project from the menu **Project -> Build** and check if succeeded
    
    ![](img/fig.10.png)


###Task 3 - Import ADALiOS bundle

01. Importing ADALiOS bundle: the library comes with a bundle that is required to access the AzureAD login page

02. On XCode, create a new group to the project called **ADALiOSBundle**

    ![](img/fig.26.png)

02. On Finder, start from the root folder of the project, and look for **Pods/ADALiOS/ADALiOS** folder

03. Look for the storyboard files (one for iPad and one for iPhone) inside **ADALiOS** library folder

    ![](img/fig.27.png)

05. Drag the to files into XCode, under the newly created **ADALiOSBundle** folder

    ![](img/fig.11.png)

06. Build the project and check everything is ok.



<a name="exercise2"></a>
##Exercise 2: Create the application using storyboards and ADALiOS library
In this exercise you will add the action buttons to the application, connect them to the code and use the Login and Clear token features with ADALiOS

###Task 1 - Add buttons to the view and connect to the code

01. Using the storyboard feature, open the Main.storyboard file and add two buttons for Login and Clear actions

    ![](img/fig.13.png)

    ![](img/fig.14.png)
    
02. Change the storyboard size to have a real reference of the screen. Do this selecting the view controller and changing the size settings from the right panel
        
    ![](img/fig.15.png)

03. Open the **Assistant Editor** clicking the icon on the right corner of the XCode window. This will show the counterparts of a file

    ![](img/fig.25.png)

04. Select the **ViewController.h** file to connect the buttons

    ![](img/fig.16.png)

05. Right click on the buttons and drag the line to the header file of the controller.

    ![](img/fig.17.png)

    ![](img/fig.18.png)

06. Select the **ViewController.m** in the right editor to connect actions with this counterpart

    ![](img/fig.20.png)

07. Right click on the buttons and drag the line to the implementation file of the controller

    ![](img/fig.21.png)
    ![](img/fig.22.png)

08. Finally add an Image and a Label in the main screen

    ![](img/fig.24.png)


###Task 2 - Add behaviour to the buttons and use ADALiOS methods

01. On the **ViewController.m** file add the following imports before the implementation declaration

```
#import <ADALiOS/ADAuthenticationContext.h>
#import <ADALiOS/ADAuthenticationParameters.h>
#import <ADALiOS/ADAuthenticationSettings.h>
#import <ADALiOS/ADLogger.h>
#import <ADALiOS/ADInstanceDiscovery.h>
```

02. Inside the implementation declaration add the following variables

```
ADAuthenticationContext* authContext;
NSString* authority;
NSString* redirectUriString;
NSString* resourceId;
NSString* clientId;
NSString* token;
```

03. Set the variables values in the viewDidLoad method to initialize the info when the application starts

```
authority = [NSString alloc];
resourceId = [NSString alloc];
clientId = [NSString alloc];
redirectUriString = [NSString alloc];
token = [NSString alloc];

//Azure AD account info
authority = @"http://xxx.xxx";
resourceId = @"http://xxx.xxx";
clientId = @"xxxx-xxxx-xxxx-xxxx";
redirectUriString = @"http://xxx.xxx";
```

04. Call the login action with ADALiOS 

```
    - Add the following method to the implementation file

        -(void) getToken : (BOOL) clearCache completionHandler:(void (^) (NSString*))completionBlock;
        {
            ADAuthenticationError *error;
            authContext = [ADAuthenticationContext authenticationContextWithAuthority:authority error:&error];

            NSURL *redirectUri = [NSURL URLWithString:redirectUriString];

            if(clearCache){
                [authContext.tokenCacheStore removeAllWithError:nil];
            }

            [authContext acquireTokenWithResource:resourceId clientId:clientId redirectUri:redirectUri completionBlock:^(ADAuthenticationResult *result) {
                if (AD_SUCCEEDED != result.status){
                    // display error on the screen
                    [self showError:result.error.errorDetails];
                }
                else{
                    completionBlock(result.accessToken);
                }
            }];
        }
```
```
        -(void) showError:(NSString *)error{
            dispatch_async(dispatch_get_main_queue(), ^{
                NSString *errorMessage = [@"Login failed. Reason: " stringByAppendingString: error];
                UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Error" message:errorMessage delegate:self cancelButtonTitle:@"Retry" otherButtonTitles:@"Cancel", nil];
                [alert show];
            });
        }
```

05. Add behaviour to the login action button adding the following inside the action method

```
        - (IBAction)loginAction:(id)sender {
            [self getToken:FALSE completionHandler:^(NSString *token){
                dispatch_async(dispatch_get_main_queue(), ^{
                    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Success" message:token delegate:self cancelButtonTitle:@"Ok" otherButtonTitles:nil, nil];
                    [alert show];
                });
            }];
        }
```

06. Call the clear action with ADALiOS Adding the following to the clear action button behaviour method


```        
        - (IBAction)clearAction:(id)sender {
            ADAuthenticationError* error;
            id<ADTokenCacheStoring> cache = [ADAuthenticationSettings sharedInstance].defaultTokenCacheStore;
            NSArray* allItems = [cache allItemsWithError:&error];

            if (allItems.count > 0)
            {
                [cache removeAllWithError:&error];
            }

            if (error)
            {
                dispatch_async(dispatch_get_main_queue(), ^{
                    NSString *errorMessage = [@"Clear caché failed. Reason: " stringByAppendingString: error.errorDetails];
                    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Error" message:errorMessage delegate:self cancelButtonTitle:@"Ok" otherButtonTitles:nil, nil];
                    [alert show];
                });
                return;
            }

            NSHTTPCookieStorage* cookieStorage = [NSHTTPCookieStorage sharedHTTPCookieStorage];
            NSArray* cookies = cookieStorage.cookies;
            if (cookies.count)
            {
                for(NSHTTPCookie* cookie in cookies)
                {
                    [cookieStorage deleteCookie:cookie];
                }
            }

            dispatch_async(dispatch_get_main_queue(), ^{
                UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Success" message:@"Cookies Cleared" delegate:self cancelButtonTitle:@"Ok" otherButtonTitles:nil, nil];
                [alert show];
            });
        }
```    

07. Build and Run application with the **Play** button in the left corner of the screen

    ![](img/fig.23.png)


##Summary

By completing this hands-on lab you have learnt:

01. Some of the basics of iOS development.

02. How to add the ADALiOS library and bundles to the application 

03. How to call login and clear operations to handle the token from an AzureAD account for further requests

