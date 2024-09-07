<a href="https://www.nuget.org/packages/CorePush">
  <img src="https://buildstats.info/nuget/CorePush" alt="Nuget Package Details" />
</a>

[![NuGet Version](https://badge.fury.io/nu/CorePush.svg)](https://badge.fury.io/nu/CorePush)


# .NET Core Push Notifications for Web, Android and iOS
Send notifications to:
- ✅ **iOS** - Apple Push Notifications (via Latest Apple Push Notifications HTTP2 JWT API)
- ✅ **Android** - via Firebase Cloud Messaging (via Latest Firebase HTTP v1 API)
- ✅ **Web** - via Firebase Cloud Messaging (via Latest Firebase HTTP v1 API)

CorePush is a simple lightweight library with minimal overhead. Send notifications to Android and Web using Firebase Cloud Messaging and iOS APN with JWT HTTP/2 API.

# Installation - NuGet

Version 4.0.0+ requires .NET7.0. For earlier versions please use v3.1.1 of the library as it's targeting netstandard2.0, though please note, it uses legacy FCM send API. 
The easiest way to get started with CorePush is to use [nuget](https://www.nuget.org/packages/CorePush) package.

dotnet cli:
```
dotnet add package CorePush
```

Package Manager Console:
```
Install-Package CorePush
```

Check out Tester project [Program.cs](https://github.com/andrei-m-code/net-core-push-notifications/blob/master/CorePush.Tester/Program.cs) for a quick getting started.

# Firebase Cloud Messages for Android, iOS and Web

To start sending Firebase messages you need to have Google Project ID and JWT Bearer token. Steps to generate JWT bearer token:
1. Enable HTTP v1 API if you haven't done it yet. Go here for instructions: https://console.firebase.google.com/project/YOUR-GOOGLE-PROJECT-ID/settings/cloudmessaging/ Your project ID looks like this: my-project-123456.
2. From that page you can also go to "Manage Service Accounts". Here is the link: https://console.cloud.google.com/iam-admin/serviceaccounts and select your project.
3. Create Service Account with "Firebase Service Management Service Agent" role.
4. Download Service Account JSON file and use it to configure FirebaseSender either by deserializing it into FirebaseSettings or by directly passing json string into the constructor.

Sending messages is very simple so long as you know the format:

```csharp
var firebaseSettingsJson = await File.ReadAllTextAsync('./link/to/my-project-123345-e12345.json');
var fcm = new FirebaseSender(firebaseSettingsJson, httpClient);
await fcm.SendAsync(payload);
```
Useful links:
- Message formats: https://firebase.google.com/docs/cloud-messaging/concept-options#notifications
- Migrating from legacy API: https://firebase.google.com/docs/cloud-messaging/migrate-v1

## Firebase iOS notifications
If you want to use Firebase to send iOS notifications, please checkout this article: https://firebase.google.com/docs/cloud-messaging/ios/certs.
The library serializes notification object to JSON using Newtonsoft.Json library and sends it to Google cloud. Here is more details on the expected payloads for FCM https://firebase.google.com/docs/cloud-messaging/concept-options#notifications.

## Firebase Notification Payload Example

```json
{
  "message":{
     "token":"bk3RNwTe3H0:CI2k_HHwgIpoDKCIZvvD this is DEVICE_TOKEN",
     "notification":{
       "title":"Match update",
       "body":"Arsenal goal in added time, score is now 3-0"
     },
     "android":{
       "ttl":"86400s",
       "notification"{
         "click_action":"OPEN_ACTIVITY_1"
       }
     },
     "apns": {
       "headers": {
         "apns-priority": "5",
       },
       "payload": {
         "aps": {
           "category": "NEW_MESSAGE_CATEGORY"
         }
       }
     },
     "webpush":{
       "headers":{
         "TTL":"86400"
       }
     }
   }
 }
```

# Apple Push Notifications

To send notifications to Apple devices you have to create a publisher profile and pass settings object with necessary parameters to ApnSender constructor. Apn Sender will create and sign JWT token and attach it to every request to Apple servers:
1. P8 private key - p8 certificate generated in itunes. Just 1 line string without spaces, ----- or line breaks.
2. Private key id - 10 digit p8 certificate id. Usually a part of a downloadable certificate filename e.g. AuthKey_IDOFYOURCR.p8</param>
3. Team id - Apple 10 digit team id from itunes
4. App bundle identifier - App slug / bundle name e.g.com.mycompany.myapp
5. Server type - Development or Production APN server

```csharp
var apn = new ApnSender(settings, httpClient);
await apn.SendAsync(notification, deviceToken);
```
Please see Apple notification payload examples here: https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CreatingtheNotificationPayload.html#//apple_ref/doc/uid/TP40008194-CH10-SW1.
Tip: To send properties like {"content-available": true} you can use System.Text.Json attributes over C# properties like `[JsonPropertyName("content-available")]`.

## Example of notification payload
You can find expected notification formats for different types of notifications in the documentation. To make it easier to get started, here is a simple example of visible notification (the one that you'll see in phone's notification center) for iOS:

```csharp
public class AppleNotification
{
    public class ApsPayload
    {
        [JsonPropertyName("alert")]
        public string AlertBody { get; set; }
    }

    // Your custom properties as needed

    [JsonPropertyName("aps")]
    public ApsPayload Aps { get; set; }
}
```
Use `[JsonPropertyName("alert-type")]` attribute to serialize C# properties into JSON properties with dashes.

# Azure Functions and Azure App Service
You may be getting this error when running in Azure Functions or Azure App Service:
```
System.Security.Cryptography.CryptographicException: The system cannot find the file specified. at
System.Security.Cryptography.NCryptNative.ImportKey(SafeNCryptProviderHandle provider, Byte[] keyBlob, String format) at
System.Security.Cryptography.CngKey.Import(Byte[] keyBlob, String curveName, CngKeyBlobFormat format, CngProvider provider)
```
The solution is to add this in the Environment Variables of your service: `WEBSITE_LOAD_USER_PROFILE: 1`. More info on the issue can be found [here](https://stackoverflow.com/questions/66367406/cngkey-system-security-cryptography-cryptographicexception-the-system-cannot-fin) and [here](https://stackoverflow.com/questions/46114264/x509certificate2-on-azure-app-services-azure-websites-since-mid-2017).

# MIT License

Copyright (c) 2020 Andrei M

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

