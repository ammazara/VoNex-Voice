# VoNex System Architecture

### Overview
VoNex is designed as a scalable, API-driven communication platform that integrates **VoIP, messaging, and cloud storage** into a unified system.  
It leverages reliable cloud telephony API (Twilio) and modern web technologies to replicate and localize the Google Voice experience for Nigeria.

---

## Frontend
**Technology:** React.js (Progressive Web App)  
**Purpose:** Provide a fast, mobile-first user interface for calls, messages, and voicemails.  

**Core Modules:**
- Dashboard (Unified inbox for calls, SMS, and voicemails)
- Call & Message screens
- Voicemail playback and transcription viewer
- User settings (linked devices, forwarding rules)
- Admin panel (usage analytics, billing)

**Communication:** All frontend actions communicate with the backend over secure RESTful APIs.

---

## Backend
**Technology:** Node.js + Express.js  
**Purpose:** Core logic and orchestration layer between the UI, APIs, and database.  

**Key Services:**
1. **Authentication Service**
   - Firebase Auth / OAuth2 for secure login  
   - Supports Google Sign-in and email-based access  

2. **Call & SMS Service**
   - Routes calls and SMS via Twilio 
   - Handles call recording, forwarding, and real-time logs  

3. **Voicemail Service**
   - Stores and transcribes audio using Speech-to-Text APIs  
   - Sends email notifications with transcription  

4. **User Profile & Billing Service**
   - Manages subscriptions, linked devices, and call credits  

**API Structure:**
- `/api/auth/*` – Login, signup, token refresh  
- `/api/calls/*` – Call routing, history  
- `/api/messages/*` – SMS send/receive  
- `/api/voicemails/*` – Upload, store, transcribe  
- `/api/profile/*` – User info and billing  

---

## Database
**Technology:** MongoDB Atlas  

**Collections:**
- `users` – Profile, auth tokens, linked devices  
- `calls` – Logs, timestamps, durations  
- `messages` – SMS records and delivery reports  
- `voicemails` – Audio URLs, transcription text  
- `subscriptions` – Plan details, billing history  

---

## External Integrations
- **Twilio:** Handles voice and SMS infrastructure  
- **Firebase Authentication:** Secure user login and verification  
- **AWS S3:** Stores voicemail audio files  
- **Google Speech-to-Text API:** Converts voicemail to readable text  

---

## Security
- HTTPS-only communication  
- OAuth2 token-based authentication  
- AES-256 encryption for sensitive data  
- Regular database backups on MongoDB Atlas  

---

## Communication Flow

[User Interface - React PWA]
       ↓ HTTPS
[Backend - Node.js + Express]
       ↔ [MongoDB Atlas Database]
       ↕
  [Twilio API]
       ↕
 [User’s Local Network / Device]


 ## Data Flow

### 1️⃣ User Authentication
1. User opens the **VoNex** web app and logs in using **email or Google Sign-In**.  
2. **Frontend** sends credentials securely to the **backend** via `HTTPS`.  
3. **Backend** verifies credentials through **Firebase Auth** and returns a **JWT token**.  
4. Token is stored locally (in session storage) to allow secure API access for subsequent actions.  

---

### 2️⃣ Making a Call
1. User initiates a **call** from the dashboard interface.  
2. **Backend** receives the call request and triggers **Twilio** to create a voice session.  
3. API connects both caller and receiver, while the backend logs **call metadata** (caller ID, receiver, timestamp, duration).  
4. **MongoDB** stores call records.  
5. **Frontend** fetches updated data via a WebSocket or periodic refresh to update the user’s dashboard in real time.  

---

### 3️⃣ Receiving SMS
1. Incoming SMS hits a configured **webhook endpoint** provided by Twilio.  
2. **Backend** processes message payload (sender, receiver, content, timestamp).  
3. Message data is stored in **MongoDB** under the user’s message collection.  
4. **Frontend** retrieves new messages from `/api/messages` endpoint at intervals or via socket events.  
5. The new message is displayed instantly in the **unified inbox** interface.  

