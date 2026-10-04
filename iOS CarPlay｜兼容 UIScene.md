## Совместимость с UIScene

Для разработки CarPlay-приложений с помощью CarPlay framework на iOS 14 и выше необходимо использовать UIScene (UIScene был представлен Apple в iOS 13 для создания многооконных приложений), поэтому ваш проект должен перейти с традиционной связки UIWindow и AppDelegate на SceneDelegate. Если ваш проект уже поддерживает UIScene, этот шаг можно пропустить; если нет — следуйте шагам из этого раздела.

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/201ca825e81f427e88ed74560cb1f8ae~tplv-k3u1fbpfcp-watermark.image?)

### Что такое UIScene

До iOS 13 обязанности распределялись так: UIApplication отвечал за состояние приложения, а UIApplicationDelegate (AppDelegate) — за события и жизненный цикл приложения, включая процесс и UI. Для одноокного приложения это не проблема, но для разработки многооконных приложений на iPad или Mac Catalyst такое разделение обязанностей уже не подходит.

Поэтому в iOS 13 Apple представила UIScene для создания многооконных приложений и разделила обязанности: состояние, события и жизненный цикл, связанные с UI, перешли к [UIWindowScene](https://developer.apple.com/documentation/uikit/uiwindowscene/) и [UIWindowSceneDelegate](https://developer.apple.com/documentation/uikit/uiwindowscenedelegate/) (SceneDelegate), а [UISceneSession](https://developer.apple.com/documentation/uikit/uiscenesession/) отвечает за персистентное состояние UI.

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6ceb358854349a49eee654d4796940f~tplv-k3u1fbpfcp-watermark.image?)

### Совместимость с UIScene 

Поскольку UIScene доступен только на iOS 13 и выше, то если минимальная поддерживаемая версия вашего приложения ниже iOS 13, полностью перейти на SceneDelegate не получится: на iOS 13 и выше используется AppDelegate + SceneDelegate, а на версиях ниже iOS 13 по-прежнему только AppDelegate.

#### Объявление UIWindowScene в Info.plist

Добавьте в Info.plist следующие key-value. Пояснения к параметрам:

* Enable Multiple Windows нужно установить в NO, иначе ваше iPad-приложение будет поддерживать несколько окон (если проекты для iPhone и iPad находятся в одном проекте).
* Application Session Role — массив, в котором настраиваются сцены вашего приложения; у каждого элемента 4 параметра:
  * Class Name: тип Scene
  * Configuration Name: имя текущей конфигурации
  * Delegate Class Name: с каким классом-делегатом Scene связана конфигурация
  * StoryBoard name: какой storyBoard использует эта Scene, если он есть

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/50a9bf67131049c297b58ef719e18f36~tplv-k3u1fbpfcp-watermark.image?)

```xml
<key>UIApplicationSceneManifest</key>
<dict>
    <key>UIApplicationSupportsMultipleScenes</key>
        <false/>
    <key>UISceneConfigurations</key>
    <dict>
        <key>UIWindowSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>UIWindowScene</string>
                <key>UISceneConfigurationName</key>
                <string>DefaultSceneConfiguration</string>
                <key>UISceneDelegateClassName</key>
                <string>$(PRODUCT_MODULE_NAME).SceneDelegate</string>
            </dict>
        </array>
        <key>CPTemplateApplicationSceneSessionRoleApplication</key>
        // ...
        // CPTemplateApplicationScene
        // ...
    </dict>
</dict>
```

#### Project

**Targets > General > Deployment Info > Supports multiple windows** — снимите галочку. Состояние этой опции влияет на значение Enable Multiple Windows в Info.plist.

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98192ea75aa84ed6b0dda12f6dc95bd7~tplv-k3u1fbpfcp-watermark.image?)

#### Изменения в AppDelegate

Из-за изменения обязанностей классов часть реализаций, которые раньше находились в API AppDelegate, нужно перенести в API SceneDelegate.

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/840edc05115e4d0a948c7d713437ea40~tplv-k3u1fbpfcp-watermark.image?)

```objectivec
@implementation AppDelegate

- (UIWindow *)window {
    if (@available(iOS 13, *)) {
        return [(SceneDelegate *)TTScene.main.delegate window];
    } else {
        return _window;
    }
}

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    ...
    if (@available(iOS 13.0, *)) {} else {
        // 1. create window
        // 2. do something after window created. Обратите внимание: код, который раньше выполнялся только после создания window, тоже нужно сделать совместимым с iOS 13
    }
    ...
    return YES;
}

@end
```

Следующие методы реализуйте по необходимости.

