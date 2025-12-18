# Deployment to Firebase Hosting

## Option 1: Deploy via CLI (Recommended)

### Prerequisites
- Firebase CLI installed: `npm install -g firebase-tools`
- Your Firebase project created and configured

### Steps

1. **Copy fixed files to main names:**
   ```bash
   cd /Users/agrimverma/Desktop/crossword-race
   cp index-fixed.html index.html
   cp admin-fixed.html admin.html
   ```

2. **Initialize Firebase (if not done yet):**
   ```bash
   firebase init hosting
   ```
   - Select project: `crossword-sprint`
   - Public directory: `.` (current directory)
   - SPA: `No`
   - Overwrite index.html: `No`

3. **Create `.firebaserc` file** (if missing):
   ```bash
   echo '{"projects": {"default": "crossword-sprint"}}' > .firebaserc
   ```

4. **Deploy:**
   ```bash
   firebase deploy --only hosting
   ```

5. **Your app is live!** URL will look like:
   ```
   https://crossword-sprint.firebaseapp.com
   ```

---

## Option 2: Deploy via Firebase Console

1. Go to Firebase Console → **Hosting**
2. Click **Connect Repository** (if using GitHub)
3. Or manually upload files (less convenient)

---

## Post-Deployment Checklist

After deployment, test:

- [ ] Player can log in at https://your-project.firebaseapp.com
- [ ] Player sees "Waiting for Admin..." screen
- [ ] Admin can log in at https://your-project.firebaseapp.com/admin.html
- [ ] Admin can click "START GAME"
- [ ] Player sees puzzle loading
- [ ] Puzzle displays correctly
- [ ] Player solves puzzle → next puzzle loads automatically
- [ ] Admin sees live leaderboard updating
- [ ] Admin can click "STOP GAME"
- [ ] Players see final score screen

---

## Troubleshooting Deployment

### 404 errors on page load
→ Make sure `firebase.json` exists and contains:
```json
{
  "hosting": {
    "public": ".",
    "ignore": ["firebase.json", ".firebaserc", "node_modules", ".git"]
  }
}
```

### CSS/JS not loading
→ Files must be in the same directory as `index.html`:
- `exolve-m.css`
- `exolve-m.js`

### Custom domain (optional)
1. In Firebase Console → **Hosting** → **Domain**
2. Click **Add Custom Domain**
3. Follow domain setup instructions

---

## Quick Commands Reference

```bash
# Deploy hosting only
firebase deploy --only hosting

# Deploy everything (if using other Firebase services)
firebase deploy

# Preview before deploying
firebase hosting:channel:deploy preview

# View logs
firebase hosting:log

# Stop a previous deploy
firebase hosting:disable
```

---

## Firebase Free Tier Limits

- **Hosting:** 1 GB storage, 10 GB/month bandwidth
- **Firestore:** 50,000 reads, 20,000 writes, 20,000 deletes per day
- **Authentication:** Unlimited

For a 1-hour competition with ~50 users:
- ~5,000 Firestore reads ✅
- ~500 Firestore writes ✅
- Well within free tier!

---

## Making Changes After Deployment

1. Edit `index.html`, `admin.html`, or `exolve-m.css`
2. Run: `firebase deploy --only hosting`
3. Changes live in ~5 seconds

---

## Advanced: GitHub Auto-Deploy

1. Go to Firebase Console → **Hosting**
2. Click **Connect Repository**
3. Select your GitHub repo
4. Firebase will auto-deploy on every push to `main` branch

