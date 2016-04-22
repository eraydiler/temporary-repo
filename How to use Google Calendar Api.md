# How to use Google Calendar Api

## Abstract
Hello everyone, as you all know couple of weeks ago apple published XCode7.3/iOS9.3. To be up to date, i was required to investigate some existing apps for possible bugs or upgrades with these brand new update. I went through some troubles and found some solutions during this updating process, so i will try to make things clearer for those who possibly will need some help about these issues.


## Libraries
When i tried to build one of the projects with XCode7.3 it failed to build pointing a library. Firstly i thought that it has a newer version than what had been being used in our project, and after a simple update everything would be ok. However, after some googling i realized that the library is not supported anymore and there is another one which is newer and supported officially by the google. 

These two are the libraries i am talking about;  
`gdata-objectivec-client:` <https://github.com/google/gdata-objectivec-client>. (older)  
`google-api-objectivec-client` <https://github.com/google/google-api-objectivec-client>. (newer)

Although this could make things a little harder it was still clear.
All i needed to do was to change deprecated library with the new one. However, it wasn't what i expected at all.  

### How to add GoogleAPIClient as pod?
I simply removed old library from project by moving it to trash with a right click in the project navigator of XCode.
And after removing the old one i tried to add new library as a pod by using cocoapods. 

#### Important Note: Make sure to add right pod name into the Podfile
If you search pod name [like this](https://cocoapods.org/?q=google%20api%20client) in search bar of cocoapods website, you will see two results as `Google-API-Client` and `GoogleAPIClient`. Be careful, right one is `GoogleAPIClient`.  

![Right pod name](/Users/Eray/Desktop/Blog post/Screen Shot 2016-04-13 at 00.13.30.png)

So the right line for your podfile should be;  
**`pod 'GoogleAPIClient'`**

#### Important Note: Indeed no need to lookup cocoapods's website
Some time after i figured it out that if i want to find the correct pod name, the right way to do is to look into `.podspec` file for any pod library. In this case, [GoogleAPIClient.podspec](https://github.com/google/google-api-objectivec-client/blob/master/GoogleAPIClient.podspec) tells us right pod name with line ` s.name         = 'GoogleAPIClient'`

GoogleAPIClient library includes number of different apis inside it, and it keeps them as subspecs that you can take a look at in [GoogleApiClient.podspec.](https://github.com/google/google-api-objectivec-client/blob/master/GoogleAPIClient.podspec)
In our project just Google Calendar Api is being used. Therefore, my podfile just includes;
**`pod 'GoogleAPIClient/Calendar'`**  
This way, only the files required by Calendar Api will be added into the project after making pod install. 

Another important information you will probably need when you want to use calendar api is that you will need `GTM OAuth 2` library for authenticating user into his google account. This library is not coming with pod as dependency. So i also needed to add it into my Podfile as seperate pod;  
**`pod 'GTMOAuth2'`**  

So, my final podfile looks like;  
**`pod 'GoogleAPIClient'`**  
**`pod 'GTMOAuth2'`**  

## Error i got due to flags remained from GData library 
After installing pod libraries in my podfile shown above, i encountered a weird problem. I imported **GTMOAuth2ViewControllerTouch.h** which is included in the **GTMOAuth2**.  

![Import statement](/Users/Eray/Desktop/Blog post/Screen Shot 2016-04-14 at 17.09.58.png)  

Although XCode does not complain for this import, whenever i tried to use any method of GTMOAuth2ViewControllerTouch class i got `use of undeclared identifier` error like below.

![Use of undeclared identifier](/Users/Eray/Desktop/Blog post/Screen Shot 2016-04-14 at 17.03.42.png)

The cause of this was these two flags, remained from old GData library which i removed before, in project settings.

`"-DGDATA_REQUIRE_SERVICE_INCLUDES=1"`  
`"-DGDATA_INCLUDE_CALENDAR_SERVICE=1"`

They were under `OTHER_CFLAGS` in `.../Project-Name.xcodeproj/project.pbxproj`.

After solving the issues mentioned above i was finally able to focus on my main task, implementing adding events to google calendar for users who has a google account.

In order to explain adding events into google calendar using google's api i will use a [sample application](https://github.com/eraydiler/GoogleCalendarApi) that i wrote for this post.

## Configuration in developers/google.com
Before using google calendar api, we need to make some configuration from google's developer console. Applications that will use google calendar api need a client ID.  

### Creating a new project
After opening [developer console](https://console.developers.google.com) if you don't already have one, you will need to create a new project using the panel at the right top of page.

**Create or select a project**  
![Google Right top panel](/Users/Eray/Desktop/Blog post/Screen Shot 2016-04-17 at 01.05.31.png)  

If you selected create a new project, you will required to select a name for the project, i chose `Google Calendar Sample App`.  

**Select a name**  
![Google Select a name for the project](/Users/Eray/Desktop/Blog post/Screen Shot 2016-04-17 at 01.19.38.png)  

**Enter Product name**
![Google product name](/Users/Eray/Desktop/Blog post/Screen Shot 2016-04-17 at 01.34.06.png)  

**Select Client ID**
![Google credential selection](/Users/Eray/Desktop/Blog post/Screen Shot 2016-04-17 at 01.44.27.png)  

**Enter a bundle ID**  
![Google bundle id](/Users/Eray/Desktop/Blog post/Screen Shot 2016-04-17 at 01.46.57.png)  

Finally you will be promted with the client ID.  
**Get Client ID**  
![Google client id](/Users/Eray/Desktop/Blog post/Screen Shot 2016-04-17 at 01.47.14.png)

#### Important Note: Make sure to enable Google Calendar API from the console.

![Google enable Calendar API 1](/Users/Eray/Desktop/Blog post/Screen Shot 2016-04-17 at 17.15.56.png)

![Google enable Calendar API 2](/Users/Eray/Desktop/Blog post/Screen Shot 2016-04-17 at 17.16.01.png)

## Eventbrite configuration
I used eventbrite's api to show some random events in sample application, to use its api you will need a OAuth token which is sent within requests to api as a parameter for authentication.

[Here](http://www.eventbrite.com/myaccount/apps/) you can access the app management page in developer portal of eventbrite. **Your personal OAuth token** is what you need to make sample app work.

![Eventbrite OAuth token](/Users/Eray/Desktop/Blog post/Screen Shot 2016-04-17 at 15.00.14.png)

## Sample App
After you get your `google api client id` and `eventbrite oauth token` then just assign them to `kGoogleAPIClientID` and `kEventbriteAuthToken` in the code.

Two class i used from the api are `GTLServiceCalendar` and `GTLCalendarEvent`, i needed to define two property for authentication and saving events.
 
~~~objc
@property (nonatomic, strong) GTLServiceCalendar *calendarService;
@property (nonatomic, strong) GTLCalendarEvent *calendarEvent;
~~~

We can check if a user is authenticated to his google account just checking the bool value `_calendarService.authorizer.canAuthorize`. 

~~~objc
_calendarService.authorizer = [GTMOAuth2ViewControllerTouch  		
							  authForGoogleFromKeychainForName:kGoogleAPIKeychainItemName
                    	      clientID:kGoogleAPIClientID
                    		  clientSecret:nil];
~~~

If user is not authenticated yet, google's authentication view will be presented. For this we need to use `GTMOAuth2ViewControllerTouch` class, which is implemented with a method shown below.

~~~objc
- (void)launchGoogleAuthenticationView {
    _didCancelGoogleAuthentication = NO;

    GTMOAuth2ViewControllerTouch *authController;

    // If modifying these scopes, delete your previously saved credentials by
    // resetting the iOS simulator or uninstall the app.
    NSArray *scopes = [NSArray arrayWithObjects:kGTLAuthScopeCalendar, nil];

    authController = [[GTMOAuth2ViewControllerTouch alloc]
                      initWithScope:[scopes componentsJoinedByString:@" "]
                      clientID:kGoogleAPIClientID
                      clientSecret:nil
                      keychainItemName:kGoogleAPIKeychainItemName
                      delegate:self
                      finishedSelector:@selector(googleAuthenticationViewController:finishedWithAuth:error:)];

    UIButton *closeButton = [UIButton buttonWithType:UIButtonTypeCustom];

    [closeButton setTitle:@"Cancel" forState:UIControlStateNormal];

    [closeButton addTarget:self
                    action:@selector(didTapCloseButton:)
          forControlEvents:UIControlEventTouchUpInside];

    UIBarButtonItem *closeButtonItem = [[UIBarButtonItem alloc]
                                         initWithCustomView:closeButton];

    [authController.navigationItem setLeftBarButtonItem:closeButtonItem];

    UINavigationController *navController = [[UINavigationController alloc]
                                             initWithRootViewController:authController];

    [self presentViewController:navController
                       animated:YES
                     completion:nil];
}
~~~

The method below is used for adding an event to google calendar.

~~~objc
- (void)addEventToGoogleCalendar {
    GCAEvent *selectedEvent = _events[_currentSelectedIndexPath.row];

    if (selectedEvent.isAdded) {
        return;
    }

    _calendarEvent = [[GTLCalendarEvent alloc] init];

    [_calendarEvent setSummary:selectedEvent.name];
    [_calendarEvent setDescriptionProperty:selectedEvent.content];

    NSDate *startDate = selectedEvent.startDate;
    NSDate *endDate = selectedEvent.endDate;

    if (endDate == nil) {
        endDate = [startDate dateByAddingTimeInterval:(60 * 60)];
    }

    GTLDateTime *startTime = [GTLDateTime dateTimeWithDate:startDate
                                                  timeZone:[NSTimeZone systemTimeZone]];

    [_calendarEvent setStart:[GTLCalendarEventDateTime object]];
    [_calendarEvent.start setDateTime:startTime];

    GTLDateTime *endTime = [GTLDateTime dateTimeWithDate:endDate
                                                timeZone:[NSTimeZone systemTimeZone]];

    [_calendarEvent setEnd:[GTLCalendarEventDateTime object]];
    [_calendarEvent.end setDateTime:endTime];


    GTLQueryCalendar *insertQuery = [GTLQueryCalendar queryForEventsInsertWithObject:_calendarEvent
                                                                          calendarId:kGoogleAPICalendarID];
    [self showAlertWithTitle:nil
                  andMessage:NSLocalizedString(@"Adding Event…", nil)];

    [_calendarService executeQuery:insertQuery
                 completionHandler:^(GTLServiceTicket *ticket, id object, NSError *error) {
                     if (error == nil) {
                         [self showAlertWithTitle:nil
                                       andMessage:NSLocalizedString(@"Event Added!", nil)];

                         selectedEvent.added = YES;
                         [self.tableView reloadRowsAtIndexPaths:@[_currentSelectedIndexPath]
                                               withRowAnimation:YES];
                     } else {
                         [self showAlertWithTitle:NSLocalizedString(@"Event Entry Failed", nil)
                                       andMessage:NSLocalizedString(@"Could not add event, please try again.", nil)];
                     }
                 }];
}
~~~  

## Conclusion

In this post, i tried to explain how i used google calendar api as simply as possible that i can make. Also mentioned some challenges i had faced with while trying to use the api. I tried to explain authenticating, and adding event into a google account on a sample application. Used eventbrite as data source for sample events to add the google calendar. I hope this will help someone in some way, for possible mistakes just let me know, take care. 

Sample Application on Github: <https://github.com/eraydiler/GoogleCalendarApi>