```swift
@available(iOS 13, *)
extension AppDelegate {

    /*
     1. Если в файле Info.plist приложения нет данных конфигурации сцены или конфигурацию сцены нужно менять динамически, необходимо реализовать этот метод. UIKit вызывает его перед созданием новой сцены.
     2. Метод возвращает объект UISceneConfiguration, содержащий сведения о сцене: тип создаваемой сцены, объект-делегат для управления сценой и storyboard с начальным view controller для отображения. Если метод не реализован, данные конфигурации сцены должны быть указаны в Info.plist приложения.

     Итог: по умолчанию конфигурация задана в Info.plist, поэтому этот метод можно не реализовывать. Если конфигурации нет, нужно реализовать метод и вернуть объект UISceneConfiguration.
     В параметрах конфигурации Application Session Role — это массив, у каждого элемента три параметра:
         1) Configuration Name:   имя текущей конфигурации;
         2) Delegate Class Name:  с каким объектом-делегатом Scene связана конфигурация;
         3) StoryBoard name: какой storyboard использует эта Scene.
     Примечание: если в методе делегата вызывается Scene с именем конфигурации Default Configuration, система автоматически обращается к классу SceneDelegate. Так SceneDelegate и AppDelegate оказываются связаны.
     */
    func application(_ application: UIApplication, configurationForConnecting connectingSceneSession: UISceneSession, options: UIScene.ConnectionOptions) -> UISceneConfiguration {
        // Called when a new scene session is being created.
        // Use this method to select a configuration to create the new scene with.
        return UISceneConfiguration(name: "Default Configuration", sessionRole: connectingSceneSession.role)
    }

    // Вызывается, когда в режиме разделённого экрана закрывается одна или несколько scene
    func application(_ application: UIApplication, didDiscardSceneSessions sceneSessions: Set<UISceneSession>) {
        // Called when the user discards a scene session.
        // If any sessions were discarded while the application was not running, this will be called shortly after application:didFinishLaunchingWithOptions.
        // Use this method to release any resources that were specific to the discarded scenes, as they will not return.
    }
}
```

#### SceneDelegate

```swift
@available(iOS 13, *)
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    let configurationName = "DefaultSceneConfiguration"
    @objc var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let windowScene = (scene as? UIWindowScene), session.configuration.name == configurationName else { return }
        // 1. create window
        let window = UIWindow(windowScene: windowScene)
        // ...
        self.window = window
        window.makeKeyAndVisible()
        // 2. do something after window created
    }
  
    func sceneDidDisconnect(_ scene: UIScene) {
        guard scene.session.configuration.name == configurationName else { return }
    }

    func sceneDidBecomeActive(_ scene: UIScene) {
        guard scene.session.configuration.name == configurationName else { return }
        UIApplication.shared.delegate?.applicationDidBecomeActive?(UIApplication.shared)
    }

    func sceneWillResignActive(_ scene: UIScene) {
        guard scene.session.configuration.name == configurationName else { return }
        UIApplication.shared.delegate?.applicationWillResignActive?(UIApplication.shared)
    }

    func sceneWillEnterForeground(_ scene: UIScene) {
        guard scene.session.configuration.name == configurationName else { return }
        UIApplication.shared.delegate?.applicationWillEnterForeground?(UIApplication.shared)
    }

    func sceneDidEnterBackground(_ scene: UIScene) {
        guard scene.session.configuration.name == configurationName else { return }
        UIApplication.shared.delegate?.applicationDidEnterBackground?(UIApplication.shared)
    }

    func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        guard scene.session.configuration.name == configurationName else { return }
        guard let url = URLContexts.first?.url else { return }
        _ = UIApplication.shared.delegate?.application?(UIApplication.shared, open: url, options: [:])
    }
    
    func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
        guard scene.session.configuration.name == configurationName else { return }
        _ = UIApplication.shared.delegate?.application?(UIApplication.shared, continue: userActivity, restorationHandler: { _ in })
    }
}
```

#### Расширение UIScene для получения сцен

```swift
import UIKit
import CarPlay

@available(iOS 13.0, *)
extension UIScene {

    private static var connectedScenes: Set<UIScene> {
        UIApplication.shared.connectedScenes
    }
    
    static var main: UIWindowScene? {
        connectedScenes.first(where: { $0 is UIWindowScene }) as! UIWindowScene?
    }
    
    static var carPlay: CPTemplateApplicationScene? {
        connectedScenes.first(where: { $0 is CPTemplateApplicationScene }) as! CPTemplateApplicationScene?
    }
}
```

#### UIWindow

После перехода на UIScene иерархия UI изменилась: между прежними слоями UIScreen и UIWindow добавился слой UIWindowScene.

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af3a8af87c2a4c3db1d36a68e5a80626~tplv-k3u1fbpfcp-watermark.image?)

У UIWindow также появились свойство windowScene и инициализатор с windowScene. Чтобы UIWindow отображался на экране, его необходимо либо инициализировать с windowScene, либо установить свойство windowScene.

```objectivec
// instantiate a UIWindow already associated with a given UIWindowScene instance, with matching frame & interface orientations.
- (instancetype)initWithWindowScene:(UIWindowScene *)windowScene API_AVAILABLE(ios(13.0));

// If nil, window will not appear on any screen.
// changing the UIWindowScene may be an expensive operation and should not be done in performance-sensitive code
@property (nullable, nonatomic, weak) UIWindowScene *windowScene API_AVAILABLE(ios(13.0));
```

#### Адаптация окна о конфиденциальности при первом запуске

Если ваше окно о конфиденциальности при первом запуске перехватывается через hook метода `- application:didFinishLaunchingWithOptions:` в методе init класса AppDelegate, то нужно также сделать hook метода `- scene:willConnectToSession:options:` и перенести момент показа окна о конфиденциальности туда. Метод `- scene:willConnectToSession:options:` вызывается после возврата из `- application:didFinishLaunchingWithOptions:`.

### Дополнительные материалы

* [WWDC19｜Introducing Multiple Windows on iPad](https://developer.apple.com/videos/play/wwdc2019/212)
    * [WWDC19 Insider｜Мультиоконность на iPad](https://xiaozhuanlan.com/topic/0342159876)
* [Apple｜Specifying the Scenes Your App Supports](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/specifying_the_scenes_your_app_supports?language=objc)
