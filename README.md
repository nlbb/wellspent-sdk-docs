# Wellspent iOS SDK

Wellspent is a Screen Time SDK that allows you to lock distracting apps until your users complete a specific action in your app. It's plug-and-play and comes with pre-built UI screens you can customize to your IU. Most apps get their native screen time feature live within 2-3 days with the SDK.

**Here's how**

1. Configure Wellspent SDK, Wellspent's Screen Time SDK within your app.
2. Wellspent SDK locks distracting apps until your users complete an action in your app, based on your SDK settings.

## Key Features
* **Event-based App Blocking**: Full screen take-over that reminds your users to use your app when they open distracting apps during their self-set schedule. Distracting apps are unlocked after a specific event in your app is fired (e.g. session started, session completed).

* **Custom UI Control**: You can customize our pre-built UI to have full control over the visual style of the screen time feature in your app (onboarding screens, full-screen nudges, settings screen).

* **Analytics**: Analyze adoption and nudging rates in real-time, either in your analytics provider or with our dashboards.

## Prerequisites

* iOS 16.0+
* Swift 5.9+
* Xcode 16.0+

## Quick Overview

```swift

public class Wellspent {
   static let shared: Wellspent
   func configure(properties: WellspentSDKProperties, theme: WellspentTheme? = nil, trackingHandler: WellspentTrackingService? = nil) async -> Bool
   func start(onComplete: (() -> Void)? = nil, onDismiss: (() -> Void)? = nil)
   func showNudgeSettings(onFinish: (() -> Void)? = nil)
   func completeHabit()
   func isHabitCompleted() -> Bool?
}

```

## Getting Started

To initialize the SDK with your app properties, you can use the configuration structure. Reach out to us at
https://www.wellspent.so/demo for an API Key.

```swift
struct WellspentSDKProperties {
    init(
        apiKey: String,
        name: String,
        bundleId: String,
        appGroupIdentifier: String,
        activity: String? = nil,
        successCriteria: String? = nil
    )
}
```

## Setup App Extensions

WellspentSDK requires you to add 3 separate app extension targets to your project namely the `DeviceActivityMonitorExtension`, `ShieldActionExtension` and the `ShieldConfigurationExtension`.

1. **To Create a New Target**
   - Open Xcode.
   - Go to `File` > `New` > `Target`.
   - Search for extension.
   - Select it and click `Next`.
   - Set a suitable name.
   - Click `Finish`.

###  Add app groups capability

Add an app group with to all extension targets and your main app.

2. **Enable App Group**
   - Select the newly created extensions targets.
   - Go to `Signing & Capabilities`.
   - Click `+ Capability` and add `App Groups`.
   - Add `$(YOUR_APP_GROUP_NAME)`.
   - Repeat for all app extension targets and your main app
   - Add the AppGroup to Info.plist for all extension targets

   ```
   <dict>
	<key>AppGroup</key>
	<string>YOUR_APPGROUP_NAME</string>
    .....


   </dict>
   ```

> [!CAUTION]
> The Wellspent SDK will not work without the app extensions and their respective app groups
> capability being set up. The app group name must be added to the `Info.plist` for every extension target.

## Request FamilyControls Entitlement (for distribution)

With the app extensions created, request for the Family Controls Entitlement (Distribution) from Apple. This is neccessary to go live in the App Store but not a requirement for local development.
It's preferable to request the entitlement soon after creating the app extensions.

