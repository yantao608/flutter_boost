<p align="center">
  <img src="flutter_boost.png">
   <b></b><br>
  <a href="README_CN.md">中文文档</a>
  <br><br>
</p>

# FlutterBoost
A next-generation Flutter-Native hybrid solution. FlutterBoost is a Flutter plugin which enables hybrid integration of Flutter for your existing native apps with minimum efforts.The philosophy of FlutterBoost is to use Flutter as easy as using a WebView. Managing Native pages and Flutter pages at the same time is non-trivial in an existing App. FlutterBoost takes care of page resolution for you. The only thing you need to care about is the name of the page(usually could be an URL). 
<a name="bf647454"></a>

# Prerequisites
You need to add Flutter to your project before moving on.

# Getting Started


## Add a dependency in you Flutter project.

Open you pubspec.yaml and add the following line to dependencies:

```java
flutter_boost: ^0.0.400
```


## Integration with Flutter code.
Add init code to you App

```dart
void main() => runApp(MyApp());

class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  @override
  void initState() {
    super.initState();

    ///register page widget builders,the key is pageName
    FlutterBoost.singleton.registerPageBuilders({
      'sample://firstPage': (pageName, params, _) => FirstRouteWidget(),
      'sample://secondPage': (pageName, params, _) => SecondRouteWidget(),
    });

    ///query current top page and load it
    FlutterBoost.handleOnStartPage();
  }

  @override
  Widget build(BuildContext context) => MaterialApp(
      title: 'Flutter Boost example',
      builder: FlutterBoost.init(), ///init container manager
      home: Container());
}
```


## Integration with iOS code.

Use FLBFlutterAppDelegate as the superclass of your AppDelegate

```objc
@interface AppDelegate : FLBFlutterAppDelegate <UIApplicationDelegate>
@end
```


Implement FLBPlatform protocol methods for your App.

```objc
@interface DemoRouter : NSObject<FLBPlatform>

@property (nonatomic,strong) UINavigationController *navigationController;

+ (DemoRouter *)sharedRouter;

@end


@implementation DemoRouter

- (void)openPage:(NSString *)name
          params:(NSDictionary *)params
        animated:(BOOL)animated
      completion:(void (^)(BOOL))completion
{
    if([params[@"present"] boolValue]){
        FLBFlutterViewContainer *vc = FLBFlutterViewContainer.new;
        [vc setName:name params:params];
        [self.navigationController presentViewController:vc animated:animated completion:^{}];
    }else{
        FLBFlutterViewContainer *vc = FLBFlutterViewContainer.new;
        [vc setName:name params:params];
        [self.navigationController pushViewController:vc animated:animated];
    }
}


- (void)closePage:(NSString *)uid animated:(BOOL)animated params:(NSDictionary *)params completion:(void (^)(BOOL))completion
{
    FLBFlutterViewContainer *vc = (id)self.navigationController.presentedViewController;
    if([vc isKindOfClass:FLBFlutterViewContainer.class] && [vc.uniqueIDString isEqual: uid]){
        [vc dismissViewControllerAnimated:animated completion:^{}];
    }else{
        [self.navigationController popViewControllerAnimated:animated];
    }
}

@end
```



Initialize FlutterBoost with FLBPlatform at the beginning of your App.

```objc
 [FlutterBoostPlugin.sharedInstance startFlutterWithPlatform:router
                                                        onStart:^(FlutterViewController *fvc) {
                                                            
                                                        }];
```

## Integration with Android code.

Init FlutterBoost in Application.onCreate() 

```java
public class MyApplication extends FlutterApplication {
    @Override
    public void onCreate() {
        super.onCreate();
        FlutterBoostPlugin.init(new IPlatform() {
            @Override
            public Application getApplication() {
                return MyApplication.this;
            }

            /**
             * get the main activity, this activity should always at the bottom of task stack.
             */
            @Override
            public Activity getMainActivity() {
                return MainActivity.sRef.get();
            }

            @Override
            public boolean isDebug() {
                return false;
            }

            /**
             * start a new activity from flutter page, you may need a activity router.
             */
            @Override
            public boolean startActivity(Context context, String url, int requestCode) {
                return PageRouter.openPageByUrl(context,url,requestCode);
            }

            @Override
            public Map getSettings() {
                return null;
            }
        });
    }
```


# Basic Usage
## Concepts

All page routing requests are being sent to the native router. Native router communicates with Native Container Manager, Native Container Manager takes care of building and destroying of Native Containers. 


## Use Flutter Boost Native Container to show a Flutter page in native code.

iOS

```objc
 FLBFlutterViewContainer *vc = FLBFlutterViewContainer.new;
        [vc setName:name params:params];
        [self.navigationController presentViewController:vc animated:animated completion:^{}];
```

Android

```java
public class FlutterPageActivity extends BoostFlutterActivity {

    @Override
    public void onRegisterPlugins(PluginRegistry registry) {
        //register flutter plugins
        GeneratedPluginRegistrant.registerWith(registry);
    }

    @Override
    public String getContainerName() {
        //specify the page name register in FlutterBoost
        return "sample://firstPage";
    }

    @Override
    public Map getContainerParams() {
        //params of the page
        Map<String,String> params = new HashMap<>();
        params.put("key","value");
        return params;
    }
}
```

or

```java
public class FlutterFragment extends BoostFlutterFragment {
    @Override
    public void onRegisterPlugins(PluginRegistry registry) {
        GeneratedPluginRegistrant.registerWith(registry);
    }

    @Override
    public String getContainerName() {
        return "sample://firstPage";
    }

    @Override
    public Map getContainerParams() {
        Map<String,String> params = new HashMap<>();
        params.put("key","value");
        return params;
    }
}
```


## Use Flutter Boost to open a page in dart code.

Dart

```objc
 FlutterBoost.singleton.openPage("pagename", {}, true);
```

## Use Flutter Boost to close a page in dart code.

```objc
FlutterBoost.singleton.closePageForContext(context);
```

# Running the Demo
Please see the example for details.


# License
This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

# Acknowledgments
* Flutter
