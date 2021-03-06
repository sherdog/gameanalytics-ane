# GameAnalytics | Extension for Adobe AIR (iOS & Android)

ActionScript 3 API for the [GameAnalytics](http://www.gameanalytics.com/) native libraries.

## Native SDK versions

* iOS `v2.2.2`
* Android `v3.3.0`

## AIR SDK note

Including this and other extensions in your app increases the number of method references that must be stored in Android dex file. AIR currently supports a single dex file and since the number of such references is limited to a little over 65k, it is possible to exceed the limit by including several native extensions. This will prohibit you from building your app for Android, unless you reduce the number of features the app provides. Please, leave a vote in the report below to help adding multidex support to AIR SDK:

* [Bug 4190396 - Multidex support for Adobe AIR](https://bugbase.adobe.com/index.cfm?event=bug&id=4190396)

## Getting started

Create a game in the [GA dashboard](https://go.gameanalytics.com/home). Each platform has a separate game in the dashboard. That means you will have a different pair of key and secret for iOS and Android.

When setting up your Android game in the GA dashboard, pay attention to the Bundle Identifier settings. AIR apps for Android have their identifier prefixed with `air.` (unless you manually override this behavior). Thus the settings in the GA dashboard must reflect this.

### Additions to AIR descriptor

First, add the extension's ID to the `extensions` element.

```xml
<extensions>
    <extensionID>com.marpies.ane.gameanalytics</extensionID>
</extensions>
```

If you are targeting Android, add the two following extensions from [this repository](https://github.com/marpies/android-dependency-anes) as well (unless you know these libraries are included by some other extension):

```xml
<extensions>
    <extensionID>com.marpies.ane.googleplayservices.ads</extensionID>
    <extensionID>com.marpies.ane.googleplayservices.basement</extensionID>
</extensions>
```

Modify the `manifestAdditions` so that it contains the following permissions and meta data:

```xml
<android>
    <manifestAdditions>
        <![CDATA[
        <manifest android:installLocation="auto">
            <uses-permission android:name="android.permission.INTERNET" />
            <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>

            <uses-sdk android:targetSdkVersion="23" />

            <application>

                <meta-data android:name="com.google.android.gms.version" android:value="@integer/google_play_services_version" />

            </application>

        </manifest>
        ]]>
    </manifestAdditions>
</android>
```

After your descriptor is set up, add the GameAnalytics ANE package from the *bin* directory to your project so that your IDE can work with it. The Google Play Services ANEs are only necessary during packaging.

### Important: iOS app submission

The GA SDK for iOS uses *Advertising Identifier (IDFA)*. Make sure to read [this guide](http://www.gameanalytics.com/docs/ios-guide-for-app-submission) when submitting the app to the *App Store* to avoid rejection.

## API overview

The extension exposes most of the native APIs and for the most part you do not need to worry whether the target device is running iOS or Android.

### Configuration, initialization

There are some values that must be set before the analytics can be intialized, like app build and game's key/secret.

```as3
// Required values (at least 1 platform must be set)
GameAnalytics.config
    // iOS specific
    .setBuildiOS( "0.6.14")
    .setGameKeyiOS( "iOS-KEY" )
    .setGameSecretiOS( "iOS-SECRET" )
    // Android specific
    .setBuildAndroid( "1.0.0")
    .setGameKeyAndroid( "Android-KEY" )
    .setGameSecretAndroid( "Android-SECRET" )
```

Optionally, you can also configure the available currencies, item types and custom dimensions. These values specify what data your app is working with and what values can be used with the rest of the API.

```as3
GameAnalytics.config
    .setCustomDimensions01( new <String>[ "ninja", "samurai" ] )
    .setCustomDimensions02( new <String>[ "whale", "dolphin" ] )
    .setCustomDimensions03( new <String>[ "horde", "alliance" ] )
    .setResourceCurrencies( new <String>[ "gems", "coins" ] )
    .setResourceItemTypes( new <String>[ "boost", "lives" ] );
```

Finally, you can intialize the extension using:
```as3
if( GameAnalytics.init() ) {
    // Extension and the native library has been successfully initialized
}
```

### Events

For a quick overview about what data you can track in your app, visit [the official docs](http://www.gameanalytics.com/docs/ga-data).

#### Business event

With the business event, you can include information on the specific type of in-app item purchased, and where in the game the purchase was made.

```as3
GameAnalytics.addBusinessEvent( "USD", 199, "coinPack", "coin_pack_100", "storeScreen", "**receipt**", "**signature**", true );
```

* The `signature` parameter is applicable for Android only. If set to `null` the purchase will be registered as **not validated**.
* The last `Boolean` parameter is applicable for iOS only. When set to `true`, the native SDK will attempt to retrieve the purchase receipt automatically (provided the receipt is `null`).

#### Resource event (virtual currencies)

Use this event to track when players gain (source) or lose (sink) resources like virtual currency and lives.

```as3
GameAnalytics.addResourceEvent( GAResourceFlowType.SOURCE, "gems", 100, "reward", "videoAd" );
```

#### Progression event

Use this event to track when players start and finish levels in your game. This event follows a 3 hierarchy structure (for example World, Level and Phase) to indicate a player’s path or place in the game.

```as3
GameAnalytics.addProgressionEvent( GAProgressionStatus.FAIL, "winterArea", "level01", "wave04", 1337 );
```

#### Error event

You can use the Error event to log errors or warnings generated by your players’ in-game behaviour.

```as3
GameAnalytics.addErrorEvent( GAErrorSeverity.WARNING, "Player has insanely high score. CHEATER!" );
```

#### Design event

Track any other concept in your game using this event type. For example, you could use this event to track GUI elements or tutorial steps. Visit [the official docs](http://www.gameanalytics.com/docs/custom-events) to learn how to design such events.

```as3
GameAnalytics.addDesignEvent( "guiClick:volume:on" );
```

### Custom dimensions

You can tag your players with any piece of information that might help you in your analysis using Custom Dimensions. These dimensions can be used to analyze user segments that you define, for example, A/B test groups or whether or not a user is in the Warrior Guild.

There are three Custom Dimensions available to send. These dimensions are sent with every event type, excluding the Design event.

To set the individual dimensions, use the following methods:

```as3
GameAnalytics.setDimension01( "ninja" );
GameAnalytics.setDimension02( "dolphin" );
GameAnalytics.setDimension03( "horde" );
```

## Documentation
Generated ActionScript documentation is available in the *docs* directory, or can be generated by running `ant asdoc` from the *build* directory.

## Build ANE
ANT build scripts are available in the *build* directory. Edit *build.properties* to correspond with your local setup.

## Author
The ANE has been written by [Marcel Piestansky](https://twitter.com/marpies) and is distributed under [Apache License, version 2.0](http://www.apache.org/licenses/LICENSE-2.0.html).

The README contains information that was taken directly from [the GA docs](http://www.gameanalytics.com/docs).

## Changelog

#### August 8, 2016 (v1.1.0)

* UPDATED GameAnalytics iOS and Android SDKs
* ADDED `advertisingId` and `isLimitedAdTracking` APIs
* ADDED prefix to iOS functions to minimize chance of duplicate symbol errors

#### July 19, 2016 (v1.0.2)

* UPDATED AIRExtHelpers.framework for compatibility with other ANEs

#### June 9, 2016 (v1.0.1)

* UPDATED iOS framework deploy target to iOS 7.0, FIXED incorrect init conditional checks

#### June 5, 2016 (v1.0.0)

* Public release
