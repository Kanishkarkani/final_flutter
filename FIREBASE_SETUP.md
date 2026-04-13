# Firebase Setup Guide for Thayam Game

The error **"invalid data documents file must not be empty"** usually means Firebase isn't properly connected to your app. Follow these steps:

## Step 1: Create a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click **"Create a project"** → name it `thayamgame` (or your choice)
3. Enable Google Analytics (optional)
4. Click **"Create project"** and wait for it to initialize

## Step 2: Register Android App

1. In Firebase Console, click **"Android"** to register an Android app
2. Package name: `com.kanishkar.thayamgame` (from your `android/app/build.gradle`)
3. App nickname: `Thayam Game`
4. SHA-1 fingerprint: Run this command in your project folder:
   ```bash
   ./gradlew signingReport
   ```
   Copy the SHA-1 value from the output and paste it into Firebase
5. Click **"Register app"**

## Step 3: Download `google-services.json`

1. Firebase will show a download button for `google-services.json`
2. **Save it to**: `android/app/google-services.json`
3. Click **"Next"** to skip the download instructions (we'll configure gradle manually)

## Step 4: Update Android Gradle Files

### Update `android/build.gradle`:

At the end of the file, add (if not already present):
```gradle
dependencies {
    classpath 'com.google.gms:google-services:4.3.15'
}
```

### Update `android/app/build.gradle`:

At the **bottom** of the file, add:
```gradle
apply plugin: 'com.google.gms.google-services'
```

## Step 5: Enable Firestore Database

1. Back in Firebase Console, go to **"Build"** → **"Firestore Database"**
2. Click **"Create database"**
3. Choose **"Start in test mode"** (for development)
4. Choose a region close to you (e.g., `us-central1`)
5. Click **"Create"**

## Step 6: Set Firestore Security Rules (Optional for testing)

To allow read/write in test mode, use these rules:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true;
    }
  }
}
```

⚠️ **Warning**: This is only for testing. Before deploying, use proper authentication rules.

## Step 7: Verify Connection

Run the app:
```bash
flutter clean
flutter pub get
flutter run
```

**What to expect:**
- App launches → Start Screen
- **Online Play** section → "Create Room" button
- Click **"Create Room"**
  - If successful: Shows "Room ID: xxx" with player count
  - If error: Shows error message (read carefully — might be an auth issue)
- **Offline Play** section → "Play vs Computer" button should work immediately

## Troubleshooting

### Error: "You must be signed in to create a room"
- Firebase Auth anonymous sign-in failed
- Check internet connection
- Ensure `google-services.json` is in the correct location

### Error: "invalid data documents file must not be empty"
- `google-services.json` is missing or invalid
- Firestore isn't initialized
- Check step 5 (Firestore Database)

### Error: "Permission denied"
- Firestore security rules are too strict
- Use test mode rules (step 6) for now

## Testing

1. **Create Room (Online)**
   - Click "Create Room" → Shows Room ID
   - Share Room ID with a friend
   - Friend clicks "Join" → Enters Room ID
   - When both players join, "Start Game" button enables

2. **Play Offline**
   - Click "Play vs Computer"
   - Game starts immediately with local players

---

**Still stuck?**
- Check Android Studio logcat: `adb logcat | grep Flutter`
- Check the error message shown in the app (it now displays detailed errors)
- Ensure internet is connected
