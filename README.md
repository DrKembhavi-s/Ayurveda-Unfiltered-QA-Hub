# 🌿 Dr. Kembhavi's Ayurveda Unfiltered Q&A Hub

A fully functional, single-file web platform for BAMS students to submit academic, clinical, research, and exam-related questions — and receive answers from Dr. Aakash Kembhavi and invited guest faculty. Built with Firebase and deployable on GitHub Pages at zero cost.

---

## ✨ Features

| Feature | Description |
|---|---|
| **Student Registration & Login** | Email/password or Google Sign-In, with year and college profile |
| **Question Submission** | Tagged by subject, type, BAMS year, with optional anonymity |
| **Public Q&A Archive** | All answered questions visible to every registered student, searchable and filterable |
| **Examiner's Lens** | Every answer can carry a special note on how the topic is evaluated in written papers or viva voce |
| **Guest Sessions** | Scheduled sessions with invited guest faculty — Dr. Ayudha or any colleague — who answer questions in their assigned session |
| **Announcements** | Admin broadcasts pinned announcements visible on every student's home dashboard, with optional session links |
| **Admin Dashboard** | Full queue of pending questions, session management, and announcement management |
| **Guest Teacher Panel** | Guest faculty see only their assigned session and its pending questions |
| **Answer Attribution** | Guest answers carry the guest teacher's name; admin answers carry Dr. Kembhavi's |

---

## 🗂 Repository Structure

```
/
├── index.html        ← The entire platform (single file)
└── README.md         ← This file
```

That is the complete repository. No frameworks, no build tools, no dependencies to install.

---

## 🔥 Firebase Setup (Required Before First Deploy)

The platform uses **Firebase** for authentication and the database. The free **Spark plan** is sufficient for this platform's scale.

### Step 1 — Create a Firebase Project

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project** → give it a name (e.g. `bams-qa-hub`) → create
3. On the project dashboard, click the **web icon (`</>`)** to register a web app
4. Give the app a nickname → click **Register app**
5. Firebase will display your **config object** — keep this page open for Step 3

### Step 2 — Enable Authentication

1. In the Firebase console sidebar, go to **Build → Authentication**
2. Click **Get started**
3. Under **Sign-in method**, enable:
   - **Email/Password** — click → toggle Enable → Save
   - **Google** — click → toggle Enable → add your support email → Save

### Step 3 — Enable Firestore Database

1. Go to **Build → Firestore Database**
2. Click **Create database**
3. Choose **Start in test mode** (you will harden the rules later — see Security Rules section below)
4. Select your preferred region (e.g. `asia-south1` for India) → Done

### Step 4 — Add Your Config to `index.html`

Open `index.html` and find the configuration block near the top of the `<script>` section. Replace the placeholder values with your actual Firebase config:

```javascript
const FB_CFG = {
  apiKey:            "YOUR_ACTUAL_API_KEY",
  authDomain:        "your-project.firebaseapp.com",
  projectId:         "your-project-id",
  storageBucket:     "your-project.appspot.com",
  messagingSenderId: "123456789",
  appId:             "1:123456789:web:abcdef"
};

const ADMIN_EMAIL = "your-actual-email@gmail.com";
```

> ⚠️ **`ADMIN_EMAIL`** must exactly match the email address you will use to log in as administrator. This is how the platform recognises you as the admin and reveals the Admin dashboard.

---

## 🚀 GitHub Pages Deployment

### Step 1 — Create a GitHub Repository

1. The repository is already created at:
   **[github.com/DrKembhavi-s/Ayurveda-Unfiltered-QA-Hub](https://github.com/DrKembhavi-s/Ayurveda-Unfiltered-QA-Hub)**
2. Upload `index.html` and `README.md` to the repository root

### Step 2 — Enable GitHub Pages

1. In your repository, go to **Settings → Pages**
2. Under **Source**, select **Deploy from a branch**
3. Choose branch: `main` (or `master`) → folder: `/ (root)` → Save
4. GitHub will provide a live URL in the format:
   ```
   https://YOUR-USERNAME.github.io/REPO-NAME/
   ```
5. Your platform is now live. Share this URL with your students.

---

## 🔐 Firebase Security Rules (Recommended After Testing)

Once you have confirmed the platform works correctly, replace the default test rules in **Firestore → Rules** with the following to protect your data:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Users can read and write their own profile
    match /users/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == userId;
    }

    // Any registered student can submit a question
    // Only admin can update (answer) questions
    match /questions/{qId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null;
      allow update: if request.auth != null &&
                    (request.auth.token.email == "YOUR_ADMIN_EMAIL@gmail.com" ||
                     isGuestForQuestion(request.auth.token.email, resource.data.sessionId));
    }

    // Sessions are readable by all; writable only by admin
    match /sessions/{sId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null &&
                   request.auth.token.email == "YOUR_ADMIN_EMAIL@gmail.com";
    }

    // Announcements are readable by all; writable only by admin
    match /announcements/{aId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null &&
                   request.auth.token.email == "YOUR_ADMIN_EMAIL@gmail.com";
    }
  }
}
```

> Replace `YOUR_ADMIN_EMAIL@gmail.com` with your actual admin email in the rules above.

---

## 🗄 Firestore Data Structure

The platform uses four collections. These are created automatically when the first documents are written — no manual setup needed.

### `users`
```
users/{uid}
  name          : string
  email         : string
  year          : string   — e.g. "3rd Year BAMS"
  college       : string
  needsProfile  : boolean  — true if registered via Google and profile incomplete
  joinedAt      : timestamp
