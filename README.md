# <b>playPORTAL Android (Java) SDK</b></br>
playPORTAL<sup>TM</sup> provides a service to app developers for managing users of all ages and the data associated with the app and the app users, while providing compliance with required COPPA laws and guidelines. The playPORTAL service is easily accessed via our SDKs, this doc describes the Java SDK for use in Android apps.


## Getting Started
The playPORTAl service requires setting up your app in the playPORTAL.

* ### <b>Step 1:</b> Create playPORTAL Partner Account

	* Navigate to [playPORTAL Partner Dashboard](https://partner.iokids.net)
	* Click on <b>Sign Up For Developer Account</b>
	* After creating your account, email us at [info@playportal.io](mailto:info@playportal.io?subject=Developer%20Sandbox%20Access%20Request) to verify your account.
  </br>

* ### <b>Step 2:</b> Register your App with playPORTAL

	* After confirmation, log in to the [playPORTAL Partner Dashboard](https://partner.iokids.net)
	* In the left navigation bar click on the <b>Apps</b> tab.
	* In the <b>Apps</b> panel, click on the "+ Add App" button.
	* Add an icon, name & description for your app.
	* For "Environment" leave "Sandbox" selected.
	* Click "Add App"
  </br>

* ### <b>Step 3:</b> Generate your Client ID and Client Secret

	* Tap "Client IDs & Secrets"
	* Tap "Generate Client ID"
	* Copy these and save them to a secure place accessible by your app. Be careful not to share them or store them in public version control - they uniquely identify your app and grant the permissions to your app as defined in the [playPORTAL Partner Dashboard](https://partner.iokids.net).
  </br>

* ### <b>Step 4:</b> Add your Redirect URI

	* Add a [Custom URL Scheme for your app](https://developer.apple.com/documentation/uikit/core_app/communicating_with_other_apps_using_custom_urls?language=objc)
	* From the [playPORTAL Partner Dashboard](https://partner.iokids.net) navigate to your app and tap <b>Registered Redirect URIs</b>
	* Enter the your Custom URL Scheme
  </br>

* ### <b>Step 5:</b> Install the SDK
	* Add dependency com.dynepic.ppsdk_android-0.0.1 to build.gradle

---
## Configure
* Be sure to import the playPORTAL SDK into your "main" activity. 
	```
	import com.dynepic.ppsdk_android.*;
	```
* In your resource folder /res change the App name string to appropriate name for your app
* Copy your clientID, clientSecret & redirectURI from the playPORTAL website into your app
* In your app init, call the SDK configure (e.g. from your apps MainActivity.m):
	```
    public class MainActivity extends AppCompatActivity {

        // The next 3 strings are taken from your "app" definition in the playPORTAL web.
        String id = "iok-cid-444ad028be7e16c1157962992ccd35e33823d6728a33d8fa";
        String sec = "iok-cse-93714e707231284f4d30de950c5d6580434c0b7ae6e7d7f5";
        String uri = "yourapp://redirect";
	      
        PPManager ppsdk;
        ppsdk = PPManager.getInstance(); // playPORTAL is managed as a singleton class
        ppsdk.configure(id, sec, uri, getApplicationContext());

        // Run the playPORTAL SDK user auth check. If user isn't logged in, the SDK will present SSO login
        // and return from that will be to MainActivity
        if(!ppsdk.isAuthenticated()) {
			Intent myIntent = new Intent(this, LoginActivity.class);
            MainActivity.this.startActivity(myIntent);
        }


        // continue with your app's java code here
        
	}
	```

---
## Storage
* When a user logs in to your app, the SDK creates (or opens) a PRIVATE storage bucket just for them. This is secure storage for storing app-specific information for that user.
Data is returned (on no Error) via a lambda function. See Storage details for more info on reading/writing to storage.
```
	ppsdk.readBucket(ppsdk.getPrivateDataStorage(), "TestData", (String readKey, String readValue, String readData, String readError) -> {
		if (readError == null) {
			Log.d("Testdata read key:", readKey + " value:" + readValue + " data:" + readData);
		} else {
    		Log.d("private readBucket Error:", readError);
		}
	});
```

* The SDK also creates a global PUBLIC storage bucket that all users can write to and read from.
```
	ppsdk.readBucket(ppsdk.getPublicDataStorage(), "TestData", (String readKey, String readValue, String readData, String readError) -> {
		if (readError == null) {
			Log.d("Testdata read key:", readKey + " value:" + readValue + " data:" + readData);
		} else {
    		Log.d("global readBucket Error:", readError);
		}

	});
```
* Call the following method to write to the user's private storage bucket:
```
    ppsdk.writeBucket(ppsdk.getPrivateDataStorage(), "TestData", gson.toJson(td), false, (String writeKey, String writeValue, String writeData, String writeError) -> {
		if (writeError != null) {
			Log.e("Error writing data to global bucket:", writeError);
		}
	});
```

* Call the following method to write to the global public storage bucket:
```
	ppsdk.writeBucket(ppsdk.getPublicDataStorage(), "TestData", gson.toJson(td), false, (String writeKey, String writeValue, String writeData, String writeError) -> {
    		if (writeError != null) {
    			Log.e("Error writing data to global bucket:", writeError);
    		}
    	});
```
---

### Storage Details
Information stored in playPORTAL storage is opaque to the playPORTAL system. This has multiple implications:
1. The SDK user is responsible for defining the JSON of the stored elements
2. The playPORTAL system doesn't perform any conversions on the presented JSON elements, so the user's Java objects that are sent/received to/from the playPORTAL SDK must be passed through a gson (or Jackson) parser, i.e. can be converted to JSON. This can be done (using gson as shown gson.toJson(javaObject)). Failures will result in exceptions or data integrity issues.
3. Data objects can be stored retrieved at any granularity suitable to the user's application. Specifically, an individual element (e.g. KV pair, <string, string>) can be stored/retrieved. Similarly, more complex Java objects can be stored/retrieved. In order to facilitate data operations, we recommend using a converter, such as http://www.jsonschema2pojo.org/ to simplify the process of defining the marshalling/unmarshalling methods. 
---

## Profile
The user profile is available either via direct access, or by providing a callback.

##### Profile via direct access
```
	PPUserObject u = getProfile();
	if(u != null) Log.d("username:", u.getValueForKey("username"));	
```

##### Profile via callback 
To use a callback, the userListener method (or lambda) must be registered. The example shows using a lambda function:
```
	ppsdk.addUserListener((PPUserObject u) -> {
		Log.d("userListener invoked for user:", u.getValueForKey("username"));
	});
```
## Friends
* A user's friends can be retrieved using the following method:
```
    ppsdk.getFriendsProfiles(
```
