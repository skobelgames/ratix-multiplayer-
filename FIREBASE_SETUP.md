# Firebase Setup Guide for RATIX Multiplayer

This guide will help you set up Google Firebase for RATIX's online multiplayer functionality.

## Step 1: Create a Firebase Project

1. Go to the [Firebase Console](https://console.firebase.google.com/)
2. Click "Add project" or "Create a project"
3. Enter a project name (e.g., "RATIX Multiplayer")
4. Accept the terms and click "Continue"
5. Choose whether to enable Google Analytics (optional)
6. Click "Create project"

## Step 2: Register Your Web App

1. In your Firebase project dashboard, click the **Web icon** (</>) to add a web app
2. Enter an app nickname (e.g., "RATIX Web App")
3. **Check** the box for "Also set up Firebase Hosting" (optional but recommended)
4. Click "Register app"

## Step 3: Get Your Firebase Configuration

After registering, you'll see a configuration object that looks like this:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  authDomain: "your-project-id.firebaseapp.com",
  projectId: "your-project-id",
  storageBucket: "your-project-id.appspot.com",
  messagingSenderId: "123456789012",
  appId: "1:123456789012:web:abcdef123456"
};
```

**Copy this configuration** - you'll need it in the next step.

## Step 4: Update Your Game File

1. Open the `online-play` file
2. Find the Firebase configuration section (around line 2931)
3. Replace the placeholder values with your actual Firebase config:

```javascript
const firebaseConfig = {
    apiKey: "YOUR_API_KEY",              // Replace with your apiKey
    authDomain: "YOUR_PROJECT_ID.firebaseapp.com",  // Replace with your authDomain
    projectId: "YOUR_PROJECT_ID",        // Replace with your projectId
    storageBucket: "YOUR_PROJECT_ID.appspot.com",   // Replace with your storageBucket
    messagingSenderId: "YOUR_MESSAGING_SENDER_ID",  // Replace with your messagingSenderId
    appId: "YOUR_APP_ID"                 // Replace with your appId
};
```

## Step 5: Enable Authentication

1. In Firebase Console, go to **Authentication** in the left sidebar
2. Click "Get started"
3. Go to the **Sign-in method** tab
4. Enable **Email/Password** authentication:
   - Click on "Email/Password"
   - Toggle "Enable" to ON
   - Click "Save"

## Step 6: Set Up Firestore Database

1. In Firebase Console, go to **Firestore Database** in the left sidebar
2. Click "Create database"
3. Choose **Start in production mode** (we'll set up rules next)
4. Select a Cloud Firestore location (choose one close to your users)
5. Click "Enable"

## Step 7: Configure Firestore Security Rules

1. In Firestore Database, go to the **Rules** tab
2. Replace the default rules with these:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users collection - users can only read/write their own profile
    match /users/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == userId;
    }

    // Games collection - players can read/write games they're part of
    match /games/{gameId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null;
      allow update: if request.auth != null &&
        (request.auth.uid == resource.data.player1.uid ||
         request.auth.uid == resource.data.player2.uid);
    }

    // Matchmaking collection - users can create/delete their own entries
    match /matchmaking/{matchId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null;
      allow delete: if request.auth != null;
    }
  }
}
```

3. Click "Publish"

## Step 8: Set Up Firebase Storage (Optional - for future profile pictures)

1. In Firebase Console, go to **Storage** in the left sidebar
2. Click "Get started"
3. Accept the default security rules
4. Choose a storage location
5. Click "Done"

## Step 9: Create Firestore Indexes (Important!)

For the leaderboard to work efficiently, you need to create a composite index:

1. Go to **Firestore Database** > **Indexes** tab
2. Click "Create Index"
3. Collection ID: `users`
4. Add these fields:
   - Field path: `stats.wins` | Order: Descending
5. Click "Create"

Alternatively, when you first try to load the leaderboard, Firebase will give you an error with a direct link to create the required index.

## Step 10: Test Your Setup

1. Open your `online-play` file in a web browser
2. You should see the login/signup screen
3. Try creating an account
4. If successful, you'll be able to:
   - Create a profile with username and avatar
   - Access the main menu
   - Find matches
   - View leaderboard
   - Track your stats

## Troubleshooting

### "Firebase initialization error"
- Make sure you've replaced ALL placeholder values in the firebaseConfig
- Check that your API key is correct (it should start with "AIza")

### "Permission denied" errors
- Verify that you've published the Firestore security rules
- Make sure you're logged in (check the browser console)

### Matchmaking not working
- Ensure Firestore is enabled and working
- Check the browser console for errors
- Verify that the `matchmaking` collection can be accessed

### Leaderboard shows "Error loading"
- You may need to create the composite index (see Step 9)
- Click the error link in the browser console to auto-create the index

## Features Included

✅ **User Authentication** - Email/password signup and login
✅ **User Profiles** - Custom username and avatar selection
✅ **Statistics Tracking** - Wins, losses, draws, high scores, total points
✅ **Achievements System** - 9 different achievements to unlock
✅ **Matchmaking** - Quick match system to find opponents
✅ **Real-time Gameplay** - Turn-based synchronization
✅ **Active Games** - Resume games in progress
✅ **Leaderboard** - Top 10 players ranked by wins
✅ **Offline Mode** - Still play vs AI when desired

## Firebase Costs

Firebase offers a generous **free tier (Spark Plan)** that includes:
- Authentication: Unlimited users
- Firestore: 50,000 reads/day, 20,000 writes/day
- Storage: 5 GB

This should be more than enough for a personal game or small community. Monitor your usage in the Firebase Console.

## Need Help?

If you encounter issues:
1. Check the browser console (F12) for error messages
2. Verify all Firebase services are enabled
3. Ensure security rules are published
4. Check that your Firebase config is correct

---

**Congratulations!** Your RATIX multiplayer system is now ready to use! 🎮
