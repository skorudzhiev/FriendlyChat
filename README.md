# FriendlyChat
**Version 1.0 2018/07/12**

This repository contains code for the FriendlyChat project in the [Firebase in a Weekend: Android by Google](https://www.udacity.com/course/firebase-in-a-weekend-by-google-android--ud0352) Udacity course.

## Overview

FriendlyChat is an app that allows users to send and receive text and photos in realtime across platforms.

![alt text](https://github.com/skorudzhiev/FriendlyChat/blob/master/readme_photos/FriendlyChat-SignIn.png | width=100) ![alt text](https://github.com/skorudzhiev/FriendlyChat/blob/master/readme_photos/FriendlyChat-chatScreen.png | width=100) 

### Device permissions
*App needs the following user's permissions to provide the featured functionality*
* Request user's permission to read and write to external storage
```XML
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```
```XML
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
* Request user's permission to access the internet
```XML
<uses-permission android:name="android.permission.INTERNET"/>
```

* Request user's permission to access device network state
```XML
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

```Gradle
defaultConfig {
        minSdkVersion 16
        targetSdkVersion 26
    }
```

* The app prompts the user to log in using email and google signIn
* Upon successfull authentication chat screen is loaded
* App is build upon the following features: 
  *  send and receive text in realtime across platforms
  *  send and receive photos in realtime across platforms
* Some of the words used in a conversation are replaced by emojis 😂 😸

```Gradle
dependencies {
    
    implementation 'com.firebaseui:firebase-ui-auth:4.0.1'
    // Displaying images
    implementation 'com.github.bumptech.glide:glide:4.7.1'
    //noinspection GradleCompatible
    implementation 'com.firebaseui:firebase-ui-auth:4.0.1'
    implementation 'com.google.firebase:firebase-auth:16.0.2'
    implementation 'com.google.firebase:firebase-core:16.0.1'
    implementation 'com.google.firebase:firebase-database:16.0.1'
    implementation 'com.google.firebase:firebase-storage:16.0.1'
    implementation 'com.google.firebase:firebase-messaging:17.1.0'
    implementation 'com.google.firebase:firebase-config:16.0.0'
    }
```

## Features

* SignIn Intent implemented with FirebaseUI

```Java
startActivityForResult(
                            AuthUI.getInstance()
                                    .createSignInIntentBuilder()
                                    .setIsSmartLockEnabled(true, true)
                                    .setAvailableProviders(Arrays.asList(
                                            new AuthUI.IdpConfig.EmailBuilder().build(),
                                            new AuthUI.IdpConfig.GoogleBuilder().build()))
                                    .build(),
                            RC_SIGN_IN);
```

* Uploading choosen photo to Firebase Storage and make a reference to it from the Realtime Database

```Java
// Upload file to Firebase Storage
            Uri selectedImageUri = data.getData();

            final StorageReference photoRef =
                    storageReference.child(selectedImageUri.getLastPathSegment());
            photoRef.putFile(selectedImageUri);
            photoRef.putFile(selectedImageUri).continueWithTask(new Continuation<UploadTask.TaskSnapshot, Task<Uri>>() {
                @Override
                public Task<Uri> then(@NonNull Task<UploadTask.TaskSnapshot> task) throws Exception {
                    if (!task.isSuccessful()) {
                        throw task.getException();
                    }
                    return photoRef.getDownloadUrl();
                }
            }).addOnCompleteListener(new OnCompleteListener<Uri>() {
                @Override
                public void onComplete(@NonNull Task<Uri> task) {
                    if (task.isSuccessful()) {
                        Uri downloadUri = task.getResult();
                        FriendlyMessage friendlyMessage = new FriendlyMessage(null, mUsername, downloadUri.toString());
                        databaseReference.push().setValue(friendlyMessage);
                    }
                }
            });
```

* Configured Remote Configs from Firebase to adjust the variable of the exchanged message length

```Java
private void fetchConfig() {
        long cacheExpiration = 3600;

        if (remoteConfig.getInfo().getConfigSettings().isDeveloperModeEnabled()) {
            cacheExpiration = 0;
        }

        remoteConfig.fetch(cacheExpiration).addOnSuccessListener(new OnSuccessListener<Void>() {
            @Override
            public void onSuccess(Void aVoid) {
                remoteConfig.activateFetched();
                applyRetrievedLengthLimit();
            }
        }).addOnFailureListener(new OnFailureListener() {
            @Override
            public void onFailure(@NonNull Exception e) {
                Log.d(TAG, "Error fetching config", e);
                applyRetrievedLengthLimit();
            }
        });
    }
```

## License
See [LICENSE](LICENSE)
