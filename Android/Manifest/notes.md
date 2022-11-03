# App Manifest

* Documentation: https://developer.android.com/guide/topics/manifest/manifest-intro
* Manifest Merger. How android manifest is formed: https://www.youtube.com/watch?v=rj-oHWG6YKI

* Every app project must have an `AndroidManifest.xml` file (with precisely that name) at the root of the **project source set**. The manifest file describes essential information about your app to the Android build tools, the Android operating system, and Google Play.

* Among many other things, the manifest file is required to declare the following:
    * The components of the app, which include all activities, services, broadcast receivers, and content providers. Each component must define basic properties such as the name of its Kotlin or Java class. It can also declare capabilities such as which device configurations it can handle, and intent filters that describe how the component can be started.
    * The permissions that the app needs in order to access protected parts of the system or other apps. It also declares any permissions that other apps must have if they want to access content from this app.
    * The hardware and software features the app requires, which affects which devices can install the app from Google Play.

---

## File features

### App components

* For each **app component** that you create in your app, you must declare a corresponding XML element in the manifest file:
    * `<activity>` for each subclass of `Activity`.
    * `<service>` for each subclass of `Service`.
    * `<receiver>` for each subclass of `BroadcastReceiver`.
    * `<provider>` for each subclass of `ContentProvider`.

* If you subclass any of these components without declaring it in the manifest file, the system cannot start it.

* The name of your subclass must be specified with the `name` attribute, using the full package designation.

```
<manifest ... >
    <application ... >
        <activity android:name="com.example.myapp.MainActivity" ... >
        </activity>
    </application>
</manifest>
```

* However, if the first character in the `name` value is a period, the app's namespace (from the module-level `build.gradle` file's `namespace` property) is prefixed to the name.

### Intent filters

* App activities, services, and broadcast receivers are activated by **intents**. An intent is a message defined by an `Intent` object that describes an action to perform.

* When an app issues an intent to the system, the system locates an app component that can handle the intent based on intent filter declarations in each app's manifest file. The system launches an instance of the matching component and passes the `Intent` object to that component. If more than one app can handle the intent, then the user can select which app to use.

* An app component can have any number of intent filters (defined with the `<intent-filter>` element), each one describing a different capability of that component.

### Icons and labels

* A number of manifest elements have `icon` and `label` attributes for displaying a small icon and a text label, respectively, to users for the corresponding app component.

* In every case, the icon and label that are set in a parent element become the default `icon` and `label` value for all child elements. For example, the icon and label that are set in the `<application>` element are the default icon and label for each of the app's components.

### Permissions

* Android apps must request permission to access sensitive user data (such as contacts and SMS) or certain system features (such as the camera and internet access). Each permission is identified by a unique label

```
<manifest ... >
  <uses-permission android:name="android.permission.SEND_SMS"/>
  ...
</manifest>
```

* Your app can also protect its own components with permissions. It can use any of the permissions that are defined by Android, as listed in `android.Manifest.permission`, or a permission that's declared in another app. Your app can also define its own permissions. A new permission is declared with the `<permission>` element.

### Device compatibility

* The manifest file is also where you can declare what types of hardware or software features your app requires, and thus, which types of devices your app is compatible with. Google Play Store **does not allow** your app to be installed on devices that don't provide the features or system version that your app requires.

#### <uses-feature>

* The `<uses-feature>` element allows you to declare hardware and software features your app needs. For example, if your app cannot achieve basic functionality on a device without a compass sensor, you can declare the compass sensor as required with the following manifest tag:

```
<manifest ... >
  <uses-feature android:name="android.hardware.sensor.compass"
        android:required="true" />
  ...
</manifest>
```

#### <uses-sdk>

* Each successive platform version often adds new APIs not available in the previous version. To indicate the minimum version with which your app is compatible, your manifest must include the `<uses-sdk>` tag and its `minSdkVersion` attribute.

* However, beware that attributes in the `<uses-sdk>` element are overridden by corresponding properties in the `build.gradle` file. So if you're using Android Studio, you must specify the `minSdkVersion` and `targetSdkVersion` values there instead:

```
android {
    defaultConfig {
        applicationId 'com.example.myapp'

        // Defines the minimum API level required to run the app.
        minSdkVersion 15

        // Specifies the API level used to test the app.
        targetSdkVersion 28

        ...
    }
}
```

