 📖 Developer's Note & Comprehensive Guide: EV Mobility Platform 🚗⚡

Welcome to the repository. If you are a recruiter, a fellow developer, or just someone curious about this project, this document is written to give you a clear, realistic look under the hood. It’s not just a list of installation commands—it’s an explanation of **what** this is, **why** it was built, and **how** the code actually works.

## 🎯 The Big Picture (What is this?)

This is a production-grade, investor-ready web application built to tackle "range anxiety" for Electric Vehicle (EV) drivers. 

Instead of just presenting a static map, this platform provides an interactive Google Maps experience where users can:
1.  **Discover** nearby EV charging stations.
2.  **Filter** them based on their specific car's needs (Connector type, Charging Speed).
3.  **Book** a slot in real-time, preventing anyone else from taking it.

It demonstrates complex frontend state management and real-time backend communication without requiring a massive, heavy database setup.

---

## 🧠 How It Works: The Architecture & Code

To understand this project, you need to look at three main pillars: the Frontend UI, the State Management, and the Real-Time Mock Backend.

### 1. The Frontend (Next.js 15 & Tailwind CSS)
The app uses the modern Next.js App Router. It is designed to be "desktop-first" but fully responsive. We use **Tailwind CSS** for utility-first styling and **Framer Motion** to ensure the UI feels premium with smooth 60fps animations (like the glassmorphism effects on the station cards).

### 2. Global State Management (Zustand)
Instead of passing props down a dozen levels or setting up a heavy Redux boilerplate, this app uses **Zustand** for lightweight, fast state management. 

For example, when a user selects a filter (like "Tesla connectors only"), the `stationsStore` updates and immediately reflects on the Google Map components. 

*Here is a conceptual look at how our Zustand stores keep things clean:*
```typescript
// store/stationsStore.ts (Conceptual Example)
import { create } from 'zustand';

interface StationState {
  stations: Station[];
  activeFilters: string[];
  setFilters: (filters: string[]) => void;
}

export const useStationStore = create<StationState>((set) => ({
  stations: [], // Loaded from our mock backend
  activeFilters: [],
  setFilters: (filters) => set({ activeFilters: filters }),
}));
```

### 3. The Real-Time Booking Engine (WebSockets & Express)
This is where the app shines. Instead of a simple visual toggle, the app uses an Express.js server running **Socket.IO** to handle bookings realistically.

When a user books a charger:
1.  The frontend sends a WebSocket event to the server.
2.  The server locks that specific slot and assigns a 5-minute countdown timer.
3.  The server broadcasts this "Booked" status to *everyone* currently looking at the app.
4.  If the 5 minutes run out, the server automatically releases the slot and updates all connected clients.

*Here is how the frontend listens for those real-time updates:*
```javascript
// A simplified view of the Socket connection in the frontend
import { io } from 'socket.io-client';

const socket = io(process.env.NEXT_PUBLIC_API_URL);

// Listening for real-time station updates from the Express backend
socket.on('stationStatusUpdate', (updatedStation) => {
  // Update the UI immediately so no two users can book the same slot
  updateStationInUI(updatedStation);
});
```

---

## 📂 Project Structure (Where to find things)

If you are diving into the code, here is your map:

* **`/app`**: The core Next.js routing. This is where the main pages and global CSS live.
* **`/components`**: Reusable UI pieces.
    * `/Map`: Everything related to the Google Maps API (markers, clusters, controls).
    * `/Stations`: The visual cards and details drawers for the EV chargers.
* **`/store`**: All Zustand state files (`mapStore.ts`, `bookingsStore.ts`, etc.). This is the brain of the frontend.
* **`/server`**: The Express.js backend. Look in `server/index.js` to see the Socket.IO setup, and `server/services` for the mock booking logic.

---

## 🚀 How to Run This Locally

Because this app features a real-time booking engine, you need to run both the backend server and the frontend Next.js app simultaneously. It requires a bit of work to set up, but it's worth it.

**Step 1: Get the Code & Install Packages**
```bash
git clone <repository-url>
cd ec-wevapp
npm install
```

**Step 2: Environment Variables**
You will need a Google Maps API Key. Create a `.env.local` file in the root directory:
```env
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=your_actual_api_key_here
NEXT_PUBLIC_API_URL=http://localhost:3001
```

**Step 3: Boot up the Backend (Terminal 1)**
This starts the Express server and the WebSocket connections on port 3001.
```bash
npm run server
```

**Step 4: Boot up the Frontend (Terminal 2)**
Open a fresh terminal window and start the Next.js app on port 3000.
```bash
npm run dev
```

Finally, open `link` in your browser.

-
or the WebSocket real-time booking to someone reviewing your code?
