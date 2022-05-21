<p align="center">
![Data_security](https://user-images.githubusercontent.com/65482596/169642697-c779767e-67ba-4fb1-80c0-a5b6cb1f5411.jpg)
</p>

# how-to-better-secure-your-mobile-application

# Hi there! <img width="30" src="https://camo.githubusercontent.com/e8e7b06ecf583bc040eb60e44eb5b8e0ecc5421320a92929ce21522dbc34c891/68747470733a2f2f6d656469612e67697068792e636f6d2f6d656469612f6876524a434c467a6361737252346961377a2f67697068792e676966">

# This is a guide for securing mobile application against most of threats that still continue to troble application's security.

## Prepared and maintained by [Nidhi Yashwanth](https://github.com/nidhiyashwanth)) who's been working on this project for past three months.

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












You can find me on [![Twitter][1.2]][1].
<!-- Icons -->
[1.2]: http://i.imgur.com/wWzX9uB.png (twitter icon without padding)
[1]: https://twitter.com/NidhiYashwanth/