---

## File conventions

* This section describes the conventions and rules that generally apply to all elements and attributes in the manifest file.

#### Elements

* Only the `<manifest>` and `<application>` elements are required. They each must occur only once. Most of the other elements can occur zero or more times. However, some of them must be present to make the manifest file useful.

* All of the values are set through attributes, not as character data within an element.

* Elements at the same level are generally not ordered. For example, the `<activity>`, `<provider>`, and `<service>` elements can be placed in any order. There are two key exceptions to this rule:
    * An `<activity-alias>` element must follow the `<activity>` for which it is an alias.
    * The `<application>` element must be the last element inside the `<manifest>` element.

#### Attributes

* Technically, all attributes are optional. However, many attributes must be specified so that an element can accomplish its purpose. For truly optional attributes, the reference documentation indicates the default values.

* Except for some attributes of the root `<manifest>` element, all attribute names begin with an `android:` prefix.

#### Multiple values

* If more than one value can be specified, the element is almost always repeated, rather than multiple values being listed within a single element.

#### Resource values

* Some attributes have values that are displayed to users, such as the title for an activity or your app icon. The value for these attributes might differ based on the user's language or other device configurations (such as to provide a different icon size based on the device's pixel density), so the values should be set from a resource or theme, instead of hard-coded into the manifest file.

#### String values

* Where an attribute value is a string, you must use double backslashes (`\\`) to escape characters, such as `\\n` for a newline or `\\uxxxx` for a Unicode character.

---

## Manifest elements reference


| tag                        | Functionality                                                                                                                             |
|----------------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
| `<action>`                 | Adds an action to an intent filter.                                                                                                       |
| `<activity>`               | Declares an activity component.                                                                                                           |
| `<activity-alias>`         | Declares an alias for an activity.                                                                                                        |
| `<application>`            | The declaration of the application.                                                                                                       |
| `<category>`               | Adds a category name to an intent filter.                                                                                                 |
| `<compatible-screens>`     | Specifies each screen configuration with which the application is compatible.                                                             |
| `<data>`                   | Adds a data specification to an intent filter.                                                                                            |
| `<grant-uri-permission>`   | Specifies the subsets of app data that the parent content provider has permission to access.                                              |
| `<instrumentation>`        | Declares an Instrumentation class that enables you to monitor an application's interaction with the system.                               |
| `<intent-filter>`          | Specifies the types of intents that an activity, service, or broadcast receiver can respond to.                                           |
| `<manifest>`               | The root element of the AndroidManifest.xml file.                                                                                         |
| `<meta-data>`              | A name-value pair for an item of additional, arbitrary data that can be supplied to the parent component.                                 |
| `<path-permission>`        | Defines the path and required permissions for a specific subset of data within a content provider.                                        |
| `<permission>`             | Declares a security permission that can be used to limit access to specific components or features of this or other applications.         |
| `<permission-group>`       | Declares a name for a logical grouping of related permissions.                                                                            |
| `<permission-tree>`        | Declares the base name for a tree of permissions.                                                                                         |
| `<provider>`               | Declares a content provider component.                                                                                                    |
| `<queries>`                | Declares the set of other apps that your app intends to access. Learn more in the guide about package visibility filtering.               |
| `<receiver>`               | Declares a broadcast receiver component.                                                                                                  |
| `<service>`                | Declares a service component.                                                                                                             |
| `<supports-gl-texture>`    | Declares a single GL texture compression format that the app supports.                                                                    |
| `<supports-screens>`       | Declares the screen sizes your app supports and enables screen compatibility mode for screens larger than what your app supports.         |
| `<uses-configuration>`     | Indicates specific input features the application requires.                                                                               |
| `<uses-feature>`           | Declares a single hardware or software feature that is used by the application.                                                           |
| `<uses-library>`           | Specifies a shared library that the application must be linked against.                                                                   |
| `<uses-native-library>`    | Specifies a vendor-provided native shared library that the app must be linked against.                                                    |
| `<uses-permission>`        | Specifies a system permission that the user must grant in order for the app to operate correctly.                                         |
| `<uses-permission-sdk-23>` | Specifies that an app wants a particular permission, but only if the app is installed on a device running Android 6.0 (API level 23) or higher. |
| `<uses-sdk>`               | Lets you express an application's compatibility with one or more versions of the Android platform, by means of an API level integer.      |
