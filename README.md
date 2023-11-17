# _.Net-MAUI-Blazor-Notlar_

## ~~~
### _Apple Uygulama Yayınlama Aşamaları_

##### _Geliştirilen Uygulamanın Apple Test Flight Ortamına Yüklenmesi_

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
