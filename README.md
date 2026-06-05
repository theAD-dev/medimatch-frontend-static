# MediMatch

Healthcare coordination platform frontend — customers, pharmacies, and telehealth providers.

All data is mocked (static JSON + `localStorage`). No real backend or payments.

---

## Quick start

```bash
npm install
npm run dev
```

Open [http://localhost:3000/login](http://localhost:3000/login).

The login form is prefilled with the Jane Doe customer account. Click any **demo account card** on the login page to autofill credentials, then sign in.

---

## Demo accounts

All accounts use password: **`password123`**

| Role | Name | Email | Best for |
|------|------|-------|----------|
| Customer | Jane Doe | `jane@example.com` | Full demo — quotes, orders, telehealth chat, notifications |
| Customer | John Smith | `john@example.com` | Fresh customer flows (signup alternative) |
| Pharmacy | Pharmacy A Admin | `pharmacy@example.com` | Dashboard, quote management, orders, subscription |

### Pre-loaded data (Jane Doe)

| Type | ID | Notes |
|------|-----|-------|
| RFQ with offers | `rfq-seed-002` | 2 pharmacy offers ready to compare |
| Delivered order | `order-001` | Metformin from Pharmacy A — feedback/complaint demo |
| Telehealth chat | `chat-001` | Waiting for provider — use provider portal to schedule |
| Notification | — | Unread "New pharmacy offer" in notification center |

---

## All routes

| Route | Who | Purpose |
|-------|-----|---------|
| `/login` | Guest | Sign in |
| `/signup` | Guest | Customer or pharmacy registration |
| `/forgot-password` | Guest | Request password reset (simulated) |
| `/reset-password` | Guest | Set new password with token |
| `/customer/dashboard` | Customer | Home — stats, quick actions, recents |
| `/customer/medicines` | Customer | Upload prescription → RFQ |
| `/customer/quotes` | Customer | List active quote requests |
| `/customer/quotes/[rfqId]` | Customer | Compare and accept pharmacy offers |
| `/customer/orders` | Customer | Order list |
| `/customer/orders/[orderId]` | Customer | Order detail, complaint, feedback |
| `/customer/complaints` | Customer | Complaint status tracking |
| `/customer/notifications` | Customer | Notification center |
| `/customer/search` | Customer | Global search |
| `/customer/telehealth/questionnaire` | Customer | Health questionnaire (step 1) |
| `/customer/telehealth/providers` | Customer | Browse filtered providers |
| `/customer/telehealth/chats` | Customer | Chat list |
| `/customer/telehealth/chat/[chatId]` | Customer | Chat thread |
| `/pharmacy/dashboard` | Pharmacy | Stats, recent requests, notifications |
| `/pharmacy/quotes` | Pharmacy | Incoming RFQs — submit/reject offers |
| `/pharmacy/orders` | Pharmacy | Mark payment received / delivered |
| `/pharmacy/subscription` | Pharmacy | Plans + buy credits |
| `/telehealth/provider` | Public (no login) | Provider schedules patient visits |

Admin panel is intentionally excluded.

## UI & platform features

- **Toasts** — success/error feedback on offers, orders, complaints, credits, RFQ submit
- **Skeleton loaders** — orders, notifications, complaints, pharmacy quotes
- **Tables + pagination** — customer orders, notifications, complaints; pharmacy orders
- **Drawers** — complaint details, pharmacy quote editor (mobile)
- **Framer Motion** — toast animations, drawer slide, page fade-in
- **Dark mode** — toggle in customer/pharmacy header (persisted in localStorage)
- **Complaint images** — optional upload (max 3) when raising a complaint from order detail
- **Subscription guards** — expired subscription blocks pharmacy access; zero credits blocks offer submit (1 credit per offer)
- **Redux slices** — `complaintSlice`, `feedbackSlice`, `toastSlice`, `themeSlice`
- **messages.json** — chat messages stored separately and hydrated into chat threads

---

## Flow guides

### 1. Login

**URL:** [http://localhost:3000/login](http://localhost:3000/login)

1. Open the login page.
2. Click a demo account card (Jane, John, or Pharmacy A) to autofill email/password.
3. Click **Sign in**.
4. You are redirected by role:
   - Customer → `/customer/dashboard`
   - Pharmacy → `/pharmacy/dashboard` (or `/pharmacy/subscription` if no active plan)

---

### 2. Customer signup

**URL:** [http://localhost:3000/signup](http://localhost:3000/signup) (Customer tab)

1. Fill in: full name, email, phone, password, confirm password, address.
2. Submit.
3. You are signed in and sent to `/customer/dashboard`.

---

### 3. Pharmacy signup

**URL:** [http://localhost:3000/signup](http://localhost:3000/signup) → **Pharmacy** tab

1. Fill in: business name, email, phone, license number, address, password, confirm password.
2. Submit.
3. You are sent to `/pharmacy/subscription` to choose a plan before using the dashboard.

---

### 4. Forgot & reset password

**Forgot:** [http://localhost:3000/forgot-password](http://localhost:3000/forgot-password)

1. Enter a registered email (e.g. `jane@example.com` or `pharmacy@example.com`).
2. Click **Send reset link**.
3. Click **Continue to reset password** (simulated email link).

**Reset:** [http://localhost:3000/reset-password?token=...](http://localhost:3000/reset-password)

1. Enter new password + confirm password.
2. Submit → redirected to login with the new password saved in `localStorage`.

---

### 5. Customer dashboard

**URL:** `/customer/dashboard`  
**Login as:** `jane@example.com`

From the sidebar or quick-action cards you can reach every customer feature.

**What you see:**
- Stats: active requests, completed deliveries, provider searches left, telehealth usage
- Quick actions: Find Medicines, Telehealth, Quotes, Chats, Orders, Feedback
- Recent quotes, chats, deliveries, notifications

---

### 6. Find medicines (prescription → RFQ)

**URL:** `/customer/medicines`  
**Login as:** any customer

```
Upload prescription → OCR extract → Select medicines → Submit RFQ → Pharmacies notified
```

**Step-by-step:**

1. Go to **Find Medicines** (sidebar or dashboard).
2. **Upload** — drag & drop or browse. Accepts JPG, PNG, PDF. Multiple files supported.
3. Click **Extract Medicines** — mock OCR runs (~1s loading), then the medicine modal opens.
4. In the modal:
   - Check/uncheck medicines, select all, edit quantities
   - Add custom medicines, set optional expected prices
   - Search within the list
5. Confirm selection → review on the **Create Quote** screen (medicines, prices, delivery address).
6. Click **Submit quote request**.
7. RFQ is broadcast to all pharmacies. Note the RFQ ID on the success screen.
8. Go to **Quotes** once pharmacies respond.

---

### 7. Compare quotes & accept an offer

**URL:** `/customer/quotes` → `/customer/quotes/[rfqId]`  
**Login as:** `jane@example.com`

**Quick demo (pre-seeded):**

1. Go to **Quotes**.
2. Open request **`rfq-seed-002`** (2 pharmacy responses).
3. On the comparison page:
   - Sort by **Lowest price**, **Highest rated**, or **Fastest delivery**
   - Review pharmacy name, badge, total, delivery date, per-medicine availability
4. Click **Accept offer** on your preferred card.
5. Confirm in the modal.
6. The winning quote is locked; other offers for that RFQ are rejected.
7. An order is created automatically → go to **Orders**.

---

### 8. Orders, payment status, complaints & feedback

**URL:** `/customer/orders` → `/customer/orders/[orderId]`  
**Login as:** `jane@example.com`

**Pre-seeded order:** open **`order-001`** (delivered Metformin).

**On the order detail page:**

| Action | When | How |
|--------|------|-----|
| View timeline | Always | Accepted → Payment pending → Payment completed → Delivered |
| Raise complaint | Any time | **Raise complaint** → reason + description → Submit (stored locally) |
| Leave feedback | After delivery | Star rating + comment → Submit (updates pharmacy rating/badge) |

**Full payment flow (new order):**

1. Accept a quote (flow 7) → order appears with `payment_pending`.
2. Log in as **`pharmacy@example.com`** → **Orders**.
3. Click **Mark payment received** on that order.
4. Back as Jane → order shows `payment_completed`.
5. Pharmacy clicks **Mark delivered**.
6. Jane refreshes order → `delivered` → feedback widget appears.

---

### 9. Notifications

**URL:** `/customer/notifications`  
**Login as:** `jane@example.com`

- Bell icon in the header shows unread count.
- Events that create notifications: new quote offer, quote accepted, payment completed, delivery completed, complaint submitted, visit scheduled.
- Click **Mark all read** to clear unread state.

---

### 10. Global search

**URL:** `/customer/search`  
**Login as:** any customer

1. Enter a query (medicine name, pharmacy name, order ID, provider specialty, etc.).
2. Click **Search**.
3. Results are grouped: quotes, orders, chats, medicines, providers.
4. Search icon in the header (desktop) also links here.

**Try:** `Panadol`, `Pharmacy A`, `Metformin`, `General`

---

### 11. Telehealth — questionnaire → provider → chat

**Login as:** `jane@example.com`

```
Questionnaire → Filtered providers → Start chat → Provider schedules visit → Feedback
```

**Step-by-step:**

1. **Questionnaire** — `/customer/telehealth/questionnaire`
   - 4 steps: category, symptoms, duration, demographics (age, language, gender, history)
   - Progress bar, Back/Next, Submit on last step
2. **Providers** — `/customer/telehealth/providers`
   - Only verified providers shown, filtered by questionnaire answers
   - 5 provider searches per month (tracked per customer)
   - Click **Select** on a provider card → chat is created
3. **Chats** — `/customer/telehealth/chats`
   - Jane already has **`chat-001`** (status: waiting)
4. **Chat thread** — open the chat
   - Send messages, view provider details
   - **Visit** button shows schedule once set
   - **Cancel** available while waiting or scheduled

---

### 12. Telehealth provider portal (no login)

**URL:** [http://localhost:3000/telehealth/provider](http://localhost:3000/telehealth/provider)  
**No account needed** — open in a separate tab while Jane's chat is open.

1. Select a **provider profile** (match the one linked to the chat, e.g. provider for `chat-001`).
2. Pick a **pending patient chat** from the list.
3. Set **visit date**, **visit time**, and **visit details** (instructions for the patient).
4. Click **Submit schedule**.
5. Switch back to Jane's chat thread — a system message appears: visit scheduled.
6. Chat status updates to **scheduled**. Jane can click **Visit** to see date/time.

---

### 13. Telehealth feedback

**URL:** inside chat thread after visit is completed

1. After the provider marks a visit complete (or chat reaches completed status), the feedback widget appears in the chat.
2. Rate 1–5 stars, add a comment, submit.
3. Feedback is stored locally.

---

### 14. Pharmacy dashboard

**URL:** `/pharmacy/dashboard`  
**Login as:** `pharmacy@example.com`

**What you see:**
- Credits remaining, pending/accepted quotes, completed orders, badge level
- Recent incoming requests
- Notifications panel (mark all read)
- Subscription status card

Requires an active subscription. New pharmacy signups are redirected to subscription first.

---

### 15. Pharmacy subscription & credits

**URL:** `/pharmacy/subscription`  
**Login as:** `pharmacy@example.com` (or new pharmacy signup)

**Choose a plan (new pharmacies):**

| Plan | Credits/month |
|------|---------------|
| Starter | 50 |
| Professional | 100 |
| Enterprise | 500 |

1. Click **Select plan** on any tier.
2. Redirected to pharmacy dashboard with credits applied.

**Buy extra credits (active subscription):**

1. On the subscription page, use **Purchase credits** packs: 50 / 100 / 500.
2. Credits are added instantly (simulation only).

---

### 16. Pharmacy quote management

**URL:** `/pharmacy/quotes`  
**Login as:** `pharmacy@example.com`

1. Left panel: incoming quote requests with status badges.
2. Click a request → quote editor opens.
3. Per medicine:
   - Edit **offered price** (green if lower than expected, red strikethrough if higher)
   - Mark **unavailable** or **delivery later**
   - Set **delivery date**
4. **Submit offer** → customer gets a notification and can compare.
5. **Reject** → quote marked rejected.

**Try with:** `quote-002` (pending request from John Smith).

---

### 17. Pharmacy orders (payment & delivery)

**URL:** `/pharmacy/orders`  
**Login as:** `pharmacy@example.com`

1. View orders created when customers accept your offers.
2. **Mark payment received** — customer sees `payment_completed` notification.
3. After payment, **Mark delivered** — customer can leave feedback.

Works together with customer flows 7 and 8.

---

## Full end-to-end scenarios

### Scenario A — Medicine request (both roles)

| Step | Role | Action |
|------|------|--------|
| 1 | Customer (Jane) | Find Medicines → upload → OCR → submit RFQ |
| 2 | Pharmacy | Quotes → open request → submit offer |
| 3 | Customer (Jane) | Quotes → compare → accept offer |
| 4 | Pharmacy | Orders → mark payment received → mark delivered |
| 5 | Customer (Jane) | Orders → open order → leave feedback or raise complaint |

### Scenario B — Telehealth visit

| Step | Role | Action |
|------|------|--------|
| 1 | Customer (Jane) | Questionnaire → select provider → chat opens |
| 2 | Provider (no login) | `/telehealth/provider` → schedule visit |
| 3 | Customer (Jane) | Chat → see scheduled message → click Visit |
| 4 | Customer (Jane) | Submit telehealth feedback when visit completes |

### Scenario C — New pharmacy onboarding

| Step | Action |
|------|--------|
| 1 | Signup → Pharmacy tab → fill all fields |
| 2 | Subscription → choose Starter / Professional / Enterprise |
| 3 | Dashboard → view stats and incoming requests |
| 4 | Optional: buy extra credits on subscription page |

### Scenario D — Quick demo with seed data (Jane, ~2 min)

| Step | URL / action |
|------|----------------|
| 1 | Login as `jane@example.com` |
| 2 | Quotes → `rfq-seed-002` → accept cheapest offer |
| 3 | Orders → see new order OR open `order-001` for feedback demo |
| 4 | Notifications → see unread offer notification |
| 5 | Telehealth → Chats → `chat-001` |
| 6 | New tab: `/telehealth/provider` → schedule visit → refresh chat |

---

## Tips

- **Clear localStorage** to reset all mock data back to seed JSON: DevTools → Application → Local Storage → clear `medimatch_*` keys, then refresh.
- **Two browsers / incognito tabs** work well for customer + pharmacy + provider portal at the same time.
- **Sidebar navigation** is available on all customer and pharmacy pages after login.
- Data persists across page refreshes via `localStorage`.
