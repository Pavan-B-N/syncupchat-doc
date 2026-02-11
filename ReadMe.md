# SyncUpChat

> A real-time chat, channel, and video-conferencing platform built with a microservice architecture — supporting both **mobile** (React Native) and **web** (React + Vite) clients.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Services](#services)
  - [1. Main Server (REST API)](#1-main-server-rest-api)
  - [2. Socket Server (Real-time)](#2-socket-server-real-time)
  - [3. WebRTC Server (Video Conferencing)](#3-webrtc-server-video-conferencing)
  - [4. Mobile Client (React Native)](#4-mobile-client-react-native)
  - [5. Web Client (React + Vite)](#5-web-client-react--vite)
- [Database Schema](#database-schema)
- [API Endpoints](#api-endpoints)
- [Socket Events](#socket-events)
- [WebRTC Events](#webrtc-events)
- [Authentication Flow](#authentication-flow)
- [Environment Variables](#environment-variables)
- [Getting Started](#getting-started)
- [Deployment](#deployment)
- [Author](#author)

---

## Overview

SyncUpChat is a full-stack real-time communication platform consisting of five independent services:

| Service | Purpose | Port |
|---------|---------|------|
| **Main Server** | REST API, authentication, database | `3030` |
| **Socket Server** | Real-time messaging & events | `5000` |
| **WebRTC Server** | Video/audio conferencing via mediasoup | `3000` |
| **Mobile Client** | React Native app (Android/iOS) | — |
| **Web Client** | React SPA served by Vite | `5173` |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Clients                                  │
│  ┌──────────────────┐          ┌──────────────────┐             │
│  │  Mobile Client   │          │   Web Client     │             │
│  │  (React Native)  │          │  (React + Vite)  │             │
│  └────────┬─────────┘          └────────┬─────────┘             │
│           │                              │                      │
├───────────┼──────────────────────────────┼──────────────────────┤
│           │           Backend            │                      │
│           ▼                              ▼                      │
│  ┌────────────────┐    ┌────────────────────┐    ┌───────────┐  │
│  │  Main Server   │    │  Socket Server     │    │  WebRTC   │  │
│  │  (Express +    │    │  (Socket.IO)       │    │  Server   │  │
│  │   MongoDB)     │    │  Port: 5000        │    │(mediasoup)│  │
│  │  Port: 3030    │    │                    │    │ Port: 3000│  │
│  └───────┬────────┘    └────────────────────┘    └───────────┘  │
│          │                                                      │
│          ▼                                                      │
│  ┌────────────────┐                                             │
│  │   MongoDB      │                                             │
│  │   Database     │                                             │
│  └────────────────┘                                             │
└─────────────────────────────────────────────────────────────────┘
```

---

# Web APP Demo

<video src="./web-app-demo.mp4" controls></video>

[Watch Web App Demo Video](./web-app-demo.mp4)


# Mobile APP Demo

<video src="./mobile-app-demo.mp4" controls></video>

[Watch Mobile App Demo Video](./mobile-app-demo.mp4)

---

## Tech Stack

### Backend

| Technology | Used In |
|------------|---------|
| **Node.js** | All servers |
| **Express 5** | Main Server, WebRTC Server |
| **MongoDB + Mongoose** | Main Server (database) |
| **Socket.IO** | Socket Server, WebRTC Server |
| **mediasoup** | WebRTC Server (SFU) |
| **JWT** | Authentication (access tokens) |
| **Firebase Admin** | Push notifications |
| **Google Auth Library** | Google Sign-In verification |

### Mobile Client

| Technology | Purpose |
|------------|---------|
| **React Native 0.82** | Cross-platform mobile framework |
| **React Navigation** | Navigation (stack, bottom tabs) |
| **Axios** | HTTP requests |
| **Socket.IO Client** | Real-time messaging |
| **mediasoup-client** | WebRTC video/audio |
| **react-native-webrtc** | Native WebRTC integration |
| **Firebase Messaging** | Push notifications |
| **Lottie** | Animations |
| **React Native Vision Camera** | QR code scanning |
| **Encrypted Storage** | Secure token storage |

### Web Client

| Technology | Purpose |
|------------|---------|
| **React 19** | UI framework |
| **Vite 7** | Build tool & dev server |
| **React Router DOM** | Client-side routing |
| **Axios** | HTTP requests |
| **Socket.IO Client** | Real-time messaging |
| **mediasoup-client** | WebRTC video/audio |
| **react-hot-toast** | Toast notifications |
| **react-icons** | Icon library |
| **react-qr-code** | QR code generation |
| **Lottie React** | Animations |

---

## Repository Structure

```
Aura/
├── syncupchat/                    # Mobile client (React Native)
│   └── src/
│       ├── Screens/               # 21 screen components
│       ├── Components/            # 26 reusable components
│       ├── api/                   # API handlers & clients
│       ├── context/               # Global state (AppContext)
│       ├── mediasoup/             # WebRTC client & store
│       ├── notifications/         # Push notification setup
│       └── utils/                 # Utility functions
│
├── syncupchat-web-client/         # Web client (React + Vite)
│   └── src/
│       ├── Pages/                 # LoginPage, ChatPage, ChannelsPage, MeetPage, SettingsPage
│       ├── Components/            # Sidebar, ChatCard, ChatRoom, MeetRoom, etc.
│       ├── api/                   # axiosClient, socketClient, handlers
│       ├── context/               # AppContext (global state)
│       └── mediasoup/             # WebRTC client & store (browser)
│
├── syncupchat-mainserver/         # REST API server
│   ├── controllers/               # Business logic
│   ├── routes/                    # Express route definitions
│   ├── models/                    # Mongoose schemas
│   ├── middlewares/               # JWT verification, validators
│   └── server.js                  # Entry point
│
├── syncupchat - socket-server/    # Real-time messaging server
│   └── server.js                  # Socket.IO server
│
├── webrtc - server/               # Video conferencing server
│   └── server.js                  # mediasoup SFU server
│
└── syncupchat-doc/                # This documentation
    └── ReadMe.md
```

---

## Services

### 1. Main Server (REST API)

The central API server handling authentication, user management, chat operations, messaging, channels, and notifications.

**Entry point:** `server.js`
**Port:** `3030`
**Database:** MongoDB (local: `mongodb://0.0.0.0:27017/syncupchat`)

#### Key Features

- **Authentication:** Google Sign-In (mobile) + Email OTP login + QR code login (web)
- **JWT-protected routes** for all user/chat/message/channel operations
- **Database connection guard** — returns `503` if MongoDB is unavailable
- **CORS** — allows `localhost:3000` and `localhost:5173`
- **Push notifications** via Firebase Admin SDK

#### Route Groups

| Route | Auth | Purpose |
|-------|------|---------|
| `/auth/*` | ❌ | Sign up, sign in, OTP, QR auth |
| `/users/*` | ✅ JWT | User profile, search, explore |
| `/chats/*` | ✅ JWT | Create/fetch chats |
| `/messages/*` | ✅ JWT | Send/fetch messages, status updates |
| `/channel/*` | ✅ JWT | Create/join/fetch channels, posts |
| `/app/*` | ❌ | App version check |
| `/notifications/*` | ❌ | FCM token registration |

#### Dependencies

```
express@5, mongoose, jsonwebtoken, cors, dotenv,
firebase-admin, google-auth-library, axios, validator
```

---

### 2. Socket Server (Real-time)

Handles real-time bidirectional communication between clients for instant messaging, delivery receipts, and QR code authentication.

**Entry point:** `server.js`
**Port:** `5000`
**Transport:** WebSocket only

#### Key Features

- **Multi-device support** — a single user can have multiple socket connections (mobile + web)
- **Message delivery** — routes messages from sender to all receiver sockets
- **Delivery receipts** — `deliveredMsg` and `seenMsg` events forwarded to sender
- **QR authentication relay** — bridges QR code verification between mobile and web

#### Connection Management

```javascript
// Each user maps to a Set of socket IDs (multi-device)
connectedUsers: Map<userId, Set<socketId>>
```

#### Dependencies

```
express, socket.io, cors, dotenv
```

---

### 3. WebRTC Server (Video Conferencing)

A Selective Forwarding Unit (SFU) built on **mediasoup** that handles real-time video/audio conferencing with room management.

**Entry point:** `server.js`
**Port:** `3000`
**Transport:** WebSocket (via Socket.IO)

#### Key Features

- **Room management** — create rooms with UUIDs, join existing rooms
- **mediasoup SFU** — server-side routing of media streams (no peer-to-peer mesh)
- **Multiple participants** — each room supports multiple producers/consumers
- **Transport management** — separate send/receive WebRTC transports per peer
- **Dockerized** — includes Dockerfile for container deployment

#### WebRTC Flow

```
1. Client emits "createRoom" → receives roomId
2. Client emits "joinRoom" → receives routerRtpCapabilities + existingProducers
3. Client emits "createTransport" (×2) → send transport + receive transport
4. Client produces audio + video tracks
5. Server notifies other peers via "newProducer"
6. Other peers consume the new tracks
```

#### Dependencies

```
express@5, mediasoup, socket.io, uuid, cors, dotenv
```

---

### 4. Mobile Client (React Native)

The primary mobile app supporting Android and iOS.

**Framework:** React Native 0.82
**Navigation:** React Navigation (native stack + bottom tabs)

#### Screens

| Screen | Purpose |
|--------|---------|
| `AuthScreen` | Entry point — Google Sign-In / Email login |
| `SignUpWithEmailScreen` | New user registration with email + OTP |
| `SignInWithEmailScreen` | Email OTP login for returning users |
| `HomeScreen` | Bottom-tab navigator (Chats, Channels, SyncUp, Settings) |
| `ChatScreen` | Chat list with search |
| `ChatRoomScreen` | Individual chat with real-time messaging |
| `ChannelsScreen` | Joined + suggested channels |
| `ChannelRoomScreen` | View posts in a channel |
| `ChannelInfoScreen` | Channel details and management |
| `CreateChannelScreen` | Create a new channel |
| `CreatePostScreen` | Create a post in a channel |
| `ExplorePeopleScreen` | Discover and start chats with new users |
| `SyncUpScreen` | Meet — create/join video rooms |
| `MediaPreviewScreen` | Camera/mic preview before joining meeting |
| `ConferenceRoomScreen` | Active video conference with controls |
| `QRCodeScreen` | Scan QR code to login on web |
| `ConnectedDevicesScreen` | Manage connected web sessions |
| `SettingsScreen` | User profile and preferences |
| `VersionCheckScreen` | App update check |

#### API Layer (`src/api/`)

- `axiosClient.js` — Configured Axios instance with env-based URL switching
- `WebSocketConnection.js` — Socket.IO client with env-based URL
- `GoogleAuth.js` — Google Sign-In + backend token exchange
- `EmailAuth.js` — Email OTP signup/signin
- `ChatHandler.js` — Chat CRUD operations
- `MessageHandler.js` — Send/fetch messages, status updates, unseen counts
- `ChannelHandler.js` — Channel CRUD, join, posts
- `QRAuth.js` — QR code authentication
- `UserHandler.js` — User search, info fetching

#### State Management

**`AppContext`** — React Context providing:

- User state (`user`, `token`, `isSignedIn`)
- Socket lifecycle management
- Chat state (`chats`, `lastMessagesMap`, `unseenCountMap`)
- Active chat tracking (`activeChat`)

---

### 5. Web Client (React + Vite)

The web companion app, replicating the mobile app's core features with a premium desktop-first UI.

**Framework:** React 19 + Vite 7
**Port:** `5173`

#### Pages

| Page | Route | Purpose |
|------|-------|---------|
| `LoginPage` | `/login` | QR code auth + Email OTP login |
| `ChatPage` | `/chats`, `/chats/:chatId` | Split-panel: chat list + chat room |
| `ChannelsPage` | `/channels`, `/channels/:channelId` | Joined/suggested channels + posts |
| `MeetPage` | `/meet` | Create or join meeting room |
| `MeetRoom` | `/meet/:roomId` | Media preview → video conference |
| `SettingsPage` | `/settings` | User profile + logout |

#### Components

| Component | Purpose |
|-----------|---------|
| `Sidebar` | Left navigation bar (Chats, Channels, Meet, Settings) |
| `ChatCard` | Chat list item with avatar, last message, unseen count |
| `ChatRoom` | Message feed + socket integration + status updates |
| `MessageBubble` | Individual message with sent/delivered/seen status |
| `MessageInput` | Text input with send button + typing indicators |
| `ChannelCard` | Channel list item with join button |
| `ChannelRoom` | Channel post feed + create post |
| `OTPInput` | 6-digit OTP input with auto-focus |
| `MeetRoom` | Two-phase: media preview ↔ conference room |

#### API Layer (`src/api/`)

Mirrors the mobile app's API structure:

- `axiosClient.js` — Axios with `VITE_API_URL`, token interceptor
- `socketClient.js` — Socket.IO with `VITE_SOCKET_URL`
- `EmailAuth.js` — Email OTP login (no signup on web)
- `ChatHandler.js` — Fetch/create chats
- `MessageHandler.js` — Send/fetch messages, unseen counts
- `ChannelHandler.js` — Channel operations
- `UserHandler.js` — Fetch user info

#### mediasoup Layer (`src/mediasoup/`)

- `store.js` — Browser-compatible EventEmitter-based state store
- `mediaSoupClient.js` — WebRTC client using browser-native `getUserMedia`

#### Design System

Premium CSS design system in `index.css`:

- **Font:** Inter (Google Fonts)
- **Theme:** Custom CSS variables for colors, shadows, transitions
- **Animations:** `fadeIn`, `slideIn`, `pulse`
- **Responsive:** Sidebar collapses, split-panel layout
- **Dark accents:** Meeting room uses Google Meet-inspired dark theme

---

## Database Schema

### Models (MongoDB + Mongoose)

#### User

```javascript
{
  name: String,
  email: String,        // Unique
  profilePicture: String,
  about: String,
  provider: String,     // "google" | "email"
  createdAt: Date
}
```

#### Chat

```javascript
{
  type: String,         // "single" | "group"
  participants: [ObjectId],  // References → User
  createdAt: Date
}
```

#### Message

```javascript
{
  chatId: ObjectId,     // Reference → Chat
  senderId: ObjectId,   // Reference → User
  receiverId: ObjectId, // Reference → User
  content: String,
  type: String,         // "text" | "image" | "file"
  status: String,       // "sent" | "delivered" | "seen"
  createdAt: Date
}
```

#### Channel

```javascript
{
  name: String,
  description: String,
  createdBy: ObjectId,    // Reference → User
  members: [ObjectId],    // References → User
  profilePicture: String,
  createdAt: Date
}
```

#### Post

```javascript
{
  channelId: ObjectId,  // Reference → Channel
  content: String,
  createdBy: ObjectId,  // Reference → User
  createdAt: Date
}
```

#### OTP

```javascript
{
  email: String,
  otp: String,
  expiresAt: Date
}
```

#### QRAuth

```javascript
{
  sessionId: String,    // Unique
  token: String,
  verified: Boolean,
  createdAt: Date       // TTL: auto-expires
}
```

#### AppVersion

```javascript
{
  version: String,
  forceUpdate: Boolean,
  releaseNotes: String
}
```

---

## API Endpoints

### Authentication (`/auth`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/auth/signup` | Sign up with Google `idToken` |
| `POST` | `/auth/login` | Sign in with Google `idToken` |
| `POST` | `/auth/email/signup` | Sign up with email (sends OTP) |
| `POST` | `/auth/email/verify-signup` | Verify signup OTP |
| `POST` | `/auth/email/login` | Login with email (sends OTP) |
| `POST` | `/auth/email/verify-login` | Verify login OTP |
| `POST` | `/auth/qr/generate` | Generate QR session |
| `POST` | `/auth/qr/verify` | Verify QR session |
| `GET`  | `/auth/qr/status/:sessionId` | Check QR session status |

### Users (`/users`) — JWT Required

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/users/:id` | Get user by ID |
| `GET` | `/users/search?query=` | Search users by name/email |
| `GET` | `/users/suggest` | Get suggested users to chat with |

### Chats (`/chats`) — JWT Required

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/chats` | Get all chats for current user |
| `POST` | `/chats/single` | Create a 1:1 chat |
| `GET` | `/chats/suggest` | Get suggested chats |

### Messages (`/messages`) — JWT Required

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/messages/send` | Send a message |
| `GET` | `/messages/:chatId` | Get messages for a chat |
| `PATCH` | `/messages/status` | Update message status (delivered/seen) |
| `GET` | `/messages/unseen/count` | Get unseen message counts |

### Channels (`/channel`) — JWT Required

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/channel/create` | Create a new channel |
| `POST` | `/channel/join` | Join a channel |
| `GET` | `/channel/:id` | Get channel details |
| `GET` | `/channel/joined` | Get user's joined channels |
| `GET` | `/channel/suggested` | Get suggested channels |
| `POST` | `/channel/post/create` | Create a post in a channel |
| `GET` | `/channel/:id/posts` | Get posts for a channel |

### App (`/app`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/app/version` | Get latest app version |
| `POST` | `/app/version` | Set app version (admin) |

---

## Socket Events

### Client → Server

| Event | Payload | Description |
|-------|---------|-------------|
| `join` | `{ user: { _id } }` | Register user connection |
| `sendMessage` | `{ senderId, receiverId, message }` | Send message to a user |
| `deliveredMsg` | `{ msgId, senderId }` | Notify sender of delivery |
| `seenMsg` | `{ msgId, senderId }` | Notify sender message was seen |
| `QRCodeVerified` | `{ sessionId, token }` | Mobile confirms QR login |

### Server → Client

| Event | Payload | Description |
|-------|---------|-------------|
| `receiveMessage` | `{ message }` | Incoming message |
| `deliveredMsg` | `{ msgId }` | Message delivery confirmation |
| `seenMsg` | `{ msgId }` | Message seen confirmation |
| `QRAuthorized-FetchToken` | `{ sessionId, token }` | Web receives auth token |

---

## WebRTC Events

### Client → Server

| Event | Payload | Callback |
|-------|---------|----------|
| `createRoom` | — | `{ roomId }` |
| `joinRoom` | `{ roomId }` | `{ routerRtpCapabilities, existingProducers }` |
| `createTransport` | — | `{ id, iceParameters, iceCandidates, dtlsParameters }` |
| `connectTransport` | `{ id, dtls }` | callback() |
| `produce` | `{ id, kind, rtp }` | `{ id }` |
| `consume` | `{ producerId, rtpCaps, transportId }` | `{ id, kind, rtpParameters }` |
| `leaveRoom` | — | — |

### Server → Client

| Event | Payload | Description |
|-------|---------|-------------|
| `newProducer` | `{ producerId }` | New remote track available |
| `peerDisconnected` | `{ producers: [ids] }` | Peer left the room |

---

## Authentication Flow

### 1. Google Sign-In (Mobile Only)

```
Mobile → Google SDK → idToken → POST /auth/login → JWT token
                                                  → User stored in EncryptedStorage
```

### 2. Email OTP Login

```
Client → POST /auth/email/login { email }
Server → Generates 6-digit OTP → logs to console (dev) / sends email (prod)
Client → POST /auth/email/verify-login { email, otp }
Server → Verifies OTP → returns JWT token + user
```

### 3. QR Code Login (Web → Mobile)

```
Web → Generates sessionId → displays QR code
Mobile → Scans QR code → POST /auth/qr/verify { sessionId, token }
Socket Server → Emits "QRAuthorized-FetchToken" to web
Web → Receives token → stores in localStorage → redirects to /chats
```

---

## Environment Variables

### Main Server (`syncupchat-mainserver/.env`)

```env
PORT=3030
MONGO_REMOTE_DB=mongodb+srv://...
NODE_ENV=development           # "production" to use remote DB
JWT_SECRET=your_jwt_secret
```

### Socket Server (`syncupchat - socket-server/.env`)

```env
PORT=5000
```

### WebRTC Server (`webrtc - server/.env`)

```env
PORT=3000
```

### Web Client (`syncupchat-web-client/.env`)

```env
# Development
VITE_API_URL=http://localhost:3030/
VITE_SOCKET_URL=http://localhost:5000
VITE_WEBRTC_URL=http://localhost:3000

# Production
# VITE_API_URL=https://syncupchat-mainserver.onrender.com/
# VITE_SOCKET_URL=https://syncupchat-socket-server.onrender.com
# VITE_WEBRTC_URL=https://syncupchat-webrtc-server.onrender.com
```

### Mobile Client (`syncupchat/.env`)

```env
GOOGLE_WEB_CLIENT_ID=your_google_client_id
REACT_NATIVE_ENV=production    # Uncomment for production URLs
```

---

## Getting Started

### Prerequisites

- **Node.js** ≥ 20
- **MongoDB** (local instance or Atlas)
- **Android Studio** / **Xcode** (for mobile)
- **Git**

### 1. Clone & Install

```bash
# Main Server
cd syncupchat-mainserver
npm install

# Socket Server
cd ../syncupchat\ -\ socket-server
npm install

# WebRTC Server
cd ../webrtc\ -\ server
npm install

# Web Client
cd ../syncupchat-web-client
npm install

# Mobile Client
cd ../syncupchat
npm install
```

### 2. Configure Environment

Create `.env` files in each service directory (see [Environment Variables](#environment-variables)).

### 3. Start MongoDB

```bash
mongod
```

### 4. Start All Servers

Open separate terminals for each:

```bash
# Terminal 1 — Main Server
cd syncupchat-mainserver
npm start                    # → http://localhost:3030

# Terminal 2 — Socket Server
cd syncupchat\ -\ socket-server
node server.js               # → http://localhost:5000

# Terminal 3 — WebRTC Server
cd webrtc\ -\ server
node server.js               # → http://localhost:3000

# Terminal 4 — Web Client
cd syncupchat-web-client
npm run dev                  # → http://localhost:5173
```

### 5. Start Mobile Client

```bash
cd syncupchat
npm start                    # Metro bundler
npx react-native run-android # or run-ios
```

---

## Deployment

### Production URLs (Render)

| Service | URL |
|---------|-----|
| Main Server | `https://syncupchat-mainserver.onrender.com` |
| Socket Server | `https://syncupchat-socket-server.onrender.com` |
| WebRTC Server | `https://syncupchat-webrtc-server.onrender.com` |

### WebRTC Server (Docker)

The WebRTC server includes a `Dockerfile` for containerized deployment:

```bash
cd webrtc-server
docker build -t syncupchat-webrtc .
docker run -p 3000:3000 syncupchat-webrtc
```

### Mobile Client (Production Build)

```bash
cd syncupchat
# Set REACT_NATIVE_ENV=production in .env
npx react-native run-android --mode=release
```

### Web Client (Production Build)

```bash
cd syncupchat-web-client
# Switch .env to production URLs
npm run build                # Output → dist/
npm run preview              # Preview production build
```

---

## Author

**Pavan B N**

---

*Built with ❤️ using React Native, React, Node.js, MongoDB, Socket.IO, and mediasoup.*
