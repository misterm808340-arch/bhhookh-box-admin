# 🍔 BHHOOKH BOX — Admin Dashboard

Real-time restaurant operations console for the **BHHOOKH BOX** food delivery MVP.
Built with **Next.js 16 + TypeScript + Tailwind CSS + shadcn/ui** on top of
**Firebase Authentication + Firestore** — the *same Firebase project* used by the
customer Android app, so orders placed on the phone appear here instantly.

> 📱 Customer Android app (React Native + Expo) lives in a separate `customer-android` folder/repo.
> 🔥 Firestore security rules live in [`firebase/`](./firebase).

## ✨ Features

| Area | What you get |
| --- | --- |
| 🔐 Login | Firebase Auth (email + password). Admins are whitelisted in the `admins` Firestore collection — **no credentials in the code**. |
| 📊 Overview | Total orders, per-status counts (pending / accepted / preparing / out for delivery / delivered / rejected) and total order value (rejected excluded). |
| ⚡ Real-time orders | Firestore `onSnapshot` feed — newest first, zero polling, zero refresh. |
| 🧾 Order cards | Copyable order ID, customer name / tap-to-call phone / address, items with quantities, subtotal, delivery fee, total, COD flag, IST timestamps + "time ago". |
| 🔁 Status workflow | `Pending → Accepted → Preparing → Out for Delivery → Delivered`, plus **Reject** (two-step confirm) for pending orders — enforced again by security rules. |
| 🔔 New-order alert | Lime banner + pulsing `NEW` badge + Web-Audio chime. Sound is enabled once via the **Sound On** button (browsers require a user gesture before audio can play). |
| 📱 Responsive | Desktop grid, mobile-first stacking. |

## 🗂 Project structure

```
├── src/
│   ├── app/                    # page.tsx (setup → loader → login → dashboard state machine), layout, brand theme
│   ├── components/admin/       # LoginForm, StatCard, OrderCard, OrdersBoard, NewOrderBanner…
│   ├── components/ui/          # shadcn/ui primitives
│   ├── hooks/                  # use-admin-auth (onAuthStateChanged + admins check), use-admin-orders (onSnapshot + stats)
│   └── lib/                    # firebase.ts (lazy init), orders.ts (types + status logic), sound.ts (chime)
├── firebase/                   # firestore.rules, firestore.indexes.json, firebase.json
├── .env.local.example          # Firebase web config template
└── next.config.ts              # standalone (default) or static export via NEXT_STATIC_EXPORT=1
```

## 🚀 Run locally

```bash
git clone https://github.com/<your-username>/bhhookh-box-admin.git
cd bhhookh-box-admin
npm install
cp .env.local.example .env.local     # paste your Firebase web config values
npm run dev                          # → http://localhost:3000
```

### `.env.local`

```
NEXT_PUBLIC_FIREBASE_API_KEY=...
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=...
NEXT_PUBLIC_FIREBASE_PROJECT_ID=...
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=...
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=...
NEXT_PUBLIC_FIREBASE_APP_ID=...
```

These values are public by design (they identify the Firebase project); access
control is enforced by Firebase Auth + the Firestore rules in [`firebase/firestore.rules`](./firebase/firestore.rules).

## 🔥 Firebase setup (first run)

1. **Firebase Console** → enable **Authentication → Email/Password** and **Firestore**.
2. Create your admin user under **Authentication → Users** (use a lowercase email).
3. Create a Firestore collection `admins` and add a document whose **ID is that exact email**, e.g. `{ role: "admin" }`.
4. Publish the rules:
   ```bash
   npm i -g firebase-tools
   firebase login
   cd firebase
   firebase deploy --only firestore:rules,firestore:indexes
   ```
5. Sign in on the dashboard, place a test order from the customer app, and watch it appear. 🎉

## ☁️ Deployment

**Option A — Firebase Hosting (static export, free tier friendly):**

```bash
npm run build:static        # emits ./out
cd firebase
firebase deploy --only hosting
```
(`firebase.json` already points hosting to `../out` when deploying from the `firebase/` folder.)

**Option B — Vercel / Netlify / Node host:**

```bash
npm run build
npm start                   # standalone server on port 3000
```
On Vercel just import the repo and add the six `NEXT_PUBLIC_FIREBASE_*` env vars.

## 🧪 Testing the end-to-end loop

1. Open the dashboard → sign in with your admin account.
2. Place an order in the customer Android app (name + 10-digit phone + address + COD).
3. The order appears at the top of the board within ~1 second, banner + chime fire.
4. Walk the order through `Accepted → Preparing → Out for Delivery → Delivered`.
5. Overview stats and totals update live; changes are written straight to Firestore.

## 📄 License

[MIT](./LICENSE) © 2026 BHHOOKH BOX
