# Firebase Setup Guide for Crossword Sprint

## Step 1: Create Firebase Project (If Not Already Done)

1. Go to https://console.firebase.google.com/
2. Click **Add Project**
3. Name it: `crossword-sprint`
4. Choose a region
5. Click **Create Project**

---

## Step 2: Enable Google Authentication

1. In Firebase Console, go to **Authentication** (left sidebar)
2. Click **Get Started**
3. Click **Google** provider
4. Enable it
5. Add your email to **OAuth consent screen** → **Test users**

---

## Step 3: Create Firestore Database

1. Go to **Firestore Database** (left sidebar)
2. Click **Create Database**
3. Choose **Start in production mode**
4. Choose region (US-central1 recommended)
5. Click **Create**

---

## Step 4: Initialize Database Structure

You need to create this structure in Firestore:

```
collections/
  └── competitions/
      └── sprint_001/
          - status: "waiting"
          - puzzleSet: [] (array of puzzle strings)
          - admins: ["your-email@gmail.com"]
```

### How to Create It:

1. In Firestore Console, click **Start Collection**
2. Name it: `competitions`
3. Click **Next**
4. Add Document ID: `sprint_001`
5. Add these fields:

| Field | Type | Value |
|-------|------|-------|
| `status` | String | `waiting` |
| `puzzleSet` | Array | (empty for now, add puzzles later) |
| `admins` | Array | Add your email: `["your-email@gmail.com"]` |

6. Click **Save**

### Add Subcollection for Players:

1. Click on the `sprint_001` document
2. Click **Start Collection**
3. Name it: `players`
4. Leave it empty (players will be added when they log in)

---

## Step 5: Add Firestore Security Rules

**CRITICAL:** Without these rules, anyone can steal/delete your data!

1. In Firestore Console, go to **Rules** tab
2. Replace all content with this:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Competition document - anyone can read, only admins can update
    match /competitions/{compId} {
      allow read: if true;  // Everyone can read game status
      allow update, create: if request.auth.uid != null && 
                             resource.data.admins.hasAny([request.auth.token.email]);
      allow delete: if false;  // Never allow deletion
    }
    
    // Players subcollection - anyone can create/read, only self can update
    match /competitions/{compId}/players/{userId} {
      allow read: if true;  // Everyone can see leaderboard
      allow create: if request.auth.uid != null;
      allow update: if request.auth.uid == userId;  // Only own document
      allow delete: if false;
    }
  }
}
```

3. Click **Publish**

---

## Step 6: Get Your Firebase Config

1. In Firebase Console, click **Settings** (gear icon) → **Project Settings**
2. Scroll to **Your apps** section
3. Click **</>** (web app) button
4. Copy the config object that looks like:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "crossword-sprint.firebaseapp.com",
  projectId: "crossword-sprint",
  storageBucket: "crossword-sprint.firebasestorage.app",
  messagingSenderId: "152789...",
  appId: "1:152789...:web:...",
  measurementId: "G-..."
};
```

5. Update this config in both:
   - `index-fixed.html`
   - `admin-fixed.html`

---

## Step 7: Add Sample Puzzles

1. In Firestore Console, open `competitions` → `sprint_001`
2. Click on the `puzzleSet` array field
3. Click **Edit**
4. Click **Add array item**
5. Paste the first puzzle from `SAMPLE_PUZZLES.md`
6. Repeat for all puzzles you want to add
7. Click **Update**

**Tip:** Start with just 2-3 puzzles to test!

---

## Step 8: Update Admin Email

1. In `admin-fixed.html`, find this line:

```javascript
const ADMIN_EMAILS = [
    "agrimverma@gmail.com",  // Replace with your email
];
```

2. Replace with your actual email address

---

## Step 9: Enable Firebase Hosting (Optional but Recommended)

If you want to deploy this online:

1. In Firebase Console, go to **Hosting** (left sidebar)
2. Click **Get Started**
3. Follow the CLI setup steps below

---

# Firebase CLI Setup

If you want to deploy to Firebase Hosting:

```bash
# Install Firebase Tools (if not already installed)
npm install -g firebase-tools

# Log in to Firebase
firebase login

# Initialize Firebase in your project directory
cd /Users/agrimverma/Desktop/crossword-race
firebase init hosting

# When prompted:
# - Select your project: "crossword-sprint"
# - What do you want to use as your public directory? "." (current directory)
# - Configure as SPA? "No"
# - Overwrite index.html? "No"
```

---

# Testing Locally

1. Copy `index-fixed.html` to `index.html`:
   ```bash
   cp index-fixed.html index.html
   cp admin-fixed.html admin.html
   ```

2. Start a local server:
   ```bash
   python3 -m http.server 8000
   ```

3. Open http://localhost:8000 in your browser

4. Test login and game flow

---

# Troubleshooting

## "Competition document 'sprint_001' not found!"

→ Make sure you created the document in Firestore with exact name `sprint_001`

## "You do not have admin privileges"

→ Update `ADMIN_EMAILS` in `admin-fixed.html` with your email

## Players can't see the leaderboard

→ Check Firestore Rules - make sure `allow read: if true;` is set for players

## Puzzles don't load

→ Check that `puzzleSet` array is populated in Firestore document

## "Error loading puzzle"

→ Make sure puzzle text is valid Exolve format - check browser console for details

---

# Important Notes

- **Security:** Always use the Firestore rules provided above
- **Backups:** Download your Firestore data periodically
- **Testing:** Test with 1-2 puzzles first before uploading 10+
- **Timezone:** Times in leaderboard use user's local timezone
- **Free Tier:** Firebase Free Tier supports ~100,000 reads/writes per day (plenty for 100 users)

