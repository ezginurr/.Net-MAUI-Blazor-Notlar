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
- Xamarin.Firebase.iOS.CloudMessaging
-

##### _GoogleService-Info.plist Dosyasının Projeye Eklenmesi - .csprog Dosyası_
```ruby
	<ItemGroup>
		<BundleResource Include="Platforms\iOS\GoogleService-Info.plist" Link="GoogleService-Info.plist"/>
	</ItemGroup>
```

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
### _Android Cihazlarda Firebase Cloud Messaging(FCM)_

##### _Kullanılan Paketler_
- Xamarin.AndroidX.Collection
- Xamarin.AndroidX.Collection.Ktx
- Xamarin.AndroidX.Preference
- Xamarin.Google.Dagger
- Xamarin.Firebase.Messaging
- Xamarin.GooglePlayServices.Base

##### _google-services.json Dosyasının Projeye Eklenmesi_
```ruby
	<ItemGroup>
		<GoogleServicesJson Include="Platforms\Android\google-services.json" />
	</ItemGroup>
```

##### _AndroidManifest.xml Dosyasındaki Düzenlemeler_
```ruby
	<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
```

##### _MainActivity.cs Dosyasındaki Düzenlemeler_
```ruby
    internal static readonly string Channel_ID = "FCMChannel";
    public MainActivity() { }

    protected override void OnCreate(Bundle savedInstanceState)
    {
        base.OnCreate(savedInstanceState);

        if (ContextCompat.CheckSelfPermission(this, Android.Manifest.Permission.PostNotifications) == Permission.Denied)
        {
            ActivityCompat.RequestPermissions(this, new String[] { Android.Manifest.Permission.PostNotifications }, 1);
        }

        CreateNotificationChannel();
    }

    private void CreateNotificationChannel()
    {
        if (OperatingSystem.IsOSPlatformVersionAtLeast("android", 26))
        {
            var channel = new NotificationChannel(Channel_ID, "FCM Notification Channel", NotificationImportance.Default);

            NotificationManager notificationManager = (NotificationManager)GetSystemService(Android.Content.Context.NotificationService);
            notificationManager.CreateNotificationChannel(channel);
        }
    }
```

##### _FCMService.cs Dosyasındaki Düzenlemeler_
Firebase methodlarını override edebilmek için eklediğimiz bir servis

```ruby
    [Service(Exported = true)]
    [IntentFilter(new[] { "com.google.firebase.MESSAGING_EVENT" })]
    public class FCMService : FirebaseMessagingService
    {
        public FCMService() { }

        public override void OnNewToken(string fcmToken)
        {
            base.OnNewToken(fcmToken);

            if (Preferences.ContainsKey("DeviceToken"))
            {
                Preferences.Remove("DeviceToken");
            }
            Preferences.Set("DeviceToken", fcmToken);
        }

        public override void OnMessageReceived(RemoteMessage message)
        {
            base.OnMessageReceived(message);

            var notification = message.GetNotification();

            if (message.Data.ContainsKey("AnnouncementId"))
            {
                var annId = message.Data["AnnouncementId"].ToString();
                Preferences.Default.Set("AnnouncementId", annId);
            }

            //SendNotification(notification.Body, notification.Title, message.Data);
            int notificationId = new Random().Next();

            //if (OperatingSystem.IsOSPlatformVersionAtLeast("android", 26)) {
            //    var channel = new NotificationChannel(channelId, "FCM Notification Channel", NotificationImportance.Default);
            //    notificationManager.CreateNotificationChannel(channel);
            //}

            NotificationCompat.Builder builder = new NotificationCompat.Builder(this, MainActivity.Channel_ID)
                .SetContentTitle(message.GetNotification().Title) 
                .SetContentText(message.GetNotification().Body)
                .SetSmallIcon(Resource.Drawable.notificon)
                .SetAutoCancel(true);

            NotificationManager notificationManager = (NotificationManager)GetSystemService(NotificationService);
            notificationManager.Notify(notificationId, builder.Build());
        }
    }
```

##### _Yararlı Linkler_
- https://www.youtube.com/watch?v=gBbbctEvbOk
- https://github.com/mistrypragnesh40/PushNotificationDemoMAUI/tree/master/Platforms/Android


## ~~~
### _Firebase - .Net Core Bildirim Gönderme Ve .Net Mauı Blazor Bildirim İşleme_

##### _.Net Core_