[Follow our guide to request the Family Controls (Distribution) Entitlement.](https://wellspentsdk.notion.site/Apple-Family-Controls-Entitlement-Guide-1f15776f1ea380d1b047cd01fb0568b1?pvs=4)

## Integrating the Swift SDK

### 1. Import

Ensure the SDK is added to your project via Swift Package Manager.

```swift
dependencies: [
    .package(url: "https://github.com/nlbb/Wellspent-iOS-SDK-Binaries.git", branch: "main")
]
```
#### Add frameworks to the extensions and app targets

1. Add the `Extension_DeviceActivityMonitor` framework to the `DeviceActivityMonitor` extension target
2. Add the `Extension_ShieldAction` framework to the `ShieldActionExtension` target
3. Add the `Extension_ShieldConfiguration` framework to the `ShieldConfigurationExtension` target
4. Add the `Wellspent` framework to the main app target

#### Modify the DeviceActivityMonitorExtension code.

   - Open `DeviceActivityMonitorExtension.swift` and replace all existing code with snippet below

  ```swift
    import Extension_DeviceActivityMonitor

    class YourDeviceMonitorExtension: Extension_DeviceActivityMonitor.ActivityMonitorExtension {}

  ```

#### Modify the ShieldActionExtension code.

   - Open `ShieldActionExtension.swift` and replace all existing code with snippet below

  ```swift
    import Extension_ShieldAction

    class ShieldActionExtension: Extension_ShieldAction.ShieldActionExtension {}
  ```


#### Modify the ShieldConfigurationExtension code.

   - Open `ShieldConfigurationExtension.swift` and replace all existing code with snippet below.

  ```swift
    import Extension_ShieldConfiguration

    class ShieldConfigurationExtension: Extension_ShieldConfiguration.ShieldConfigurationExtension {}

  ```

### 2. Initialize on Launch

Initialize it in your application's entry point with the provided API key and
further properties, using the initialize method.

Doing this as early as possible ensures the SDK is ready for use.
This method will fail synchronously if and only if the passed arguments are
invalid. This method does depend on network connectivity.

```swift
import WellspentSDK

Task {
    let result = await Wellspent.shared.configure(
        properties: WellspentSDKProperties(
            apiKey: "INSERT-YOUR-API-KEY",
            name: "APP_NAME",
            bundleId: "APP_BUNDLE_ID",
            appGroupIdentifier: "YOUR_APP_GROUP",
            activity: "APP_ACTIVITY e.g learning",
            successCriteria: "SUCCESS_CRITERIA e.g lesson"
        ),
        theme: CustomTheme(), // this is optional
        trackingHandler: AnalyticsManager() // this is optional
    )
}
```

#### Implement Tracking (Optional)

To track user interactions and screen view (e.g. toggling nudge settings),
implement the `WellspentTrackingService` protocol and pass an instance to `configure`.
This allows you to forward analytics events to your provider (e.g., Firebase, Mixpanel).

Create a class that conforms to `WellspentTrackingService` to receive analytics events.

```swift
class AnalyticsManager: WellspentTrackingService {
    func trackScreenView(screenName: String, properties: [String: Any]?) {
        // Forward screen view to your analytics provider
    }

    func trackEvent(eventName: String, properties: [String: Any]?) {
        // Forward event to your analytics provider
    }
}
```

### 3. User Onboarding

You can kick-off user onboarding by invoking the `start` method

Usually this call should be initiated in response some user action, such as
tapping a "Connect" button.

```swift
    Wellspent.shared.start(
        onComplete: {
            // Handle scenario when user completed onboarding flow
        },
        onDismiss: {
            // Handle scenario when user cancels the onboarding flow
        }
    )
```


### 4. Edit Nudge Settings

Users can customize their nudge schedules and manage distracting apps directly through the SDK. This can be achieved by invoking the `showNudgeSettings` method, which presents a user interface for modifying these preferences.

```swift
import WellspentSDK

Wellspent.shared.showNudgeSettings(onFinish: {
    // Handle scenario when user finishes editing nudge settings
})
```

This method allows users to adjust their nudge schedules to better align with their goals and specify which apps they find distracting. By enabling this feature, you empower users to take control of their app usage patterns, fostering a more intentional and productive experience.


### 5. Propagating goal completion

When users complete their daily habit, developers should call the `completeDailyHabit` method to unlock the apps.

This action notifies the Wellspent that the user's daily goal has been achieved, triggering updates to ensure the intervention mechanism reflects the completion status.

By doing so, users regain access to their apps, reinforcing positive behavior and habit formation.

```swift
import WellspentSDK

WellspentSDK.shared.completeDailyHabit()
```


## Customization

#### Theming

To customise the colors and typography of the SDK screens, you need to create a class that conforms to `WellspentTheme` protocol. This protocol exposes the cutomisable UI elements on the SDK.

You can initialize the Color values in various ways, including using hex values, named colors, RGB values etc.

```swift
struct CustomTheme: WellspentTheme {
   // Background colors
   var backgroundPrimary: Color { Color("BackgroundPrimary") }
   var backgroundSecondary: Color { Color("BackgroundSecondary") }
   var backgroundTertiary: Color { Color("BackgroundTertiary") }
   var backgroundShield: Color { Color("BackgroundShield") }

   // Label colors
   var labelPrimary: Color { Color("LabelPrimary") }
   var labelSecondary: Color { Color("LabelSecondary") }
   var labelTertiary: Color { Color("LabelTertiary") }

   // Button colors
   var backgroundButton: Color { Color("BackgroundButton") }
   var backgroundButtonDisabled: Color { Color("BackgroundButtonDisabled") }
   var buttonLabel: Color { Color("ButtonLabel") }
   var buttonLabelDisabled: Color { Color("ButtonLabelDisabled") }

   // Other colors
   var fills: Color { Color("Fills") }
}
```

#### Use Asset Catalog Colors for Light and Dark Mode
For automatic support of both light and dark mode, define your colors in an Asset Catalog:

1. Open your Asset Catalog (Assets.xcassets)
2. Create a new Color Set for each theme color (e.g., "BackgroundPrimary")
3. Select the Color Set and enable "Any, Dark" appearance in the Attributes Inspector
4. Set different colors for light and dark appearances

#### After implementing the custom theme, it can be applied in two ways:

During SDK configuration:

```swift
  let result = await Wellspent.shared.configure(
    properties: WellspentSDKProperties(
        // Your configuration properties
    ),
    theme: CustomTheme(),
    trackingHandler: trackingHandler
)
```

After configuration, using the apply method:

```swift
    Wellspent.shared.apply(theme: CustomTheme())
```
