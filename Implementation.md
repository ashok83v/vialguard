Build a "Single-File Web App" using **Tailwind CSS** and **Vanilla JavaScript**. This is for a professional-looking demo by simply opening a file in a browser.

---

## 📄 Part 1: Product Requirements Document (PRD)

**Project Name:** VialGuard POC

**Objective:** Provide Lab Technicians with a high-visibility triage dashboard to prevent biological sample loss due to cold-chain breaches during logistics check-in.

### 1. User Persona: The Lab Technician

* **Pain Point:** Receiving 20+ boxes at once and not knowing which contains sensitive mRNA (4-hour stability) vs. stable reagents (24-hour stability).
* **Goal:** A "red-light/green-light" system to prioritize unpacking.

### 2. Core Features (The "Must-Haves")

* **Risk Engine:** A background timer that calculates "Time Remaining" before a sample spoils based on its arrival timestamp.
* **Visual Priority Alerts:** * **Critical (<1hr):** Red pulsing background.
* **Warning (<3hrs):** Yellow background.
* **Stable:** Green background.


* **Inventory Action:** A "Store in Freezer" button to simulate successful intake and remove the item from the active risk queue.

### 3. Success Metrics for Demo

* **Zero Latency:** Immediate UI updates.
* **Clarity:** A non-technical stakeholder should immediately understand which box to grab first.

---

## 🛠️ Part 2: Implementation Plan (The "Quick Build")

Since we are avoiding a database, we will use **Browser LocalStorage** to mimic a database. This allows the data to persist even if you refresh the page during the demo.

### Step 1: Data Schema

We'll use a JSON array of objects:

```json
{
  "id": "V-1001",
  "name": "Stem Cell Culture",
  "receivedAt": "2024-05-20T09:00:00",
  "stabilityLimit": 4, // hours
  "status": "In Transit"
}

```

### Step 2: The Tech Stack

* **UI:** HTML5 + Tailwind CSS (via CDN).
* **Logic:** Vanilla JavaScript (ES6+).
* **Icons:** Heroicons (via CDN).

---

## 💻 Part 3: The Code (Single-File Prototype)

