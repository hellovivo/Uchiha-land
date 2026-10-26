# Sasuke Uchiha — Anime Vault

A responsive GitHub Pages-ready anime-library frontend based on your two reference screenshots:
- Dark cinematic anime dashboard with violet/silver/crimson/iron styling.
- First screen is a dramatic Sasuke/Uchiha-inspired hero; scroll down for the library.
- Private member gate, separate admin login modal, admin console, video upload, editable thumbnail, search/filter, player and download links.
- Mobile responsive.

## Important before publishing

This is a **starter project**, not a secure complete production service until Firebase is configured and rules are published. Do not share your admin password or Firebase service-account keys.

### 1. Firebase web config

In Firebase Console → Project settings → Your apps → Web app, copy the web config and replace the values in `app.js`:
- `apiKey`
- `authDomain`
- `projectId`
- `storageBucket` (often `sasuke-uchiha-website.firebasestorage.app` or `.appspot.com`, use the exact value shown in Firebase)
- `messagingSenderId`
- `appId`

The public Firebase web config is expected to be in a frontend app; security comes from Authentication and Firebase Security Rules, not hiding this config.

### 2. Enable services

In Firebase Console:
1. Authentication → Sign-in method → enable **Email/Password**.
2. Create Firestore Database.
3. Storage → Get started (if available for your Firebase plan/project).
4. Deploy the Firestore and Storage rules below.

### 3. Create the admin account

1. Open the website and choose **Access** (the button is intentionally generic; there is no public Admin link in the nav).
2. Create a normal account using an admin username and a strong password (6+ chars; prefer a long unique password).
3. Firebase Console → Authentication → Users → copy that user's **UID**.
4. Firestore → create collection `admins`; create a document whose **Document ID is exactly the UID**. Add a field `role` (string) = `admin`.
5. Sign out and sign in using Access → admin username/password. The account will get the admin console only if its UID is in `admins`.

The admin modal is not linked in the public navigation, but no client-side “hidden route” is a security boundary. UID checks and Firebase rules are what protect writes.

### 4. Firestore rules

Publish these rules in Firestore → Rules. They allow signed-in members to read the video catalog, and only admins to create/update/delete video records.

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function signedIn() { return request.auth != null; }
    function isAdmin() {
      return signedIn() &&
        exists(/databases/$(database)/documents/admins/$(request.auth.uid));
    }

    match /admins/{uid} {
      allow read: if signedIn() && request.auth.uid == uid;
      allow create, update, delete: if false;
    }

    match /videos/{videoId} {
      allow read: if signedIn();
      allow create: if isAdmin()
        && request.resource.data.createdBy == request.auth.uid;
      allow update, delete: if isAdmin();
    }
  }
}
```

### 5. Storage rules

Publish in Firebase Storage → Rules. These rules restrict uploads/deletes to admins and require signed-in members to access video files. Confirm the `admins/{uid}` document exists before testing.

```javascript
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    function signedIn() { return request.auth != null; }
    function isAdmin() {
      return signedIn() &&
        firestore.exists(/databases/(default)/documents/admins/$(request.auth.uid));
    }

    match /videos/{uid}/{fileName} {
      allow read: if signedIn();
      allow write: if isAdmin() && request.auth.uid == uid
        && request.resource.size < 500 * 1024 * 1024
        && request.resource.contentType.matches('video/.*');
      allow delete: if isAdmin() && request.auth.uid == uid;
    }

    match /thumbnails/{uid}/{fileName} {
      allow read: if signedIn();
      allow write: if isAdmin() && request.auth.uid == uid
        && request.resource.size < 10 * 1024 * 1024
        && request.resource.contentType.matches('image/.*');
      allow delete: if isAdmin() && request.auth.uid == uid;
    }
  }
}
```

If Firebase Storage rules cannot use the Firestore lookup due to project setup, check Firebase's current cross-service rules guidance. Keep uploads denied until admin checks work. Do not use `allow read, write: if true`.

### 6. Publish with GitHub Pages (phone-friendly)

1. Download and extract the ZIP.
2. Open your repository `Sasuke-uchiha-wedsite` on GitHub.
3. Upload `index.html`, `style.css`, `app.js`, and `README.md` to the repository root (not only the ZIP).
4. Commit changes.
5. Repository → Settings → Pages → ensure the correct branch (usually `main`) and folder `/ (root)`.
6. Wait a few minutes, then open the published site and refresh. If old design remains, verify `index.html` in the repository root changed and check the Pages deployment status.

## Notes / limitations

- Username is mapped to a Firebase email-style ID internally; it is not a conventional email address. Keep usernames unique and use a strong password.
- Firebase Authentication minimum password length may be configured separately; this UI asks for at least 6 characters. A numeric 4-digit PIN is intentionally not used because it is too weak for a public website.
- “Download” opens the Firebase Storage download URL. Browser/device download behavior can vary; users with access can save the file.
- Anyone with a video download URL might be able to share it if the URL is exposed. For private/licensed content, consider a backend that issues authenticated short-lived download URLs.
- Only upload videos you own or have permission to distribute.
