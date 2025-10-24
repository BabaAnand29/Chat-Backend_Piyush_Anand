# REAL-TIME CHAT API — Real-Time Chat Backend (FastAPI + Firebase)

## 🧩 Overview

**REAL-TIME CHAT API** is a real-time chat backend built using **FastAPI** and **Firebase Realtime Database**.  
It enables secure, event-driven communication where users authenticate through Firebase Auth, exchange messages instantly via WebSockets, and benefit from features like typing indicators and read receipts.

This project was developed as part of an internship assignment focused on evaluating real-time data handling, authentication, and event-driven API design.

---

## 🚀 Features

| **Category** | **Feature** | **Status** |
|---------------|-------------|-------------|
| Authentication | Firebase Auth — user register/login and JWT verification for protected routes | ✅ |
| Chat Rooms | Create and list chat rooms | ✅ |
| Messaging | Send and list messages (stored in Firebase RTDB) | ✅ |
| Real-Time Updates | WebSocket (`/ws/rooms/{room_id}`) for instant message broadcasting | ✅ |
| Read Receipts | Track and acknowledge message reads | ✅ |
| Typing Indicators | Real-time “user is typing” notifications | ✅ (bonus) |
| Secure Access | All endpoints protected by Firebase JWT verification | ✅ |
| Documentation | Swagger UI and Postman Collection included | ✅ |

---

## ⚙️ Tech Stack

- **FastAPI** – Asynchronous Python framework for building high-performance APIs  
- **Uvicorn** – ASGI server to run the FastAPI backend  
- **Firebase Authentication** – Secure user authentication and JWT verification  
- **Firebase Realtime Database** – Real-time message storage and synchronization  
- **Firebase Cloud Functions** – Event-driven triggers for backend automation  
- **WebSocket** – Real-time communication for chat and typing indicators  
- **Swagger UI** – Interactive API documentation generated automatically   

---

## 🛠️ Setup & Installation

### 1️⃣ Clone Repository
```bash
git clone https://github.com/<your-username>/norman-main.git
cd norman-main
```

### 2️⃣ Create Virtual Environment
```bash
python -m venv .venv
.\.venv\Scripts\activate       # For Windows
# source .venv/bin/activate    # For Mac/Linux
```

### 3️⃣ Install Dependencies
```bash
pip install -r requirements.txt
```

### 4️⃣ Configure Firebase
- Create a **Firebase project**.  
- Enable **Email/Password Authentication** under **Authentication → Sign-in method**.  
- Create a **Realtime Database** in test mode for local testing.  
- Download your `serviceAccountKey.json` from:  
  `Project Settings → Service accounts → Generate new private key`  
  Place it at:
  ```
  Secrets/serviceAccountKey.json
  ```

Add your Firebase configuration and other required details/variables to your `.env` file:
```
FIREBASE_DB_URL = "https://<your-project-id>.firebaseio.com/"
```

### 5️⃣ Run the Server
```bash
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

### 6️⃣ Access API Documentation  
Open your browser and visit:  
👉 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

---

## 🔐 Authentication (Firebase ID Token)

Use Firebase Authentication to log in and obtain an **idToken** for authorized API access.

Example (Postman HTTP Request):

**POST**
```
https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=<YOUR_WEB_API_KEY>
```

**Body (JSON):**
```json
{
  "email": "your_email@example.com",
  "password": "your_password",
  "returnSecureToken": true
}
```

Use the returned `idToken` as:
```
Authorization: Bearer <idToken>
```
for REST requests, 
or 
append it as  
```
?token=<idToken>
```
in WebSocket URLs.

---

## 🔗 REST API — Quick Setup & Testing

### Prerequisites
- Server running:
  ```bash
  uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
  ```
- A fresh Firebase ID token (expires ~60 min).  
  Get it via:
  ```bash
  POST https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=<YOUR_WEB_API_KEY>
  ```
  **Body:**
  ```json
  { "email": "you@example.com", "password": "your_password", "returnSecureToken": true }
  ```

---

### ✅ Test via Swagger (localhost)
Visit: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)  
Click **Authorize** → paste:
```
Bearer <YOUR_ID_TOKEN>
```

| **Action** | **Method** | **URL Pattern (Swagger)** | **Notes** |
|-------------|-------------|---------------------------|------------|
| Create room | POST | `/rooms/` | Body: `{"name":"General"}` |
| List rooms | GET | `/rooms/` | — |
| Send message | POST | `/{room_id}/messages` | Body: `{"text":"Hello"}` |
| List messages | GET | `/{room_id}/messages?limit=50` | Adjust limit as needed |

---

### ✅ Test via Postman (localhost)

#### A) Collection Setup
- Open collection → **Auth tab** → Bearer Token = your `idToken`.  
- Variables:
  ```
  base_url = http://127.0.0.1:8000
  room_id = <your real room id>
  ```

#### B) Requests (included in collection)

| **Name** | **Method** | **URL Template** |
|-----------|-------------|------------------|
| Create room | POST | `{{base_url}}/rooms/` |
| List rooms | GET | `{{base_url}}/rooms/` |
| Send message | POST | `{{base_url}}/{{room_id}}/messages` |
| List messages | GET | `{{base_url}}/{{room_id}}/messages` |

Send message → Body: raw JSON
```json
{ "text": "Hello from Piyush Anand" }
```

If:
- `401` → Refresh `idToken`
- `404` → Fix `room_id`
- `200` → Success! WS clients receive the broadcast instantly.

---

### 🔎 Example Direct URLs (localhost)

| **Action** | **Method** | **URL** |
|-------------|-------------|----------|
| Create room | POST | `http://127.0.0.1:8000/rooms/` |
| List rooms | GET | `http://127.0.0.1:8000/rooms/` |
| Send message | POST | `http://127.0.0.1:8000/<ROOM_ID>/messages` |
| List messages | GET | `http://127.0.0.1:8000/<ROOM_ID>/messages?limit=50` |

