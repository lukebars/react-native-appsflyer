# API


- [initSdk](#initSdk)
- [trackAppLaunch](#trackAppLaunch)
- [onInstallConversionData](#onInstallConversionData) 
- [onAppOpenAttribution](#onAppOpenAttribution) 
- [trackEvent](#trackEvent)
- [setCustomerUserId](#setCustomerUserId) 
- [getAppsFlyerUID](#getAppsFlyerUID) 
- [stopTracking](#stopTracking) 
- [trackLocation](#trackLocation) 
- [setUserEmails](#setUserEmails) 
- [setAdditionalData](#setAdditionalData) 
- [sendDeepLinkData](#sendDeepLinkData) 
- [updateServerUninstallToken](#updateServerUninstallToken) 
- [setCollectIMEI](#setCollectIMEI) 
- [setCollectAndroidID](#setCollectAndroidID) 
- [setAppInviteOneLinkID](#setAppInviteOneLinkID) 
- [generateInviteLink](#generateInviteLink) 
- [trackCrossPromotionImpression](#trackCrossPromotionImpression) 
- [trackAndOpenStore](#trackAndOpenStore) 
- [setCurrencyCode](#setCurrencyCode) 


---

##### <a id="initSdk"> **`initSdk(options, success, error)`**

Initialize the AppsFlyer SDK with the devKey and appID.<br/>
The dev key is required for all apps and the appID is required only for iOS.<br/>
(you may pass the appID on Android as well, and it will be ignored)<br/>

| parameter    | type     | description                               |
| -----------  |----------|------------------------------------------ |
| options      | json     | init options                              |
| success      | function | success callback                          |
| error        | function | error callback                          |


| Option  | Description   |
| -------- | ------------- |
| devKey   | Your application [devKey](https://support.appsflyer.com/hc/en-us/articles/211719806-Global-app-settings-#sdk-dev-key) provided by AppsFlyer (required)  |
| appId      | Your iTunes [application ID](https://support.appsflyer.com/hc/en-us/articles/207377436-Adding-a-new-app#available-in-the-app-store-google-play-store-windows-phone-store)  (iOS only)  |
| isDebug    | Debug mode - set to `true` for testing only  |

*Example:*

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
  (res) => {
    console.log(res);
  },
  (err) => {
    console.error(err);
  }
);
```

With Promise:
```javascript
try {
  var res = await appsFlyer.initSdk(options);
} catch (err) {}
```

---
##### <a id="trackAppLaunch"> **`trackAppLaunch()`**

(iOS Only)

Necessary for tracking sessions and deep link callbacks in iOS on background-to-foreground transitions.<br/>
This API Should be used with the relevant [AppState](https://facebook.github.io/react-native/docs/appstate.html) logic.

*Example:*

```javascript
state = {
  appState: AppState.currentState,
};

componentDidMount();
{
  AppState.addEventListener('change', this._handleAppStateChange);
}

componentWillUnmount();
{
  AppState.removeEventListener('change', this._handleAppStateChange);
}

_handleAppStateChange = (nextAppState) => {
  if (this.state.appState.match(/inactive|background/) && nextAppState === 'active') {
    if (Platform.OS === 'ios') {
      appsFlyer.trackAppLaunch();
    }
  }

  this.setState({appState: nextAppState});
};
```


Or with Hooks:

```javascript
const [appState, setAppState] = useState(AppState.currentState);

useEffect(() => {
  function handleAppStateChange(nextAppState) {
    if (appState.match(/inactive|background/) && nextAppState === 'active') {
      if (Platform.OS === 'ios') {
        appsFlyer.trackAppLaunch();
      }
    }

    setAppState(nextAppState);
  }

  AppState.addEventListener('change', handleAppStateChange);

  return () => {
    AppState.removeEventListener('change', handleAppStateChange);
  };
});
```

---

##### <a id="onInstallConversionData"> **`onInstallConversionData(callback) : function:unregister`**

Accessing AppsFlyer Attribution / Conversion Data from the SDK (Deferred Deeplinking).<br/>

The code implementation for the conversion listener must be made prior to the initialization code of the SDK.


| parameter    | type     | description                               |
| -----------  |----------|------------------------------------------ |
| callback     | function | conversion data result                    |

*Example:*

```javascript
this.onInstallConversionDataCanceller = appsFlyer.onInstallConversionData(
  (res) => {
    if (JSON.parse(res.data.is_first_launch) == true) {
      if (res.data.af_status === 'Non-organic') {
        var media_source = res.data.media_source;
        var campaign = res.data.campaign;
        alert('This is first launch and a Non-Organic install. Media source: ' + media_source + ' Campaign: ' + campaign);
      } else if (res.data.af_status === 'Organic') {
        alert('This is first launch and a Organic Install');
      }
    } else {
      alert('This is not first launch');
    }
  }
);

appsFlyer.initSdk(/*...*/);
```

*Example onInstallConversionData:*

```javascript
{
  "data": {
    "af_message": "organic install",
    "af_status": "Organic",
    "is_first_launch": "true"
  },
  "status": "success",
  "type": "onInstallConversionDataLoaded"
}
```

> **Note** is_first_launch will be "true" (string) on Android and true (boolean) on iOS. To solve this issue wrap is_first_launch with JSON.parse(res.data.is_first_launch) as in the example above.

`appsFlyer.onInstallConversionData` returns a function the will allow us to call `NativeAppEventEmitter.remove()`.<br/>
Therefore it is required to call the canceller method in the next [appStateChange](https://facebook.github.io/react-native/docs/appstate.html) callback. <br/>This is required for both the `onInstallConversionData` and `onAppOpenAttribution` callbacks.

*Example:*

```javascript
 _handleAppStateChange = (nextAppState) => {
    if (this.state.appState.match(/active|foreground/) && nextAppState === 'background') {
      if (this.onInstallConversionDataCanceller) {
        this.onInstallConversionDataCanceller();
        this.onInstallConversionDataCanceller = null;
      }
      if (this.onAppOpenAttributionCanceller) {
        this.onAppOpenAttributionCanceller();
        this.onAppOpenAttributionCanceller = null;
      }
    }
```



---

##### <a id="onAppOpenAttribution"> **`onAppOpenAttribution(callback) : function:unregister`**


| parameter    | type     | description                               |
| -----------  |----------|------------------------------------------ |
| callback     | function | deeplink data result                    |

*Example:*

```javascript
this.onAppOpenAttributionCanceller = appsFlyer.onAppOpenAttribution((res) => {
  console.log(res);
});

appsFlyer.initSdk(/*...*/);
```

---

##### <a id="trackEvent"> **`trackEvent(eventName, eventValues, success, error)`**

In-App Events provide insight on what is happening in your app. It is recommended to take the time and define the events you want to measure to allow you to measure ROI (Return on Investment) and LTV (Lifetime Value).

Recording in-app events is performed by calling sendEvent with event name and value parameters. See In-App Events documentation for more details.

**Note:** An In-App Event name must be no longer than 45 characters. Events names with more than 45 characters do not appear in the dashboard, but only in the raw Data, Pull and Push APIs.

| parameter    | type     | description                                   |
| -----------  |----------|------------------------------------------     |
| eventName    | string   | The name of the event                         |
| eventValues  | json     | The event values that are sent with the event |
| success      | function | success callback                              |
| error        | function | success callback                              |

*Example:*

```javascript
const eventName = 'af_add_to_cart';
const eventValues = {
  af_content_id: 'id123',
  af_currency: 'USD',
  af_revenue: '2',
};

appsFlyer.trackEvent(
  eventName,
  eventValues,
  (res) => {
    console.log(res);
  },
  (err) => {
    console.error(err);
  }
);
```

---

##### <a id="setCustomerUserId"> **`setCustomerUserId(userId, callback)`**

Setting your own Custom ID enables you to cross-reference your own unique ID with AppsFlyer’s user ID and the other devices’ IDs. This ID is available in AppsFlyer CSV reports along with postbacks APIs for cross-referencing with you internal IDs.


| parameter | type     | description      |
| ----------|----------|------------------|
| userId    | string   | user ID          |
| callback  | function | success callback |


*Example:*

```javascript
appsFlyer.setCustomerUserId('some_user_id', (res) => {
  //..
});
```

---

##### <a id="getAppsFlyerUID"> **`getAppsFlyerUID(callback)`**

AppsFlyer's unique device ID is created for every new install of an app. Use the following API to obtain AppsFlyer’s Unique ID.


| parameter | type     | description                     |
| ----------|----------|------------------               |
| callback  | function | returns `(error, appsFlyerUID)` |


*Example:*

```javascript
appsFlyer.getAppsFlyerUID((err, appsFlyerUID) => {
  if (err) {
    console.error(err);
  } else {
    console.log('on getAppsFlyerUID: ' + appsFlyerUID);
  }
});
```

---

##### <a id="stopTracking"> **`stopTracking(isStopTracking, callback)`**

In some extreme cases you might want to shut down all SDK functions due to legal and privacy compliance. This can be achieved with the stopSDK API. Once this API is invoked, our SDK no longer communicates with our servers and stops functioning.

There are several different scenarios for user opt-out. We highly recommend following the exact instructions for the scenario, that is relevant for your app.

In any event, the SDK can be reactivated by calling the same API, by passing false.

| parameter       | type     | description                                          |
| ----------      |----------|------------------                                    |
| isStopTracking  | boolean  | True if the SDK is stopped (default value is false). |
| callback        | function | success callback                                     |


*Example:*

```javascript
appsFlyer.stopTracking(true, (res) => {
  //...
});
```

---

##### <a id="trackLocation"> **`trackLocation(longitude, latitude, callback)`**

Manually record the location of the user.

| parameter       | type     | description               |
| ----------      |----------|------------------         |
| longitude       | float    | longitude                 |
| latitude        | float    | latitude                  |
| callback        | function | Success / Error Callbacks |


*Example:*

```javascript
const latitude = -18.406655;
const longitude = 46.40625;

appsFlyer.trackLocation(longitude, latitude, (err, coords) => {
  if (err) {
    console.error(err);
  } else {
    //...
  }
});
```

---

##### <a id="setUserEmails"> **`setUserEmails(options, success, error)`**

Set the user emails and encrypt them.

| parameter       | type     | description               |
| ----------      |----------|------------------         |
| configuration   | json     | email configuration       |
| success         | function | success callback          |
| error           | function | error callback             |


| option          | type  | description   |
| --------------  | ----  |------------- |
| emailsCryptType | int   | none - 0 (default), SHA1 - 1, MD5 - 2 |
| emails          | array | comma separated list of emails |


*Example:*

```javascript
const options = {
  emailsCryptType: 2,
  emails: ['user1@gmail.com', 'user2@gmail.com'],
};

appsFlyer.setUserEmails(
  options,
  (res) => {
    //...
  },
  (err) => {
    console.error(err);
  }
);
```

---

##### <a id="setAdditionalData"> **`setAdditionalData(additionalData, callback)`**

The setAdditionalData API is required to integrate on the SDK level with several external partner platforms, including Segment, Adobe and Urban Airship. Use this API only if the integration article of the platform specifically states setAdditionalData API is needed. 

| parameter       | type     | description               |
| ----------      |----------|------------------         |
| additionalData  | json     | additional data           |
| callback        | function | success callback          |


*Example:*

```javascript
appsFlyer.setAdditionalData(
  {
    val1: 'data1',
    val2: false,
    val3: 23,
  },
  (res) => {
    //...
  }
);
```

---

##### <a id="sendDeepLinkData"> **`sendDeepLinkData(callback)`**
  
(Android only)

Report Deep Links for Re-Targeting Attribution (Android). This method should be called when an app is opened using a deep link.

| parameter       | type     | description               |
| ----------      |----------|------------------         |
| callback        | function | success callback |


*Example:*

```javascript
  componentDidMount() {
    Linking.getInitialURL()
      .then((url) => {
        if (appsFlyer) {
          appsFlyer.sendDeepLinkData(url); // Report Deep Link to AppsFlyer
          // Additional Deep Link Logic Here ...
        }
      })
      .catch((err) => console.error('An error occurred', err));
  }
```

More about Deep Links in React-Native: [React-Native Linking](https://facebook.github.io/react-native/docs/linking.html).<br/>
More about Deep Links in Android: [Android Deep Linking , Adding Filters](https://developer.android.com/training/app-indexing/deep-linking.html#adding-filters).

---

##### <a id="updateServerUninstallToken"> **`updateServerUninstallToken(token, callback)`**
  
(Android only)

Manually pass the Firebase / GCM Device Token for Uninstall measurement.
  
| parameter       | type     | description               |
| ----------      |----------|------------------         |
| token           | string   | FCM Token                 |
| callback        | function | success callback          |


*Example:*

```javascript
appsFlyer.updateServerUninstallToken('token', (res) => {
  //...
});
```

---

##### <a id="setCollectIMEI"> **`setCollectIMEI(isCollect, callback)`**
  
(Android only)

Opt-out of collection of IMEI.<br/>
If the app does NOT contain Google Play Services, device IMEI is collected by the SDK.<br/>
However, apps with Google play services should avoid IMEI collection as this is in violation of the Google Play policy.<br/>

| parameter       | type     | description               |
| ----------      |----------|------------------         |
| isCollect       | boolean  | opt-in boolean            |
| callback        | function | success callback          |


*Example:*

```javascript
appsFlyer.setCollectIMEI(false, (res) => {
   //...
});
```

---

##### <a id="setCollectAndroidID"> **`setCollectAndroidID(isCollect, callback)`**
  
(Android only)

Opt-out of collection of Android ID.<br/>
If the app does NOT contain Google Play Services, Android ID is collected by the SDK.<br/>
However, apps with Google play services should avoid Android ID collection as this is in violation of the Google Play policy.<br/>

| parameter       | type     | description               |
| ----------      |----------|------------------         |
| isCollect       | boolean  | opt-in boolean            |
| callback        | function | success callback          |


*Example:*

```javascript
appsFlyer.setCollectAndroidID(true, (res) => {
   //...
});
```

---

##### <a id="setAppInviteOneLinkID"> **`setAppInviteOneLinkID(oneLinkID, callback)`**
  
Set the OneLink ID that should be used for User-Invite-API.<br/>
The link that is generated for the user invite will use this OneLink ID as the base link ID.

| parameter       | type     | description               |
| ----------      |----------|------------------         |
| oneLinkID       | string   | oneLinkID                 |
| callback        | function | success callback          |


*Example:*

```javascript
appsFlyer.setAppInviteOneLinkID('abcd', (res) => {
  //...
});
```

---

##### <a id="generateInviteLink"> **`generateInviteLink(parameters, success, error)`**


| parameter       | type     | description                      |
| ----------      |----------|------------------                |
| parameters      | json     | parameters for Invite link       |
| success         | function | success callback (generated link)|
| error           | function | error callback                   |


*Example:*

```javascript
appsFlyer.generateInviteLink(
 {
   channel: 'gmail',
   campaign: 'myCampaign',
   customerID: '1234',
   userParams: {
     myParam: 'newUser',
     anotherParam: 'fromWeb',
     amount: 1,
   },
 },
 (link) => {
   console.log(link);
 },
 (err) => {
   console.log(err);
 }
);
```

A complete list of supported parameters is available [here](https://support.appsflyer.com/hc/en-us/articles/115004480866-User-Invite-Tracking). Custom parameters can be passed using a userParams{} nested object, as in the example above.

---

##### <a id="trackCrossPromotionImpression"> **`trackCrossPromotionImpression(appId, campaign)`**

To attribute an impression use the following API call.<br/>
Make sure to use the promoted App ID as it appears within the AppsFlyer dashboard.

| parameter       | type      | description               |
| ----------      |---------- |------------------         |
| appId           | string    | promoted application ID   |
| campaign        | string    | Promoted Campaign         |



*Example:*

```javascript
appsFlyer.trackCrossPromotionImpression("com.myandroid.app", "myCampaign");
```

For more details about Cross-Promotion tracking please see the relevent doc [here](https://support.appsflyer.com/hc/en-us/articles/115004481946-Cross-Promotion-Tracking).

---

##### <a id="trackAndOpenStore"> **`trackAndOpenStore(appId, campaign, params)`**

Use the following API to attribute the click and launch the app store's app page.

| parameter       | type     | description               |
| ----------      |----------|------------------         |
| appId           | string   | promoted application ID   |
| campaign        | string   | promoted campaign         |
| params          | json     | additional parameters     |


*Example:*

```javascript
var crossPromOptions = {
  customerID: '1234',
  myCustomParameter: 'newUser',
};

appsFlyer.trackAndOpenStore(
  'com.myandroid.app',
  'myCampaign',
  crossPromOptions
);
```

---

##### <a id="setCurrencyCode"> **`setCurrencyCode(currencyCode, callback)`**

Setting user local currency code for in-app purchases.<br/>
The currency code should be a 3 character ISO 4217 code. (default is USD).<br/>
You can set the currency code for all events by calling the following method.<br/>

| parameter       | type     | description               |
| ----------      |----------|------------------         |
| currencyCode    | string   | currencyCode              |
| callback        | function | success callback          |


*Example:*

```javascript
appsFlyer.setCurrencyCode(currencyCode, () => {});
```

---
