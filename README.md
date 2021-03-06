# RN-Integration-with-existing-switf-app

嘗試將 React Native 與現有 swift app 做結合，教程與範例來源 [React Native Tutorial: Integrating in an Existing App](https://www.raywenderlich.com/136047/react-native-existing-app)，依據教程步驟，並在這邊紀錄一些重點要點。

教程中使用 RN 0.34，這邊則是使用 0.51，pods install 後，需要修正部分程式碼才能編譯成功或修改 import 的路徑，在 0.52.1 中仍然有些錯誤，無法將 swift app 編譯成功。可能需要手動改動的 issues:

* [#13198](https://github.com/facebook/react-native/issues/13198)

* [#17274](https://github.com/facebook/react-native/issues/17274)

* [#16192](https://github.com/facebook/react-native/pull/16192)

## Build Setup

```
# install react-native dependencies, cd js/
yarn

# install swift dependencies,  cd ios/
pod install

# run react-native, cd js/
yarn run start

# run swift app, Xcode open ios/Mixer/Mixer.xcworkspace, run ios simulators
```

## Swift 取得 RN Components

#### RN

透過 AppRegistry.registerComponent 方法， swift 可取得 components 添加到 ViewController

```
# js/index.js
AppRegistry.registerComponent('AddRatingApp', () => AddRatingApp);
```

#### swift

```
# ios/Mixer/AddRatingViewController.swif

import React

class AddRatingViewController: UIViewController {
  override func viewDidLoad() {
    super.viewDidLoad()

    // TODO: Add React View
    addRatingView = MixerReactModule.sharedInstance.viewForModule(
        "AddRatingApp",
        initialProperties: ["identifier": mixer.identifier, "currentRating": currentRating])
    self.view.addSubview(addRatingView)
  }

  override func viewDidLayoutSubviews() {
    super.viewDidLayoutSubviews()

    // TODO: Layout React View
    addRatingView.frame = self.view.bounds
  }
}
```

## Swift & RN 溝通與事件增聽

swift 預先準備好 RCTEventEmitter Manager 編寫 func methods，提供給 RN 呼叫發出事件，也因為 RN 是基於 objective C，所以需要 swift 需要引用 objective C 的方法：

#### Swift

```
# ios/Mixer/AddRatingManager.swif

@objc(AddRatingManager)
class AddRatingManager: RCTEventEmitter {
  @objc func dismissPresentedViewController(_ reactTag: NSNumber) {
    // todo
  }
}

# ios/Mixer/AddRatingManagerBridge.m
RCT_EXTERN_METHOD(dismissPresentedViewController:(nonnull NSNumber *)reactTag)
```

#### RN

RN NativeModules 可以取得 Switf 中的 RCTEventEmitter Modules

```
# js/AddRatingApps.js

import ReactNative, {
  NativeModules,
} from 'react-native';

const { AddRatingManager } = NativeModules;

// on click handler
AddRatingManager.dismissPresentedViewController(this.props.rootTag);
```

#### Swift send event to RN

透過 RN NativeEventEmitter 方法，可註冊 Swift 的事件方法，接收 Swift 發送的事件。

```
# ios/Mixer/AddRatingManager.swif

@objc(AddRatingManager)
class AddRatingManager: RCTEventEmitter {
  override func supportedEvents() -> [String]! {
    return ["AddRatingManagerEvent"]
  }
  @objc func save(_ reactTag: NSNumber, rating: Int, forIdentifier identifier: Int) -> Void {
        // todo
        self.sendEvent(
            withName: "AddRatingManagerEvent",
            body: ["name": "saveRating", "message": rating, "extra": identifier])
    }
}
```

```
# js/AddRatingApps.js

import ReactNative, {
  NativeEventEmitter,
} from 'react-native';

componentDidMount() {
  const AddRatingManagerEvent = new NativeEventEmitter(AddRatingManager);
  this._subscription = AddRatingManagerEvent.addListener(
    'AddRatingManagerEvent',
    (info) => {
      console.log(JSON.stringify(info));
    }
  );
}
```

### Other resources

* [younatics/React-Native-Integration-with-existing-app](https://github.com/younatics/React-Native-Integration-with-existing-app)

* [Yoga 教程: 使用跨平台布局引擎](https://archimboldi.me/posts/%E7%BF%BB%E8%AF%91-yoga-%E6%95%99%E7%A8%8B-%E4%BD%BF%E7%94%A8%E8%B7%A8%E5%B9%B3%E5%8F%B0%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E.html)
