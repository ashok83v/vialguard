# 🧬 VialGuard — Lab Logistics POC

> A single-file, no-install demo that shows how real-time cold-chain risk alerting could work in a clinical lab setting.

---

## What This Is

VialGuard is a **proof-of-concept triage dashboard** for lab technicians who receive biological samples (vials, batches, reagents) that must be stored within a tight temperature window. The app calculates, in real time, how long each sample has left before it degrades — and tells you which one to grab first.

To run it: **open `index.html` in any browser**. No server, no install, no internet required (except for initial font/CSS loading).

---

## How to Use It

### Step 1 — Sign In

When you open the app, you'll see a **Technician Sign-In screen**. Enter:
- **Full Name** — e.g. `Dr. Sarah Chen`
- **Badge ID** — e.g. `#4821`

This personalizes the session. Every action you take (storing a sample, adding a shipment) is recorded with your name and badge number. Click **Enter Dashboard** to proceed.

---

### Step 2 — Read the Dashboard

After signing in, you'll see three summary cards at the top:

| Card | What it shows |
|---|---|
| **Active Shipments** | Total number of samples currently in the triage queue |
| **Critical Risk** | Number of samples with less than 1 hour of stability remaining |
| **Demo Controls** | Buttons to add a new shipment or reset to the original demo data |

Below the cards is the **Incoming Triage Queue** — a table listing every sample, automatically sorted so the most urgent items are always at the top.

---

### Step 3 — Understand the Risk Colours

Each row in the table is colour-coded based on how much time the sample has left:

| Colour | Badge | Meaning |
|---|---|---|
| 🔴 **Red (pulsing)** | CRITICAL | Less than 1 hour remaining — handle this first |
| 🟡 **Yellow** | WARNING | 1 to 3 hours remaining — needs attention soon |
| 🟢 **None** | STABLE | More than 3 hours remaining — safe for now |

The **Time Remaining** column counts down live, second by second. The **Stability Limit** column shows a progress bar filling up as the sample approaches expiry.

---

### Step 4 — Store a Sample

Once a sample is physically unpacked and placed in the correct freezer, click the **STORE IN FREEZER** button on that row. The item is removed from the queue, the Active Shipments count drops by one, and the action is recorded in the Audit Trail.

---

### Step 5 — Add a New Shipment

Click the blue **+ ADD NEW SHIPMENT** button to open a form. Fill in:
- **Item ID** — a unique identifier like `VG-999`
- **Sample Type / Name** — e.g. `mRNA Batch C`
- **Stability Limit** — how many hours this sample can survive before degrading (e.g. `4`)
- **Received At** — defaults to right now; change it if the sample arrived earlier

Click **Add to Queue** to add it to the live table. It will immediately be sorted into the correct risk position.

---

### Step 6 — Review the Audit Trail

Click the **Audit Log** button in the top navigation bar. A panel slides in from the right showing a timestamped log of every action taken during the session:

- Who signed in
- Which samples were stored, and what their risk level was at the time
- When new shipments were added
- If the demo data was reset

Each entry shows the technician's name, badge ID, and the exact time of the action. This is what an FDA 21 CFR Part 11-compliant electronic record would look like in a production system.

---

### Step 7 — Switch Technician

Click the **Switch** link next to your name in the nav bar to end your session. The login screen reappears so a different technician can sign in. The queue data is preserved; only the session log is cleared.

---

### Step 8 — Reset the Demo

Click **↺ Reset Demo Data** to restore the original four sample shipments. Use this between demo runs to get back to the starting state.

---

## How the Risk Engine Works

The app doesn't use a database or a server. All calculations happen in the browser:

1. Each sample has a **received timestamp** and a **stability limit in hours**.
2. Every second, the app calculates: `Time Remaining = Stability Limit − Hours Since Arrival`
3. Depending on the result, the row gets a risk level (Critical / Warning / Stable).
4. The queue re-sorts automatically so Critical items always appear first.
5. Data is saved to **browser LocalStorage**, so it survives a page refresh.

---

## Tech Stack

| Component | Technology |
|---|---|
| UI | HTML5 + Tailwind CSS (via CDN) |
| Logic | Vanilla JavaScript (no frameworks) |
| Fonts | Inter + JetBrains Mono (Google Fonts) |
| Persistence | Browser LocalStorage |
| Build step | None — open the file directly |
