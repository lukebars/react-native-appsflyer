
<img src="https://www.appsflyer.com/wp-content/uploads/2016/11/logo-1.svg"  width="450">

# React Native AppsFlyer plugin for Android and iOS. 

🛠 In order for us to provide optimal support, we would kindly ask you to submit any issues to support@appsflyer.com

*When submitting an issue please specify your AppsFlyer sign-up (account) email , your app ID , production steps, logs, code snippets and any additional relevant information.*

[![npm version](https://badge.fury.io/js/react-native-appsflyer.svg)](https://badge.fury.io/js/react-native-appsflyer) 


## Table of content

- [Adding the SDK to your project](#installation)
- [Initializing the SDK](#init-sdk)
- [Guides](#guides)
- [API](#api) 
  
### <a id="plugin-build-for"> This plugin is built for

- iOS AppsFlyerSDK **v5.0.0**
- Android AppsFlyerSDK **v5.0.0** 


## <a id="installation"> 📲 Adding the SDK to your project

```
$ npm install react-native-appsflyer --save
```

Then run the following:

*iOS*
```
$ cd ios && pod install
$ react-native run-ios
```

*Android*
```
$ react-native run-android
```

> Starting from RN [v0.60](https://facebook.github.io/react-native/blog/2019/07/03/version-60), and react-native-appsflyer `v1.4.7` the plugin uses [autolinking](https://github.com/react-native-community/cli/blob/master/docs/autolinking.md). <br/>
If your app does not support autolinking, check out the Installation Guide [here](./Docs/Installation.md).

## <a id="init-sdk"> 🚀 Initializing the SDK

Initialize the SDK to enable AppsFlyer to detect installations, sessions (app opens) and updates.  

```javascript
import React, {Component} from 'react';
import {Platform, StyleSheet, Text, View} from 'react-native';
import appsFlyer from 'react-native-appsflyer';

appsFlyer.initSdk(
  {
    devKey: 'K2***********99',
    isDebug: false,
    appId: '41*****44',
  },
  (result) => {
    console.log(result);
  },
  (error) => {
    console.error(error);
  }
);
```

| Setting  | Description   |
| -------- | ------------- |
| devKey   | Your application [devKey](https://support.appsflyer.com/hc/en-us/articles/211719806-Global-app-settings-#sdk-dev-key) provided by AppsFlyer (required)  |
| appId      | Your iTunes [application ID](https://support.appsflyer.com/hc/en-us/articles/207377436-Adding-a-new-app#available-in-the-app-store-google-play-store-windows-phone-store)  (iOS only)  |
| isDebug    | Debug mode - set to `true` for testing only  |


**Important** - For iOS another step is required. AppState logic is required to record Background-to-foreground transitions. Check out the [relevant guide](./Docs/API.md#--appsflyertrackapplaunch-void) to see how this mandatory step is implemented.

 ## <a id="guides"> 📖 Guides

Great installation and setup guides can be viewed [here](/Docs/Guides.md).
- [init SDK Guide](/Docs/Guides.md#init-sdk)
- [Deeplinking Guide](/Docs/Guides.md#deeplinking)
- [Uninstall Guide](/Docs/Guides.md#track-app-uninstalls)



## <a id="api"> 📑 API
  
See the full [API](/Docs/API.md) available for this plugin.