---

### 4️⃣ Voicemail & Transcription
1. When a user misses a call, the system routes it to **voicemail** via Twilio API.  
2. The **audio file** is saved automatically to **AWS S3**.  
3. The **S3 file URL** is sent to the backend.  
4. Backend triggers the **Google Speech-to-Text API** for transcription.  
5. Both the **transcription** and **audio link** are stored in **MongoDB**.  
6. Backend notifies the user via **email** and updates the voicemail section of their dashboard.  

---

### 5️⃣ Analytics & Billing
1. **Backend** periodically aggregates call and SMS logs from MongoDB.  
2. Computes usage statistics such as total minutes, messages sent, and data trends.  
3. Tracks **billing events** and API usage per user account.  
4. **Frontend** visualizes analytics in the dashboard using charts and tables.  
5. Admin dashboard monitors API performance and system uptime.  

---



## Technical Feasibility

### Stack Efficiency
- **Node.js + Express:** Lightweight, event-driven backend ideal for handling real-time calls, SMS, and API requests.  
- **MongoDB Atlas:** Flexible NoSQL database suited for storing unstructured telecom data such as call logs, messages, and voicemails.  
- **React.js (PWA):** Delivers an app-like experience in browsers, reducing dependency on native apps during MVP phase.  
- **Twilio API:** Provides telecom-level performance without building local infrastructure from scratch.

---

### API Reliability
- **Twilio** offers proven uptime (>99%) and is already operational in Nigeria.  
- API supports essential services; number provisioning, voice routing, SMS delivery, and webhooks, ensuring real-time communication.  
- Failover and redundancy built into this API minimize downtime during high traffic.  
- Local carrier integration ensures call stability and number verification across MTN, Glo, Airtel, and 9mobile networks.

---

### Cloud Infrastructure
- **AWS EC2:** Reliable virtual server environment for hosting backend services.  
- **AWS S3:** Scalable and secure storage for voicemail audio files and logs.  
- **MongoDB Atlas:** Cloud-hosted database with automatic backups and clustering.  
- **Firebase Auth:** Serverless authentication layer with Google integration, token refresh, and minimal maintenance overhead.  
- Entire system can auto-scale based on traffic spikes without manual provisioning.

---

### Security and Compliance
- **HTTPS + TLS Encryption:** Ensures secure data transfer between client, server, and APIs.  
- **OAuth2 Authentication:** Protects API endpoints and user sessions with industry-standard authorization.  
- **AES-256 Encryption:** Encrypts sensitive user information and stored credentials.  
- **NDPR Compliance:** System architecture aligns with Nigeria Data Protection Regulation (NDPR) for user privacy.  
- **Cloud Backups:** Automated daily backups via MongoDB Atlas safeguard against data loss.

---

### Development Feasibility
- The chosen stack (Node.js, React, MongoDB) is **open-source, well-documented**, and widely adopted by Nigerian developers.  
- Strong local developer communities and affordable hosting options make the stack easy to maintain.  
- Minimal infrastructure cost for MVP can be deployed using **AWS Free Tier** or **Render.com** for early testing.  
- Modular backend structure allows quick integration of third-party APIs and services as the product evolves.

---

### Scalability Path
- **Phase 1:** Launch in Nigeria using Twilio for local numbers.  
- **Phase 2:** Expand to other African markets (Eg. Ghana, Kenya) by swapping the telephony provider while maintaining the same backend.  
- **Phase 3:** Integrate mobile payment options for credit purchases and subscriptions (via Paystack or Flutterwave).  
- **Phase 4:** Introduce mobile apps (React Native) with push notifications and offline caching.  
- **Phase 5:** Add AI-driven voicemail sorting and translation into local languages (Hausa, Yoruba, Igbo).  

link to user-flow on google drive
https://drive.google.com/drive/folders/1VToUR0gVQjevDbsVLpSOH9VB2ZdLB_S1?usp=sharing
 