```

### `questions`
```
questions/{questionId}
  title         : string
  detail        : string
  subject       : string
  qtype         : string   — Academic / Clinical / Research / Exam Prep / Viva Voce
  year          : string
  anonymous     : boolean
  askedBy       : string   — user uid
  askerName     : string
  askerCollege  : string
  status        : string   — "pending" | "answered"
  sessionId     : string?  — null if not linked to a session
  sessionTitle  : string?
  answer        : string?
  examinerNote  : string?
  answeredByName: string?
  createdAt     : timestamp
  answeredAt    : timestamp?
```

### `sessions`
```
sessions/{sessionId}
  title         : string
  description   : string
  subject       : string
  guestName     : string
  guestTitle    : string   — credentials/designation
  guestEmail    : string   — must match the guest's login email exactly
  scheduledAt   : timestamp
  status        : string   — "upcoming" | "active" | "completed"
  createdAt     : timestamp
```

### `announcements`
```
announcements/{announcementId}
  title         : string
  body          : string
  sessionId     : string?  — optional link to a session
  pinned        : boolean
  createdAt     : timestamp
```

---

## 👤 Role Guide

### Dr. Kembhavi (Admin)
- Log in with the email set as `ADMIN_EMAIL` in the config
- The **⚙ Admin** button appears in the navigation bar
- From Admin, manage:
  - **Q&A Queue** — view pending questions and post answers with optional Examiner's Lens notes
  - **Sessions** — create and edit guest sessions, assign guest teacher emails, set status
  - **Announcements** — post pinned or unpinned announcements, optionally linked to a session

### Guest Teacher (e.g. Dr. Ayudha, Dr. Anita)
1. Admin creates a Session and enters the guest teacher's email
2. Guest teacher registers/logs in using that exact email
3. The **My Session** button appears in their navigation bar
4. They see only the session(s) assigned to them, and pending questions within those sessions
5. They answer questions directly — their name appears on the answer card

### Student
- Register with name, year, and college, or use Google Sign-In
- Browse all answered questions on the dashboard
- Filter by question type; search by keyword or subject
- View upcoming and past sessions; submit questions to a specific session
- Track their own submitted questions under **Profile**

---

## 🛠 Customisation

All customisable content is in the `<script>` block of `index.html`.

| What to change | Where to find it |
|---|---|
| Admin email | `const ADMIN_EMAIL = "..."` |
| Firebase project | `const FB_CFG = {...}` |
| Subject list | `const SUBJECTS = [...]` |
| Question types | `const QTYPES = [...]` |
| BAMS year options | `const YEARS = [...]` |
| Platform name | Update the string `"Dr. Kembhavi's Ayurveda Unfiltered Q&A Hub"` in `renderNav()` and `renderAuth()` |
| Answer attribution | Update `"Dr. Aakash Kembhavi"` and credentials in `loadQuestion()` |
| Colour scheme | Modify CSS variables in `:root` — `--g` (green), `--s` (saffron), `--v` (violet for sessions) |

---

## 🌐 Custom Domain (Optional)

To serve the platform from a custom domain (e.g. `qa.astangawellness.com`):

1. In your domain registrar's DNS settings, add a `CNAME` record pointing to `DrKembhavi-s.github.io`
2. In **GitHub → Ayurveda-Unfiltered-QA-Hub → Settings → Pages**, enter your custom domain
3. GitHub will provision an SSL certificate automatically within minutes

---

## 📋 Recommended Firestore Indexes

If Firestore returns an index error in the browser console, follow the link it provides to create the required composite index automatically. The queries most likely to need this are:

- `questions` — `sessionId` (ascending) + `status` (ascending)
- `sessions` — `guestEmail` (ascending) + `status` (ascending)
- `announcements` — `createdAt` (descending)

Each index takes 1–2 minutes to build after creation.

---

## 📦 Tech Stack

| Layer | Technology |
|---|---|
| Hosting | GitHub Pages (free, static) |
| Frontend | Vanilla HTML, CSS, JavaScript — no frameworks |
| Authentication | Firebase Authentication (Email/Password + Google OAuth) |
| Database | Cloud Firestore (NoSQL, real-time) |
| Firebase SDK | Firebase Compat v9.22.0 via CDN |

---

## 📄 Licence

This platform was developed for the exclusive use of Dr. Aakash Kembhavi's academic and clinical Q&A initiative under **Astanga Wellness Pvt. Ltd.** and the **ARISE Programme**.

---

*Built with Firebase + GitHub Pages · Maintained by Dr. Aakash Kembhavi · [github.com/DrKembhavi-s/Ayurveda-Unfiltered-QA-Hub](https://github.com/DrKembhavi-s/Ayurveda-Unfiltered-QA-Hub)*