**Headers:**
```
Authorization: Bearer <YOUR_ID_TOKEN>
Content-Type: application/json
```

**Example POST body:**
```json
{ "text": "Hello" }
```

---

## 📦 Deliverable (Postman)

The REST endpoints collection required by the assignment is included in this repo:  
```
Postman/FastAPI Realtime Chat (REST).postman_collection.json
```

---

## 🔌 WebSocket Usage (Postman)

### Prerequisites
- Server running  
- Existing `room_id` in Firebase  
- Fresh Firebase `idToken` tokens expire (~60 minutes).

### 1️⃣ Get a fresh Firebase ID token
POST:
```
https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=<YOUR_WEB_API_KEY>
```
Body -> raw -> JSON:
```json
{ "email": "your_email@example.com", "password": "your_password", "returnSecureToken": true }
```
Click Send → copy idToken from the response.

---

### 2️⃣ Connect Two WebSocket Clients

We’ll use two WS tabs connected to the same room.


**Tab A(first client)** :

1.	Postman → New → WebSocket Request.

2.	In the URL box, paste:
```
ws://127.0.0.1:8000/ws/rooms/<ROOM_ID>?token=<YOUR_ID_TOKEN>
```
3.	Click Connect → status should show Connected.

**Tab B(second client)** :

Do same as Tab A

Both should show **Connected**.

If either tab shows 401 in the Handshake pane → your token is expired/invalid.
If 404 → the room doesn’t exist; create the room or check the id.

---

### 3️⃣ Verify new Message Broadcast over WebSocket

Now we’ll send a message via REST and watch both WS tabs receive it instantly

In Postman, open a new HTTP tab.

Send REST message:
```
POST http://127.0.0.1:8000/<ROOM_ID>/messages
```
Headers:
```
Authorization: Bearer <YOUR_ID_TOKEN>
Content-Type: application/json
```
Body -> raw -> JSON:
```json
{ "text": "Hello from Piyush Anand" }
```

Both WS tabs should receive:
```json
{
  "type": "message",
  "message_id": "...",
  "sender_id": "...",
  "text": "Hello from Piyush Anand",
  "created_at": 1761327695488
}
```
If the REST call is 200 but WS shows nothing, check the Troubleshooting list at the end.
---

## 💬 Typing Indicator — Test Plan

The typing feature sends lightweight WS frames so other users in the same room see “user is typing…”.

**A) Start typing**
1.	Go to WebSocket Tab A (connected earlier).
2.	In the Message box, paste:
```json
{ "type": "typing", "value": true }
```
3.	Click Send.

Expected:
o	Tab A receives:
```json
{ "type": "typing_ack", "value": true }
```

	Tab B receives:
```json
{ "type": "typing", "user_id": "<TabA_UID>", "value": true }
```

**Stop typing (Tab A → Tab B should clear it)**

1.	In WebSocket Tab A, send:
```json
{ "type": "typing", "value": false }
```

**Expected**

Tab A receives:
```json
{ "type": "typing_ack", "value": false }
```

Tab B receives:
```json
{ "type": "typing", "user_id": "<TabA_UID>", "value": false }
```

Disconnect cleanup:

1.	In WebSocket Tab A, click the Disconnect icon 

2.	Expected in Tab B:

```json
{ "type": "typing", "user_id": "<UID>", "value": false }
```
(This ensures the UI cleans up “typing…” if a user closes their tab.)

Optional: verify DB state (if you store typing in RTDB)
In Firebase Realtime Database:

rooms/<ROOM_ID>/typing/<TabA_UID> = true | false

It should flip to true on typing start and false on stop/disconnect.


---

## 🏷️ Read Receipts (Optional)

If your WS supports read receipts via read_upto:
1. In WebSocket Tab A, send:
```json
{ "type": "read_upto", "created_at_ms": 9999999999999 }
```
2. Expected in Tab A:
```json
{ "type": "read_ack", "upto": 9999999999999 }
```
In Firebase, messages at or before that created_at should show:
```
rooms/<ROOM_ID>/messages/<message_id>/read_by/<TabA_UID> = true
```

---

## 🧰 Troubleshooting

| **Issue** | **Fix** |
|-------------|---------|
| 401 | Token expired — get new ID token |
| 404 | Room doesn’t exist — create via POST `/rooms` |
| REST 200 but WS silent | Ensure same `room_id` + single server instance |
| Swagger vs Postman mismatch | Copy full Request URL from `/docs` |
| WS connected but no data | Try sending `{ "type": "typing", "value": true }` |

---

## 📁 Postman Collection

Exportable Postman collection included at:  
```
Postman/FastAPI Realtime Chat (REST).postman_collection.json
```

---

## 👨‍💻 Author

**Piyush Anand**    
📧 piyush200anand@gmail.com  

---

## 🏁 Summary

The **Chat-API backend** demonstrates a secure and efficient real-time chat system using **FastAPI** and **Firebase**.  
It fulfills all mandatory and bonus requirements of the internship assignment, including:

- Firebase authentication & JWT-based security  
- Real-time message broadcasting through WebSockets  
- Persistent storage in Firebase RTDB  
- Typing indicators & read receipts  
- Clear documentation & Postman support  

This project showcases clean architecture, production-ready structure, and professional documentation suitable for corporate evaluation.
