![Data_security](https://user-images.githubusercontent.com/65482596/169642697-c779767e-67ba-4fb1-80c0-a5b6cb1f5411.jpg)

# how-to-better-secure-your-mobile-application

# Hi there! <img width="30" src="https://camo.githubusercontent.com/e8e7b06ecf583bc040eb60e44eb5b8e0ecc5421320a92929ce21522dbc34c891/68747470733a2f2f6d656469612e67697068792e636f6d2f6d656469612f6876524a434c467a6361737252346961377a2f67697068792e676966">

# This is a guide for securing mobile application against most of threats that still continue to troble application's security.

## Prepared and maintained by [Nidhi Yashwanth](https://github.com/nidhiyashwanth).

## Will keep making changes as the requirements arise ❗

## Contents - How to better secure your app?

* [Introduction Security](#introduction-security)
* [Encrypt your data](#encrypt-your-data)
* [Detect insecure devices](#detect-insecure-devices)
* [Authenticate users and keys with biometrics](#authenticate-users-and-keys-with-biometrics)
* [Communicate securely](#communicate-securely)
* [Address issues found by Google Play](#address-issues-found-by-google-play)
* [Be the first to know](#be-the-first-to-know)
* [Test test and test again](#test-test-and-test-again)
* [Audit third-party libraries](#audit-third-party-libraries)

### Introduction Security

-The goal is to make Android the safest mobile platform in the world. That's why we consistently invest in technologies that bolster the security of the platform, its apps, and the global Android ecosystem.
-I'm delighted to share with you my findings on how to keep users safe and secure.

### Encrypt your data

- The Security library provides an implementation of the [security best practices](https://developer.android.com/topic/security/best-practices) related to reading and writing data at rest, as well as key creation and verification.
- The library uses the builder pattern to provide safe default settings for the following security levels:
    - Strong security that balances great encryption and good performance. This level of security is appropriate for consumer apps, such as banking and chat apps, as well as enterprise apps that perform certificate revocation checking.
    - Maximum security. This level of security is appropriate for apps that require a hardware-backed keystore and user presence for providing key access.
    - **Key management**
    	- A **keyset** that contains one or more keys to encrypt a file or shared preferences data. The keyset itself is stored in `SharedPreferences`. 
    	- A **primary (master) key** that encrypts all keysets. This key is stored using the Android keystore system.
    - **Classes included in library**
    	- **EncryptedFile:** Provides custom implementations of `FileInputStream` and `FileOutputStream`, granting your app more secure streaming read and write operations.
    	- **EncryptedSharedPreferences:** Wraps the SharedPreferences class and automatically encrypts keys and values using a two-scheme method:
    		- **Keys** are encrypted using a deterministic encryption algorithm such that the key can be encrypted and properly looked up.
    		- **Values** are encrypted using AES-256 GCM and are non-deterministic.
    
    
	- **The following sections show how to use these classes to perform common operations with files and shared preferences.**
		- To use the Security library, add the following dependency to your app module's build.gradle file:
			
```
			dependencies {
			implementation "androidx.security:security-crypto:1.0.0-rc04"

			// For Identity Credential APIs
			implementation "androidx.security:security-identity-credential:1.0.0-alpha02"
			}
```

Read Files The following code snippet demonstrates how to use EncryptedFile to read the contents of a file in a more secure way:

```kotlin

	// Although you can define your own key generation parameter specification, it's
	// recommended that you use the value specified here.
	val mainKey = MasterKey.Builder(applicationContext)
	.setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
	.build()
	val fileToRead = "my_sensitive_data.txt"
	val encryptedFile = EncryptedFile.Builder(
	applicationContext,
	File(DIRECTORY, fileToRead),
	mainKey,
	EncryptedFile.FileEncryptionScheme.AES256_GCM_HKDF_4KB
	).build()

	val inputStream = encryptedFile.openFileInput()
	val byteArrayOutputStream = ByteArrayOutputStream()
	var nextByte: Int = inputStream.read()
	while (nextByte != -1) {
	byteArrayOutputStream.write(nextByte)
	nextByte = inputStream.read()
	}

	val plaintext: ByteArray = byteArrayOutputStream.toByteArray()

```

Write files The following code snippet demonstrates how to use `EncryptedFile` to write the contents of a file in a more secure way:

```kotlin

	// Although you can define your own key generation parameter specification, it's
	// recommended that you use the value specified here.
	val mainKey = MasterKey.Builder(applicationContext)
	.setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
	.build()

	// Create a file with this name, or replace an entire existing file
	// that has the same name. Note that you cannot append to an existing file,
	// and the file name cannot contain path separators.
	val fileToWrite = "my_sensitive_data.txt"
	val encryptedFile = EncryptedFile.Builder(
	applicationContext,
	File(DIRECTORY, fileToWrite),
	mainKey,
	EncryptedFile.FileEncryptionScheme.AES256_GCM_HKDF_4KB
	).build()

	val fileContent = "MY SUPER-SECRET INFORMATION"
	.toByteArray(StandardCharsets.UTF_8)
	encryptedFile.openFileOutput().apply {
	write(fileContent)
	flush()
	close()
	}

```

**For use cases requiring additional security, complete the following steps:**

1. Create a `KeyGenParameterSpec.Builder` object, passing `true` into [setUserAuthenticationRequired()](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder#setUserAuthenticationRequired(boolean)) and a value greater than 0 into [setUserAuthenticationValidityDurationSeconds().](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder#setUserAuthenticationValidityDurationSeconds(int))
2. Prompt the user to enter credentials using createConfirmDeviceCredentialIntent(). Learn more about how to request [user authentication for key use.](https://developer.android.com/training/articles/keystore#UserAuthentication)
3. Override `onActivityResult()` to get the confirmed credential callback.

For more information, see [Requiring user authentication for key use.](https://developer.android.com/training/articles/keystore#UserAuthentication)


### Detect insecure devices

- Rooted or unlocked devices, or emulators may fail to protect user data and expose your app to attack. Use SafetyNet Attestation to determine if a device running your app has been tampered with. Based on the results from SafetyNet Attestation, consider acting to protect your app’s content.
- The SafetyNet Attestation API is an anti-abuse API that allows app developers to assess the Android device their app is running on. The API should be used as a part of your abuse detection system to help determine whether your servers are interacting with your genuine app running on a genuine Android device.
- The SafetyNet Attestation API provides a cryptographically-signed attestation, assessing the device's integrity. In order to create the attestation, the API examines the device's software and hardware environment, looking for integrity issues, and comparing it with the reference data for approved Android devices. The generated attestation is bound to the nonce that the caller app provides. The attestation also contains a generation timestamp and metadata about the requesting app.


### Authenticate users and keys with biometrics
- Use the [Biometric APIs](https://developer.android.com/training/sign-in/biometric-auth), part of the Jetpack BiometricPrompt Library, to take advantage of a device’s biometric sensors when authenticating users in your app.


### Communicate securely
- HTTPS and SSL provide secure protocols for transferring data between your app and servers. However, there are a number of common mistakes that developers make that can lead to insecure data transfer. Check that you’re not making any of these in your app.
- [Security with HTTPS and SSL](https://developer.android.com/training/articles/security-ssl)
- The Secure Sockets Layer (SSL)—now technically known as [Transport Layer Security](https://en.wikipedia.org/wiki/Transport_Layer_Security) (TLS)—is a common building block for encrypted communications between clients and servers. It's possible that an application might use SSL incorrectly such that malicious entities may be able to intercept an app's data over the network. To help you ensure that this does not happen to your app, this article highlights the common pitfalls when using secure network protocols and addresses some larger concerns about using Public-Key Infrastructure (PKI).
- **Concepts**
	- In a typical SSL usage scenario, a server is configured with a certificate containing a public key as well as a matching private key. As part of the handshake between an SSL client and server, the server proves it has the private key by signing its certificate with [public-key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography).
	- However, anyone can generate their own certificate and private key, so a simple handshake doesn't prove anything about the server other than that the server knows the private key that matches the public key of the certificate. One way to solve this problem is to have the client have a set of one or more certificates it trusts. If the certificate is not in the set, the server is not to be trusted.
	- There are several downsides to this simple approach. Servers should be able to upgrade to stronger keys over time ("key rotation"), which replaces the public key in the certificate with a new one. Unfortunately, now the client app has to be updated due to what is essentially a server configuration change. This is especially problematic if the server is not under the app developer's control, for example if it is a third party web service. This approach also has issues if the app has to talk to arbitrary servers such as a web browser or email app.
	- In order to address these downsides, servers are typically configured with certificates from well known issuers called [Certificate Authorities (CAs)](https://en.wikipedia.org/wiki/Certificate_authority). The host platform generally contains a list of well known CAs that it trusts. As of Android 8.0 (Nougat), Android currently contains over 100 CAs that are updated in each release and do not change from device to device. Similar to a server, a CA has a certificate and a private key. When issuing a certificate for a server, the CA signs the server certificate using its private key. The client can then verify that the server has a certificate issued by a CA known to the platform.
	- [Learn More...](https://developer.android.com/training/articles/security-ssl)


### Address issues found by Google Play
- The App Security Improvement program is a service that helps detect known security vulnerabilities in your app. This service automatically scans your app as it’s submitted to Google Play. If any vulnerabilities are discovered, you get alerts by email and in the Google Play Console, with links to details about how to improve your app.
- [App security improvement program](https://developer.android.com/google/play/asi)
- The App Security Improvement program is a service provided to Google Play app developers to improve the security of their apps. The program provides tips and recommendations for building more secure apps and identifies potential security enhancements when your apps are uploaded to Google Play. To date, the program has facilitated developers to fix over 1,000,000 apps on Google Play.
- **How it works**
	- Before any app is accepted into Google Play, we scan it for safety and security, including potential security issues. We also continuously re-scan the over one million apps on Google Play for additional threats.
	- If your app is flagged for a potential security issue, we'll notify you immediately to help you quickly address the issue and help keep your users safe. We’ll deliver alerts to you using both email and the Google Play Console, with links to a support page with details about how to improve the app.
	- Typically, these notifications will include a timeline for delivering the improvement to users as quickly as possible. For some kinds of issues, we may require you to make security improvements in the app before you can publish any more updates to it.
	- You can confirm that you’ve fully addressed the issue by uploading the new version of your app to the Google Play Console. Be sure to increment the version number of the fixed app. After a few hours, check the Play Console for the security alert; if it’s no longer there, you’re all set.
	- - **Example of a security improvement alert for an app in the Play Console.**
		![google_play_security_notification](https://user-images.githubusercontent.com/65482596/169651062-d580622a-a914-4fb7-8fdb-3d8829a131fb.png)
	- [Learn More...](https://developer.android.com/google/play/asi)


### Be the first to know
- You cannot eliminate the possibility of there being undetected vulnerabilities in your app. Security researchers commonly assess new and updated apps for security issues. By setting up a vulnerability disclosure program (VDP) you provide guidelines for these experts to disclose vulnerabilities to you.
	- [Learn More...](https://developers.google.com/android/play-protect/starting-a-vdp)


### Test test and test again

- [Fundamentals of Testing](https://developer.android.com/security)
- **Organize your code for testing**
	- As your app expands, you might find it necessary to fetch data from a server, interact with the device's sensors, access local storage, or render complex user interfaces. The versatility of your app demands a comprehensive testing strategy.
	- Create and test code iteratively
		- When developing a feature iteratively, you start by either writing a new test or by adding cases and assertions to an existing unit test. The test fails at first because the feature isn't implemented yet.
		- It's important to consider the units of responsibility that emerge as you design the new feature. For each unit, you write a corresponding unit test. Your unit tests should nearly exhaust all possible interactions with the unit, including standard interactions, invalid inputs, and cases where resources aren't available. Take advantage of [Jetpack libraries](https://developer.android.com/jetpack) whenever possible; when you use these well-tested libraries, you can focus on validating behavior that's specific to your app.
		- The two cycles associated with iterative, test-driven
		- ![](https://developer.android.com/images/training/testing/testing-workflow.png)
		- The full workflow, as shown in Figure 1, contains a series of nested, iterative cycles where a long, slow, UI-driven cycle tests the integration of code units. You test the units themselves using shorter, faster development cycles. This set of cycles continues until your app satisfies every use case.
		- **View your app as a series of modules**
		- To make your code easier to test, develop your code in terms of modules, where each module represents a specific task that users complete within your app. This perspective contrasts the stack-based view of an app that typically contains layers representing the UI, business logic, and data.
		- It's important to set well-defined boundaries around each module, and to create new modules as your app grows in scale and complexity. Each module should have only one area of focus, and the APIs that allow for inter-module communication should be consistent. To make it easier and quicker to test these inter-module interactions, consider creating fake implementations of your modules. In your tests, the real implementation of one module can call the fake implementation of the other module.
		- - To learn more about how to define modules in your app, as well as platform support for creating and publishing modules, see Android App Bundles.
		- [Learn More...](https://developer.android.com/training/testing/fundamentals)


### Audit third-party libraries
- Your app may rely on third-party libraries for common use cases. However, third-party libraries can be a source of data leakage, especially those using external services, such as those for marketing and analytics.
- Audit your third-party libraries to check that you are using the original code from its open source project. Also, check to see whether any libraries are unnecessary. Then remove any libraries you don’t need or where you cannot be sure of the source.
- This step is important because third-party libraries can cause your app to be flagged as potentially harmful per the Malware or Mobile Unwanted Software policies.
- [Potentially Harmful Applications (PHAs)](https://developers.google.com/android/play-protect/potentially-harmful-applications)
	- Potentially Harmful Applications (PHAs) are apps that could put users, user data, or devices at risk. These apps are often generically referred to as malware. We've developed a range of categories for different types of PHAs, including trojans, phishing, and spyware apps, and we are continuously updating and adding new categories.
	- **Potentially harmful?**
		- There is some confusion around the ambiguity of the word potentially when used to describe malicious apps. Google Play Protect removes apps that have been flagged as Potentially Harmful because the app does contain malicious behavior not because we are simply unsure if the app is harmful or not. The word potentially is used here because malicious apps function differently depending on a variety of variables thus an app that is harmful to one Android device might not pose a risk at all to another Android device. For example, a device running the latest version of Android is not affected by harmful apps which use deprecated APIs to perform malicious behavior but a device that is still running a very early version of Android might be at risk. Mobile billing fraud poses a risk to devices connected to service carriers but devices which only connect to WIFI are not affected by these apps.
		- Apps are flagged as a PHA if they clearly pose a risk to some or all Android devices and users.
	- **User-wanted PHAs**
		- Some apps that can weaken or disable Android security features aren't categorized as PHAs. These apps provide functionality that users want, such as rooting the device and other development features. Even though these apps are potentially harmful, users install them intentionally, so Google Play Protect manages them differently than other PHAs.
		- When a user begins to installI an app that's classified as user-wanted, Google Play Protect warns the user of the app's potential hazards just once. The user can decide whether to continue with the installation. After installation, the user-wanted classifications prevents Google Play Protect from sending additional warnings, so there's no disruption to the user experience.
	- **Classifications**
		- There are several categories for classifying PHAs that help Play Protect detect them and determine the right action to take. These categories include malicious apps like trojans, spyware, and phishing apps, as well as user-wanted apps. If Play Protect detects a PHA, it displays a warning. For certain malicious apps, Play Protect automatically disables or removes the app. When Play Protect detects that a PHA contains features from multiple categories, it classifies the app based on the most harmful characteristics. For example, if an app applies to both ransomware and spyware categories, the Verify Apps message identifies it as ransomware.
		- You can view the current PHA categories and definitions [here.](https://developers.google.com/android/play-protect/phacategories)


# Hash-code-generation-Singing-apk

### Introduction
- During a typical development cycle, you test an app using `flutter run` at the command line,or by using the **Run** and **Debug** options in your IDE. By default, Flutter builds a _debug_ version of your app.
- When you're ready to prepare a _release_ version of your app, for example to [publish to the Google Play Store][play], this page can help. Before publishing, you might want to put some finishing touches on your app.


## Signing the app

### Methods to Sign an App

1)By using Keytool + android Studio

2)All by Android Studio itself. (New method)

 - Method 1:By using Keytool + android Studio

### Create an upload keystore

If you have an existing keystore, skip to the next step.
If not, create one by either:
* Following the [Android Studio key generation steps]({{site.android-dev}}/studio/publish/app-signing#sign-apk) 
* Running the following at the command line:

    On Mac/Linux, use the following command:

    ```terminal
    keytool -genkey -v -keystore ~/upload-keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload
    ```

    On Windows, use the following command:

    ```terminal
    keytool -genkey -v -keystore c:\Users\USER_NAME\upload-keystore.jks -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 -alias upload
    ```

    This command stores the `upload-keystore.jks` file in your home
    directory. If you want to store it elsewhere, change
    the argument you pass to the `-keystore` parameter.
    **However, keep the `keystore` file private;
    don't check it into public source control!**
    - [Learn More...](https://docs.flutter.dev/deployment/android#signing-the-app)

### Reference the keystore from the app

Create a file named `[project]/android/key.properties`
that contains a reference to your keystore:

```
storePassword=<password from previous step>
keyPassword=<password from previous step>
keyAlias=upload
storeFile=<location of the key store file, such as /Users/<user name>/upload-keystore.jks>
```

### Configure signing in gradle

- Configure gradle to use your upload key when building your app in release mode 
by editing the `[project]/android/app/build.gradle` file.
- [for more...](https://docs.flutter.dev/deployment/android#configure-signing-in-gradle)


####  How will it protects google people and developer ?

- KeyStore has Private key ,Public Key and many other data.
- After signing an app ,and placing in playstore ,Playstore has public key of that app.
- So when next any body want to update the app in playstore ,one need to sign in the app with it private key.
- And playstore can check it authenticity by using available public key of that app .
- you cant update application without same keystore value else app show different signature. for example you installed an app from playstore after some days if an update come for application then you will only be able to update previous app if its signature and new app signature is same.
- So keep your KeyStore save ,other wise you can't able to update your app in playstore.And i am sure you know the consequence of this.
-You will surprise to know that ,when ever you debug the app it singed with KeyStore evey time.
- Now i know ,What you will Ask ,i have't done this any time, So this is automatically done by android sdk.
- So
    - There are two build modes in android
    - `debug mode`: when you are developing and testing your application.
    - `release mode` :when you want to build a release version of your application that you can distribute directly to users or publish on an application marketplace such as Google Play.
    -The Android build process signs your application differently depending on which build mode you use to build your application.

    - When you build in debug mode the Android SDK build tools use the Keytool utility (included in the JDK) to create a debug key. Because the SDK build tools created the debug key, they know the debug key's alias and password. Each time you compile your application in debug mode, the build tools use the debug key along with the Jarsigner utility (also included in the JDK) to sign your application's .apk file. Because the alias and password are known to the SDK build tools, the tools don't need to prompt you for the debug key's alias and password each time you compile.

    - When you build in release mode you use your own private key to sign your application. If you don't have a private key, you can use the Keytool utility to create one for you. When you compile your application in release mode, the build tools use your private key along with the Jarsigner utility to sign your application's .apk file. Because the certificate and private key you use are your own, you must provide the password for the keystore and key alias.

    - The debug signing process happens automatically when you run or debug your application using Eclipse with the ADT plugin. Debug signing also happens automatically when you use the Ant build script with the debug option. You can automate the release signing process by using the Eclipse Export Wizard or by modifying the Ant build script and building with the release option.

- You May some found deug.keystore or release.keystore ,these are keystore files
- `debug.keystore` file is merely for developing and testing purposes, so using that you can't release your app to Google Play using that only.
- Caution: You cannot release your application to the public when signed with the debug certificate.
- `release.keystore` file is required only when you want to release your app to Google Play.

### Get Key Fingerprints

- To hook your app up with services like Google APIs you'll need to print out each of your keys' fingerprints and give them to the services you're using. To do that, use:
    
    ```$ keytool -list -v -keystore [keystore path] -alias [alias-name] -storepass [storepass] -keypass [keypass]```
    [learn more...](https://flutteragency.com/how-to-generate-sha-1-in-flutter/)


### Other Security Stuffs

- SSL

    - SSL (Secure Sockets Layer) is the standard security technology for establishing an encrypted link between a web server and a browser. This link ensures that all data passed between the web server and browsers remain private and integral.

- HTPPS=HTTP+SSL

    - https://www.youtube.com/watch?v=JCvPnwpWVUQ

    - https://www.youtube.com/watch?v=SJJmoDZ3il8


- SHA

    - In cryptography, SHA-1 (Secure Hash Algorithm 1) is a cryptographic hash function designed by the United States National Security Agency and is a U.S. Federal Information Processing Standard published by the United States NIST. SHA-1 produces a 160-bit (20-byte) hash value known as a message digest. A SHA-1 hash value is typically rendered as a hexadecimal number, 40 digits long.

    - SHA-1 is no longer considered secure against well-funded opponents. In 2005, cryptanalysts found attacks on SHA-1 suggesting that the algorithm might not be secure enough for ongoing use, and since 2010 many organizations have recommended its replacement by SHA-2 or SHA-3. Microsoft, Google and Mozilla have all announced that their respective browsers will stop accepting SHA-1 SSL certificates by 2017.


- Digest

    - Digest access authentication is one of the methods a web server can use to negotiate credentials, such as username or password, with a user's web browser. This can be used to confirm the identity of a user before sending sensitive information, such as online banking transaction history. It applies a hash function to the username and password before sending them over the network.

    - Technically, digest authentication is an application of MD5 cryptographic hashing with usage of nonce values to prevent replay attacks. It uses the HTTP protocol.


- MD5

    - MD5 is an algorithm that is used to verify data integrity through the creation of a 128-bit message digest from data input (which may be a message of any length) that is claimed to be as unique to that specific data as a fingerprint is to the specific individual.

    


You can find me on [![Twitter][1.2]][1].
<!-- Icons -->
[1.2]: http://i.imgur.com/wWzX9uB.png (twitter icon without padding)
[1]: https://twitter.com/NidhiYashwanth/
