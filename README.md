# _.Net-MAUI-Blazor-Notlar_

## ~~~
### _Apple Uygulama Yayınlama Aşamaları_

##### _Geliştirilen Uygulamanın Apple Test Flight Ortamına Yüklenmesi_

## ~~~
### _Initializing Verilerinin Tasarım Yüklendikten Sonra Çekilmesi_

##### _Main Threadin UI Threadi Beklemeden OnInitializedAsync() Methodunda Yapılan İşlemleri Gerçekleştirmesi_

```ruby
protected override async Task OnInitializedAsync()
    {
        await Task.Run(OnInitializing);
    }

    private async void OnInitializing()
    {
        try
        {
            _isLoading = true;
            await InvokeAsync(StateHasChanged);

            ...
        }
        catch
        {
            ...
        }
        finally
        {
            _isLoading = false;
            await InvokeAsync(StateHasChanged);
        }
    }
```


## ~~~
### _iOS Cihazlarda Firebase Cloud Messaging(FCM)_

##### _Kullanılan Paketler_
-
-

##### _GoogleService-Info.plist Dosyasının Projeye Eklenmesi_


##### _Info.plist Dosyasındaki Düzenlemeler_
```ruby
<key>UIBackgroundModes</key>
	<array>
		<string>location</string>
		<string>fetch</string>
		<string>remote-notification</string>
	</array>
<key>FirebaseAppDelegateProxyEnabled</key>
	<false/>
```

##### _entitlements.plist Dosyasındaki Düzenlemeler_
```ruby
<key>aps-environment</key>
<string>production</string>
```

