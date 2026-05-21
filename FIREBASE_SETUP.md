# Firebase Backend Setup — L.L Bicycle Center

This guide walks you through connecting your GitHub Pages site to Firebase so that contact form messages and newsletter subscriptions are saved to Firestore.

---

## Step 1 — Create a Firebase Project

1. Go to [https://console.firebase.google.com](https://console.firebase.google.com)
2. Click **"Add project"**
3. Name it something like `ll-bicycle-center`
4. Enable **Google Analytics** (recommended — already wired in the code)
5. Click **Create project**

---

## Step 2 — Register Your Web App

1. Inside your project, click the **`</>`** (Web) icon
2. Name the app (e.g. `LL Bicycle Website`)
3. **Do NOT** tick "Firebase Hosting" (you're using GitHub Pages)
4. Click **Register app**
5. You'll see a `firebaseConfig` object — **copy all the values**

---

## Step 3 — Paste Config into index.html

Open `index.html` and find this block near the bottom:

```js
const firebaseConfig = {
    apiKey:            "REPLACE_WITH_YOUR_API_KEY",
    authDomain:        "REPLACE_WITH_YOUR_AUTH_DOMAIN",
    projectId:         "REPLACE_WITH_YOUR_PROJECT_ID",
    ...
};
```

Replace each `"REPLACE_WITH_..."` value with the real values from step 2.

---

## Step 4 — Set Up Firestore Database

1. In Firebase Console → **Build → Firestore Database**
2. Click **Create database**
3. Choose **"Start in test mode"** (you'll secure it in Step 5)
4. Pick the region closest to Uganda — use **`europe-west1`** (Belgium) or **`me-west1`** (Tel Aviv) as the nearest available

This creates two collections automatically when the first form is submitted:

| Collection | What's stored |
|---|---|
| `messages` | Contact form submissions (name, email, subject, message, timestamp, read status) |
| `subscribers` | Newsletter emails (email, timestamp, active flag) |

---

## Step 5 — Secure Firestore with Rules

In Firebase Console → Firestore → **Rules**, replace the default rules with:

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Anyone can submit a message (write-only, no read)
    match /messages/{doc} {
      allow create: if request.resource.data.keys().hasAll(['name','email','subject','message'])
                    && request.resource.data.name is string
                    && request.resource.data.email is string
                    && request.resource.data.message.size() <= 2000;
      allow read, update, delete: if false; // only you read these (via console)
    }

    // Anyone can subscribe; no one can read the list from the frontend
    match /subscribers/{doc} {
      allow create: if request.resource.data.keys().hasAll(['email'])
                    && request.resource.data.email is string;
      allow read, update, delete: if false;
    }
  }
}
```

Click **Publish**.

---

## Step 6 — Add Your GitHub Pages Domain to Authorised Domains

1. Firebase Console → **Authentication → Settings → Authorised domains**
2. Add your domain:  `llbicyclecenter.ug`  (and `yourusername.github.io` if you use that URL too)

This prevents other websites from using your Firebase config to submit fake data.

---

## Step 7 — Push to GitHub Pages

```bash
git add index.html
git commit -m "Add Firebase backend for contact form and newsletter"
git push
```

Your site is live — form submissions now go directly to Firestore!

---

## Reading Your Submissions

Go to **Firebase Console → Firestore Database → Data tab**.

- Click `messages` to see all contact form enquiries
- Click `subscribers` to see newsletter signups

You can also export them to CSV from the overflow menu (⋮).

---

## Optional Extras

### Get email notifications for new messages
Use **Firebase Extensions → Trigger Email** to get an email to `llbicyclecenter@gmail.com` whenever a new document appears in `messages`.

Install it from:
Firebase Console → **Extensions → Explore Extensions → "Trigger Email from Firestore"**

### View analytics
Firebase Console → **Analytics → Dashboard** shows contact form submissions and product inquiry clicks logged as custom events.

---

## Security Reminder

Your `apiKey` in the HTML is **safe to be public** — Firebase API keys are not secret. They identify your project, not authenticate admin access. The Firestore Rules in Step 5 are what actually protect your data.