Copy this code into a file named `index.html` and open it in any browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>VialGuard | Lab Logistics POC</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @keyframes pulse-red { 0%, 100% { background-color: #fee2e2; } 50% { background-color: #fca5a5; } }
        .critical-alert { animation: pulse-red 2s infinite; }
    </style>
</head>
<body class="bg-gray-50 font-sans">

    <nav class="bg-slate-900 text-white p-4 shadow-lg">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-xl font-bold tracking-tight">🧪 VIAL<span class="text-blue-400">GUARD</span></h1>
            <span class="bg-blue-900 text-xs px-3 py-1 rounded-full border border-blue-400">POC MODE</span>
        </div>
    </nav>

    <main class="container mx-auto py-8 px-4">
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8 text-center">
            <div class="bg-white p-6 rounded-xl shadow-sm border border-gray-200">
                <p class="text-gray-500 text-sm font-medium uppercase">Active Shipments</p>
                <p id="total-count" class="text-3xl font-bold">0</p>
            </div>
            <div class="bg-white p-6 rounded-xl shadow-sm border border-gray-200">
                <p class="text-gray-500 text-sm font-medium uppercase text-red-600">Critical Risk</p>
                <p id="risk-count" class="text-3xl font-bold text-red-600">0</p>
            </div>
            <div class="bg-white p-6 rounded-xl shadow-sm border border-gray-200">
                <button onclick="resetDemo()" class="text-blue-600 hover:underline text-sm font-medium mt-2 uppercase">Reset Demo Data</button>
            </div>
        </div>

        <div class="bg-white rounded-xl shadow-md overflow-hidden border border-gray-200">
            <div class="p-4 border-b bg-gray-100 flex justify-between items-center">
                <h2 class="font-bold text-gray-700 uppercase tracking-wider">Incoming Triage Queue</h2>
                <span class="text-xs text-gray-400 italic">Auto-refreshing every 30s</span>
            </div>
            <table class="w-full text-left border-collapse">
                <thead>
                    <tr class="bg-gray-50 text-gray-600 text-sm">
                        <th class="p-4">Item ID</th>
                        <th class="p-4">Sample Type</th>
                        <th class="p-4">Received</th>
                        <th class="p-4">Time Remaining</th>
                        <th class="p-4 text-right">Action</th>
                    </tr>
                </thead>
                <tbody id="shipment-table">
                    </tbody>
            </table>
        </div>
    </main>

    <script>
        // Placeholder Data (Simulating a database)
        const initialData = [
            { id: "VG-202", name: "mRNA COVID Vaccine", received: new Date(Date.now() - 12600000), limit: 4 }, // 3.5 hrs ago
            { id: "VG-505", name: "Stem Cell Batch B", received: new Date(Date.now() - 3600000), limit: 2 }, // 1 hr ago
            { id: "VG-881", name: "Taq Polymerase", received: new Date(Date.now() - 7200000), limit: 24 }, // 2 hrs ago
            { id: "VG-102", name: "Generic Buffer Sol.", received: new Date(), limit: 48 } // Just now
        ];

        function getShipments() {
            const stored = localStorage.getItem('vialguard_data');
            return stored ? JSON.parse(stored) : initialData;
        }

        function render() {
            const shipments = getShipments();
            const table = document.getElementById('shipment-table');
            const totalCountEl = document.getElementById('total-count');
            const riskCountEl = document.getElementById('risk-count');
            
            table.innerHTML = "";
            let riskCount = 0;

            shipments.forEach((item, index) => {
                const elapsed = (new Date() - new Date(item.received)) / 3600000;
                const remaining = item.limit - elapsed;
                
                let rowClass = "";
                let statusBadge = "";
                
                if (remaining < 1) {
                    rowClass = "critical-alert";
                    statusBadge = "🔴 CRITICAL";
                    riskCount++;
                } else if (remaining < 3) {
                    rowClass = "bg-yellow-50";
                    statusBadge = "🟡 WARNING";
                } else {
                    statusBadge = "🟢 STABLE";
                }

                table.innerHTML += `
                    <tr class="border-b hover:bg-gray-50 transition-colors ${rowClass}">
                        <td class="p-4 font-mono text-sm">${item.id}</td>
                        <td class="p-4 font-semibold text-gray-800">${item.name}</td>
                        <td class="p-4 text-sm text-gray-500">${new Date(item.received).toLocaleTimeString()}</td>
                        <td class="p-4 font-bold text-sm">${remaining.toFixed(1)} hrs left</td>
                        <td class="p-4 text-right">
                            <button onclick="storeVial(${index})" class="bg-slate-800 text-white px-4 py-2 rounded shadow hover:bg-blue-600 text-xs font-bold transition-all">
                                STORE IN FREEZER
                            </button>
                        </td>
                    </tr>
                `;
            });

            totalCountEl.innerText = shipments.length;
            riskCountEl.innerText = riskCount;
        }

        function storeVial(index) {
            let shipments = getShipments();
            shipments.splice(index, 1);
            localStorage.setItem('vialguard_data', JSON.stringify(shipments));
            render();
        }

        function resetDemo() {
            localStorage.setItem('vialguard_data', JSON.stringify(initialData));
            render();
        }

        render();
        setInterval(render, 30000); // Re-calculate risk every 30 seconds
    </script>
</body>
</html>

```

---

## 🎤 Part 4: Demo Guide

### The Pitch (The "Why")

"Today, our lab technicians receive high-value biological samples with varying temperature sensitivities. Mis-prioritizing one mRNA box for an hour can lead to a **$10,000 product loss**. VialGuard solves this by translating 'Time of Receipt' into 'Actionable Risk'."

### The Demo Steps

1. **The Arrival:** Show the dashboard with 4 items. Point out the **VG-505 (Stem Cell)** item—it’s highlighted in red/yellow because its stability window is closing.
2. **The Risk Calculation:** Explain that the app isn't just a list; it's calculating "Time Remaining" in real-time.
3. **The Triage:** Click the **"Store in Freezer"** button for the most critical item. Show how the "Risk Count" in the top card updates immediately.
4. **The Audit Trail (Mental Model):** Explain that in a full version, this click would log the technician's ID and the freezer temperature to an FDA-compliant database.

**Add a feature to this POC that lets you "Add New Shipment" via a popup form to show how the list grows?**