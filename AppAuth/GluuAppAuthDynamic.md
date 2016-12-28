# Gluu AppAuth Dynamic Registration.

### Clone the project
https://github.com/openid/AppAuth-Android.git

### changes in xml file: 
app/res/values/idp_configs.xml

Do the folloing changes in app/res/values/idp_configs.xml file:
	- Change google_enabled value to true
 	- Change google_auth_redirect_uri to https://client.gluu.org/cb
the uri "https://client.gluu.org/cb" should be the desired redirect uri.

Add this two properties in the app/res/values/idp_configs_optional.xml file
```xml
<string name="gluu_discovery_uri" translatable="false">https://example.gluu.org/.well-known/openid-configuration</string>
<string name="dynamic_registration_endpoint" translatable="false">https://example.gluu.org/oxauth/seam/resource/restv1/oxauth/register</string>
```

---

Do the following changes in app/java/net.openid.appauthdemo/IdentityProvider.java

    - Change <R.string.google_discovery_uri> to <R.string.gluu_discovery_uri>
    - Change <NOT_SPECIFIED>, // dynamic registration not supported to <R.string.dynamic_registration_endpoint>
    - Change the <R.string.google_client_id> to <NOT_SPECIFID> for dynamic registration

---

We need to add header in library in request in the method RegistrationRequestTask in the file called AuthorizationService.java
Path to the file is /library/java/net/openid/appauth/AuthorizationService.java.

```java
conn.setRequestProperty("Content-Type", "application/json");
```
This line can be added on line number 453.

---

And also change the manifest file and change the activity tag to

```xml
<activity android:name="net.openid.appauth.RedirectUriReceiverActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="http" />
        <data android:scheme="https" />
        <data android:host="example.gluu.org" />
    </intent-filter>
</activity>
```

So that all the URIs that matches "example.gluu.org" are opened in the app itself
