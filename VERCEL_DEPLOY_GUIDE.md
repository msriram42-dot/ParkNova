# ParkingSpot: Vercel frontend + Render Express backend

## Why the original Vercel upload displayed zero facilities

1. `CommuterDashboard.tsx` used `Promise.all([getFacilities(), getMyBookings()])`.
   `getMyBookings()` is a protected endpoint that returns 401 for visitors; the combined
   promise discarded a successful facilities response.
2. A default Vite frontend deployment on Vercel does not automatically run
   the `server.ts` Express/Socket.IO server as a persistent backend. Without a
   running backend, `/api/facilities` and `/api/health` cannot return JSON.
3. The existing MongoDB connection helper does NOT persist `parkingStore` data: user,
   facility, booking, and slot records currently live in JavaScript Maps. A restart
   resets them to sample data. Do not use this as a live payment/booking production system.

## What this fixed ZIP includes

- Public facilities load independently of authenticated booking history.
- Request errors are shown as errors, not mislabeled as zero search results.
- A request ID avoids overwriting new search results with stale fetches.
- A single configurable `VITE_API_ORIGIN` targets a separately hosted Express API
  and Socket.IO backend from Vercel.
- CORS accepts the configured Vercel frontend origin.
- Render Blueprint configuration (`render.yaml`), Vercel SPA config (`vercel.json`)
  and sample environment variables (NO actual secrets).
- Production startup checks for a unique JWT_SECRET and ADMIN_PASSWORD.

## Step 1: Put this FIXED project on GitHub

Replace your repository files with the contents of this ZIP, then commit and push.
Do NOT upload a local `.env`, credentials, or your MongoDB URI to GitHub.

## Step 2: Deploy Express backend to Render

1. Visit https://dashboard.render.com/ and create a new Web Service from your repository,
   or choose New > Blueprint to use `render.yaml`.
2. Runtime: Node; Node version 22.16.0. Build command:
       npm install --include=dev && npm run build
   Start command:
       npm start
3. Set backend environment variables:
   - NODE_ENV = production
   - JWT_SECRET = freshly generated random secret, at least 32 bytes
   - ADMIN_EMAIL = your non-public administrator email
   - ADMIN_PASSWORD = unique long, strong password
   - FRONTEND_URL = https://YOUR-PROJECT.vercel.app
   - GEMINI_API_KEY = your own key (if using ParkBot)
   - MONGODB_URI = optional in CURRENT code; connection exists but the app's
     records are still in memory; this alone does not provide persistence.
4. Wait for successful deployment, then visit:
       https://YOUR-RENDER-SERVICE.onrender.com/api/health
   Expected: JSON with `status: ok`.
   Then check `/api/facilities` and confirm you receive `facilities`.

## Step 3: Deploy frontend to Vercel

1. Import/update the same GitHub repository in Vercel.
2. Framework Preset: Vite; Build Command: npm run build; Output Directory: dist.
3. Add environment variable:
       VITE_API_ORIGIN=https://YOUR-RENDER-SERVICE.onrender.com
   It must be the Render origin (no `/api` suffix and no trailing slash).
4. Redeploy Vercel after setting the environment variable.
5. Visit the Vercel app in a private browser window. Facilities are public; login
   is not required to view them. Booking history will load only after login.

## If the browser still shows no facilities

Open browser DevTools > Network, then reload the site and inspect the
`GET https://YOUR-RENDER-SERVICE.onrender.com/api/facilities?...` request.
- 200 with a non-empty `facilities` array: check selected city/area/category.
- 401 from `/api/bookings/my`: expected for unauthenticated users, but the fixed
  dashboard will no longer call this endpoint for visitors.
- Failed to fetch / CORS: verify Render backend URL and FRONTEND_URL exact domain.
- 404 / HTML from `/api/facilities`: a frontend page is answering an API request;
  recheck VITE_API_ORIGIN and redeploy.
- 5xx: check Render service logs.

## Important production limitations

This patch fixes the original visibility/deployment routing problem. The
server's Map-based in-memory data store is NOT MongoDB-backed persistence even
when MONGODB_URI is configured; registered users/bookings may disappear on restart.
The original project also contains seed accounts and mock-payment flows.
Do not accept real payments or live reservations until real database CRUD,
authentication review, and Razorpay server-side verification are implemented.
