# IITH Lost & Found

A simple campus board for posting and browsing lost/found items — built for Lambda Hackathon 2026 (theme: Smart Campus Solutions for IITH).

Anyone can post a lost or found item with a location and contact info, search/filter the board, and mark an item resolved once it's claimed. It's one HTML file and a free Firebase database — no server to run.

## 1. Set up Firebase (free, ~5 minutes)

This app needs somewhere to store items so everyone sees the same board. Firebase's free tier handles this with no backend code.

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and sign in with any Google account.
2. Click **Add project** → name it (e.g. `iith-lost-found`) → skip Google Analytics → **Create project**.
3. In the left sidebar, click **Build → Firestore Database** → **Create database** → choose **Start in test mode** → pick any location → **Enable**.
   - Test mode keeps this open for the hackathon. If you want to lock it down later, see the note at the bottom.
3b. Also click **Build → Storage** → **Get started** → **Start in test mode** → same location → **Done**. This is where uploaded photos get stored — needed for the image upload feature.
4. Click the gear icon (⚙️) next to "Project Overview" → **Project settings**.
5. Scroll to **Your apps** → click the **</>** (web) icon → give it a nickname → **Register app**.
6. Firebase shows you a `firebaseConfig` object. Copy it.
7. Open `index.html` in this folder, find this block near the bottom, and paste your values in:

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

That's the only code change needed to get it working.

## 2. (Optional but recommended) Set up EmailJS for automatic match notifications

This app compares every new post against the opposite board (lost vs found) and flags likely matches — same place tag, or an overlapping word in the item name. When someone posts and a match turns up, a "Notify" button appears next to it. If the matched person's contact is a phone number, it opens WhatsApp with a pre-filled message. If it's an email, EmailJS sends it automatically, no click needed.

The app works fine without this step — matches with email contacts will just need a manual "Notify" click instead of firing automatically. To enable auto-send:

1. Go to [emailjs.com](https://www.emailjs.com) → sign up free (200 emails/month free).
2. **Email Services** → **Add New Service** → connect Gmail (or any provider) → note the **Service ID**.
3. **Email Templates** → **Create New Template**. Use variables `{{to_email}}`, `{{matched_item}}`, `{{place}}`, `{{poster_contact}}` in the body, e.g.:
   > Someone may have found your *{{matched_item}}* near {{place}}. Reach them at: {{poster_contact}}
   Note the **Template ID**.
4. **Account → General** → copy your **Public Key**.
5. In `index.html`, find this block and fill in your three values:

```js
const EMAILJS = { publicKey: "YOUR_EMAILJS_PUBLIC_KEY", serviceId: "YOUR_SERVICE_ID", templateId: "YOUR_TEMPLATE_ID" };
```

## 3. Run it locally

Just open `index.html` in a browser — double-click it, or drag it into a Chrome tab. No install, no build step.

(If items don't load, open the browser console (F12) — it'll usually be a typo in `firebaseConfig`. The page also now shows a red error banner on-screen if something breaks, so you don't need devtools to see what went wrong.)

**Testing on a phone:** don't AirDrop or message the raw `index.html` file to your phone and open it directly — phones block the network calls Firebase needs when a page is opened as a local file, and you'll see a generic "script error." Test it from an actual link instead: push to GitHub, turn on GitHub Pages (step 5 below), and open that URL on your phone.

## 4. Push to GitHub

```bash
git init
git add .
git commit -m "IITH Lost & Found - Lambda Hackathon 2026"
git remote add origin <your-empty-github-repo-url>
git push -u origin main
```

Make sure the repo is **public** — that's a submission requirement.

## 5. (Optional) Put it online with a link

GitHub Pages is free and works with this project as-is:
1. On GitHub, go to your repo → **Settings → Pages**.
2. Under "Branch," pick `main` and `/ (root)` → **Save**.
3. Your live link will appear at the top after a minute (`https://<username>.github.io/<repo>/`).

## How it works (for your own understanding)

- `index.html` is the whole app: structure, styling, and logic in one file.
- Firestore is a live database — `onSnapshot` means the page updates instantly for everyone when someone posts or resolves an item, with no refresh needed.
- Firebase Storage holds the uploaded photos; Firestore just stores the link to each photo.
- "Today" / "Yesterday" / "Xd ago" is calculated automatically from the post's timestamp every time the page renders — nobody types it in.
- Place tags: the poster picks one tag from the `PLACE_TAGS` list when posting; viewers can select any combination of place chips plus a time filter plus the search box, and all three combine (AND logic).
- Smart matching: whenever you post, the app checks every unresolved post on the *opposite* board (lost checks against found, and vice versa) for a shared place tag or overlapping keyword in the item name, and surfaces any hits immediately — plus a live "🔎 N matches" badge stays on each card so people browsing later can see it too.
- Contact button: every card has one. It opens WhatsApp (if the contact is a phone number) or a pre-filled email (if it's an email) straight to that poster — no need to search for their number, no need to wait for the auto-match system to fire. This is the actual reply flow: someone finds an item, searches the board, and if a matching Lost post already exists, hits Contact on it directly.
- No login system, so anyone can post or mark items resolved. That keeps setup simple for the hackathon; a natural "next step" to mention in your pitch is adding login so only the original poster can mark their item resolved.

## Editing the place tags

The board currently uses these IITH locations, set in `index.html` near the top of the `<script>` section:

```js
const PLACE_TAGS = [
  "Freshers Hostels","Hostel Circle","Old Hostels","New Hostels",
  "Lecture Hall Complex","KRC","Main Gate","Faculty Towers","SNCC","Wet Canteen"
];
```

To add, remove, or rename a location, just edit this list — the filter chips and the post form pick it up automatically, and each tag gets its own color without any extra setup.

## Ideas to extend if you have time left

- Multiple photos per item.
- Auto-archive resolved items after a few days.
- Login so only the original poster can resolve their own item.

These aren't required — the current version already covers a full flow (post with photo → browse → filter by place/time/search → share on WhatsApp → resolve), which is what the judging criteria (functionality, completeness, UI/UX) look for.
