ParkingSpot

ParkingSpot is a smart parking and EV charging reservation demo. Users can find parking facilities, filter by vehicle type and location, view available slots, and create bookings with QR passes.

Features

Search parking facilities in Chennai and other sample cities.

Filter car, bike, and EV charging spaces.

View facility details, prices, available slots, and locations on a map.

Register, log in, and manage bookings.

Use QR booking passes and view owner and administrator dashboards.

Get parking suggestions from ParkBot. Gemini integration is optional.

Tech Stack

React, TypeScript, Vite, Tailwind CSS, Node.js, Express, and Socket.IO. The server can connect to MongoDB, but the current facility, user, slot, and booking records are stored in memory.

Run Locally

Install Node.js 22.16 or newer, then open the project folder in VS Code. In the terminal, run:

npm ci
npm run dev

Open http://localhost:3000. To check the backend, visit http://localhost:3000/api/health and http://localhost:3000/api/facilities?city=Chennai.

The included sample facilities load without a MongoDB connection or Gemini API key. To enable Gemini-based ParkBot responses, put your own GEMINI_API_KEY in a local .env file. Do not commit .env or other secrets.

Deployment

The Vite frontend can be deployed to Vercel, and the Express backend needs a running Node.js service such as Render. Set VITE_API_ORIGIN on Vercel to the backend origin, for example https://your-backend.onrender.com (without /api). Redeploy the frontend after setting it.

For backend settings and deployment steps, see VERCEL_DEPLOY_GUIDE.md.

Current Limitations

Listings and availability are sample data. Users, facilities, slots, and bookings currently live in server memory, so changes can disappear when the backend restarts. Connecting MONGODB_URI alone does not make these records persistent. Payment flows include demo behavior; do not use the app for real payments or live reservations without persistent storage and server-side payment verification.