##### _AppDelegate.cs Dosyasındaki Düzenlemeler_
```ruby
public class AppDelegate : MauiUIApplicationDelegate, IUNUserNotificationCenterDelegate, IMessagingDelegate
{
    protected override MauiApp CreateMauiApp() => MauiProgram.CreateMauiApp();
    public override bool FinishedLaunching(UIApplication app, NSDictionary options)
    {
        try
        {
            //   ~~~
            Firebase.Core.App.Configure();

            UNUserNotificationCenter.Current.Delegate = this;

            Messaging.SharedInstance.AutoInitEnabled = true;
            Messaging.SharedInstance.Delegate = this;

            if (UIDevice.CurrentDevice.CheckSystemVersion(10, 0))
            {
                // iOS 10 or later
                var authOptions = UNAuthorizationOptions.Alert | UNAuthorizationOptions.Badge | UNAuthorizationOptions.Sound;
                UNUserNotificationCenter.Current.RequestAuthorization(authOptions, (granted, error) =>
                {
                    //Console.WriteLine(granted);
                    Preferences.Set("Granted", granted);
                });

                // This Delete I used To Display Always Notification Even App Is Open
                //UNUserNotificationCenter.Current.Delegate = new UserNotificationCenterDelegate();
            }
            else
            {
                Preferences.Set("Granted", false);
            }

            UIApplication.SharedApplication.RegisterForRemoteNotifications();
        }
        catch (Exception ex)
        {

        }

        return base.FinishedLaunching(app, options);
    }

    //   ~~~
    [Export("messaging:didReceiveRegistrationToken:")]
    public void DidReceiveRegistrationToken(Messaging message, string fcmToken)
    {
        if (Preferences.ContainsKey("DeviceToken"))
        {
            Preferences.Remove("DeviceToken");
        }
        Preferences.Set("DeviceToken", fcmToken);

        //Messaging.SharedInstance.FcmToken
    }

    //   ~~~
    [Export("application:didRegisterForRemoteNotificationsWithDeviceToken:")]
    public void RegisteredForRemoteNotifications(UIApplication application, NSData deviceToken)
    {
        //var apnsToken = deviceToken.GetBase64EncodedString(NSDataBase64EncodingOptions.None);
        Messaging.SharedInstance.ApnsToken = deviceToken;
    }

    [Export("application:didReceiveRemoteNotification:fetchCompletionHandler:")]
    public void DidReceiveRemoteNotification(UIApplication application, NSDictionary userInfo, Action<UIBackgroundFetchResult> completionHandler)
    {
        Messaging.SharedInstance.AppDidReceiveMessage(userInfo);

        // Handle receiving remote notification
        completionHandler(UIBackgroundFetchResult.NewData);
    }

    [Export("userNotificationCenter:willPresentNotification:withCompletionHandler:")]
    public void WillPresentNotification(UNUserNotificationCenter center, UNNotification notification, Action<UNNotificationPresentationOptions> completionHandler)
    {
        NSDictionary userInfo = notification.Request.Content.UserInfo;
        //Messaging.SharedInstance.AppDidReceiveMessage(userInfo);

        // Show the notification as an alert even when the app is in the foreground
        completionHandler(UNNotificationPresentationOptions.Sound);
    }

    [Export("userNotificationCenter:didReceiveNotificationResponse:withCompletionHandler:")]
    public void DidReceiveNotificationResponse(UNUserNotificationCenter center, UNNotificationResponse response, Action completionHandler)
    {
        NSDictionary userInfo = response.Notification.Request.Content.UserInfo;
        //Messaging.SharedInstance.AppDidReceiveMessage(userInfo);

        // Handle the notification response (e.g., when user taps the notification)
        completionHandler();
    }



    //    ~~~
    //public class UserNotificationCenterDelegate : UNUserNotificationCenterDelegate
    //{
    //    public UserNotificationCenterDelegate() { }

    //    public override void DidReceiveNotificationResponse(UNUserNotificationCenter center, UNNotificationResponse response, Action completionHandler)
    //    {
    //        // handle your notification here.
    //    }

    //    public override void WillPresentNotification(UNUserNotificationCenter center, UNNotification notification, Action<UNNotificationPresentationOptions> completionHandler)
    //    {
    //        //    SystemSound.Vibrate.PlayAlertSound();
    //        //    SystemSound.Vibrate.PlaySystemSound();

    //        //    var userInfo = notification.Request.Content.UserInfo;

    //        //    Messaging.SharedInstance.AppDidReceiveMessage(userInfo);

    //        //    completionHandler(UNNotificationPresentationOptions.Alert | UNNotificationPresentationOptions.Badge | UNNotificationPresentationOptions.Sound);
    //        completionHandler(UNNotificationPresentationOptions.Alert);
    //    }
    //}

    //   ~~~
    //[Export("userNotificationCenter:didFailToRegisterForRemoteNotificationsWithError:")]
    //public async void FailedToRegisterForRemoteNotifications(UIApplication application, NSError error)
    //{
    //    // Handle the failure to register for remote notifications
    //}
}
```

##### _Apple Developer - APNs Key Üretimi ve Profil Düzenlemeleri_


##### _Yararlı Linkler_
- https://firebase.google.com/docs/cloud-messaging/ios/client?hl=en
- https://github.com/michalpr/Docs/blob/main/Firebase_MAUI_iOS_push_notification.md
- https://github.com/xamarin/GoogleApisForiOSComponents/blob/main/docs/Firebase/CloudMessaging/GettingStarted.md


## ~~~
### _Splash Ekranının Ardından Gelen Beyaz Ekran_

##### _wwwroot İçerisinde index.html Dosyasında Düzenlemeler_

```ruby
<body>
    <div class="status-bar-safe-area"></div>
    <div id="app">Loading...</div>

    <div id="blazor-error-ui">
        An unhandled error has occurred.
        <a href="" class="reload">Reload</a>
        <a class="dismiss">🗙</a>
    </div>
    <script src="_framework/blazor.webview.js" autostart="false"></script>
</body>
```
```ruby
<div id="app" style="background-color: black;">Loading...</div>
```

##### _App.xaml Dosyasında Düzenlemeler_

```ruby
<Style TargetType="BlazorWebView">
    <Setter Property="BackgroundColor" Value="Black" />
</Style>
<Style TargetType="NavigationPage" ApplyToDerivedTypes="True">
    <Setter Property="BackgroundColor" Value="Black" />
</Style>
```


## ~~~
### _..._

##### _..._