```ruby
public FCMNotificationService() 
        {
            var defaultApp = FirebaseApp.Create(new AppOptions()
            {
                Credential = GoogleCredential.FromFile(Path.Combine(Environment.CurrentDirectory, "argusmydkey.json")),
                //Credential = GoogleCredential.FromFile(Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "argusmyd-key.json")),
            });
        }
```
```ruby
public async Task<bool> SendNotification(Announcement announcement)
        {
            try
            {
                var myToken = "cIZpyJxur0...:APA91bHVfDXHc7.....";
                var message = new Message()
                {
                    Notification = new Notification()
                    {
                        Title = "Yeni Bir Duyuru Eklendi!",
                        Body = "'" + announcement.Title + "'" + " başlıklı duyuru eklendi. - with token"
                    },
                    Token = myToken,
                };

                string response = await FirebaseMessaging.DefaultInstance.SendAsync(message);

                return true;
            }
            catch (System.Exception ex)
            {
                return false;
            }
        }

        public async Task<bool> SendNotificationToAllUsers(Announcement announcement)
        {
            try
            {
                var message = new Message()
                {
                    Notification = new Notification()
                    {
                        Title = "Yeni Bir Duyuru Eklendi!",
                        Body = "'" + announcement.Title + "'" + " başlıklı duyuru eklendi."
                    },
                    Topic = "allUsers",
                };

                string response = await FirebaseMessaging.DefaultInstance.SendAsync(message);

                return true;
            }
            catch (System.Exception ex)
            {
                return false;
            }
        }

        public async Task<bool> SendNotificationToOrganizationUsers(Announcement announcement)
        {
            try
            {
                var message = new Message()
                {
                    Data = new Dictionary<string, string>()
                    {
                        { "AnnouncementId", announcement.Id },
                    },
                    Notification = new Notification()
                    {
                        Title = "Yeni Bir Duyuru Eklendi!",
                        Body = "'" + announcement.Title + "'" + " başlıklı duyuru eklendi."
                    },
                    Topic = announcement.OrganizationId,
                };

                string response = await FirebaseMessaging.DefaultInstance.SendAsync(message);

                return true;
            }
            catch (System.Exception ex)
            {
                return false;
            }
        }
```

##### _.Net Mauı Blazor_

```ruby
[Export("userNotificationCenter:didReceiveNotificationResponse:withCompletionHandler:")]
    public void DidReceiveNotificationResponse(UNUserNotificationCenter center, UNNotificationResponse response, Action completionHandler)
    {
        NSDictionary userInfo = response.Notification.Request.Content.UserInfo;
        //Messaging.SharedInstance.AppDidReceiveMessage(userInfo);

        if (userInfo.ContainsKey(new NSString("AnnouncementId")))
        {
            var annId = userInfo[new NSString("AnnouncementId")].ToString();
            Preferences.Default.Set("AnnouncementId", annId);
        }

        completionHandler();
    }
```
```ruby
var notificationAnnId = Preferences.Default.Get("AnnouncementId", "");
if (notificationAnnId == "")
{
	navMng.NavigateTo("/home");
}
else
{
	//Preferences.Default.Set("AnnouncementId", "");
	Preferences.Default.Remove("AnnouncementId");
	await AppService.Instance.SetDisplayedAnnouncement(notificationAnnId);
	navMng.NavigateTo("/displayannouncement");
}
```


## ~~~
### _Çoklu Dil Seçeneği - Localization_

##### _Resource Dosyalarının Eklenmesi_
![image](https://github.com/ezginurr/.Net-MAUI-Blazor-Notlar/assets/48153438/712614fe-358b-48db-836d-c12991143740)

##### _MauiProgram.cs Dosyasındaki Düzenlemeler_

Microsoft.Extensions.Localization paketi indirildi ve aşağıdaki ekleme yapıldı.

```ruby
builder.Services.AddLocalization();
```

##### _MainPage.xaml.cs Dosyasındaki Düzenlemeler_

```ruby
public MainPage()
	{
		InitializeComponent();

        var lang = Preferences.Default.Get("language", "tr");
        var ci = new CultureInfo(lang);
        CultureInfo.CurrentCulture = ci;
        CultureInfo.CurrentUICulture = ci;
    }
```

##### _Dil Seçiminin Yapıldığı Razor Sayfasındaki Düzenlemeler_

```ruby
@inject IStringLocalizer<LangResources> Localizer
@using System.Globalization;
```
```ruby
<div class="login-page">
        <div class="ring">
            @Localizer["Loading"]
            <span></span>
        </div>
    </div>
```
```ruby
<a @onclick="SelectTR">Türkçe</a>
<a @onclick="SelectEN">English</a>
```
```ruby
private async void SelectTR()
    {
        try
        {
            Preferences.Default.Set("language", "tr");

            (Application.Current as App).MainPage.Dispatcher.Dispatch(() =>
            {
                CultureInfo cultureInfo = new CultureInfo("tr");
                CultureInfo.DefaultThreadCurrentUICulture = cultureInfo;
                CultureInfo.DefaultThreadCurrentCulture = cultureInfo;
                Thread.CurrentThread.CurrentCulture = cultureInfo;
                Thread.CurrentThread.CurrentUICulture = cultureInfo;
            });
        }
        catch (Exception ex) { }
        await InvokeAsync(StateHasChanged);
    
    }

    private async void SelectEN()
    {
        try 
        {
           Preferences.Default.Set("language", "en");
        
           (Application.Current as App).MainPage.Dispatcher.Dispatch(() =>
           {
               CultureInfo cultureInfo = new CultureInfo("en");
               CultureInfo.DefaultThreadCurrentUICulture = cultureInfo;
               CultureInfo.DefaultThreadCurrentCulture = cultureInfo;
               Thread.CurrentThread.CurrentCulture = cultureInfo;
               Thread.CurrentThread.CurrentUICulture = cultureInfo;
           });
        }
        catch (Exception ex) { }
       await InvokeAsync(StateHasChanged);
    }
```

##### _Diğer Razor Sayfalarındaki Düzenlemeler_

```ruby
@inject IStringLocalizer<LangResources> Localizer
```
```ruby
<h3>@Localizer["Welcome"] @_user.FirstName!</h3>
```



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
