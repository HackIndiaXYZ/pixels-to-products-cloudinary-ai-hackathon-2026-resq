# 🎨 ImaGod — AI-Powered Creative Studio

## [🎬 DEMO VIDEO](https://drive.google.com/file/d/1J7sgcsiQw9Xe2mmh6Kj0MWvk28ZnV6ls/view?usp=sharing)
## [Project Live Link](https://ima-god.vercel.app)

> **Hackathon Track: Your Media-Savvy Startup**
>
> ImaGod is a full-stack, SaaS-style AI image creation and editing platform where **Cloudinary is the engine**, not just the file host. Every core feature — text-to-image generation, background removal, photo enhancement, generative replace, generative recolor, outpainting, and deblurring — runs through Cloudinary's AI and transformation pipeline in real time.

[![Built with Cloudinary](https://img.shields.io/badge/Built%20with-Cloudinary-3448C5?style=for-the-badge&logo=cloudinary&logoColor=white)](#-cloudinary-integration-summary)
[![React](https://img.shields.io/badge/React-19-61DAFB?style=for-the-badge&logo=react&logoColor=black)](#-tech-stack)
[![Node.js](https://img.shields.io/badge/Node.js-Express%205-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)](#-tech-stack)
[![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-47A248?style=for-the-badge&logo=mongodb&logoColor=white)](#-tech-stack)

---

## 📋 Table of Contents

- [Features at a Glance](#-features-at-a-glance)
- [Architecture](#-architecture)
  - [High-Level Overview](#high-level-overview)
  - [System Architecture Diagram](#system-architecture-diagram)
  - [Data Flow](#data-flow)
  - [Frontend Architecture](#frontend-architecture)
  - [Backend Architecture](#backend-architecture)
  - [Cloudinary Integration Summary](#cloudinary-integration-summary)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Step 1 — Clone the Repository](#step-1--clone-the-repository)
  - [Step 2 — Install Dependencies](#step-2--install-dependencies)
  - [Step 3 — Configure Environment Variables](#step-3--configure-environment-variables)
  - [Step 4 — Start the Development Servers](#step-4--start-the-development-servers)
  - [Step 5 — Verify Everything Works](#step-5--verify-everything-works)
- [Environment Variables Reference](#-environment-variables-reference)
- [API Reference](#-api-reference)
- [Credits & Monetisation](#-credits--monetisation)
- [License](#-license)

---

## ✨ Features at a Glance

| # | Feature | Description | Cloudinary Capability |
|---|---|---|---|
| 1 | **Text-to-Image Generation** | Type a prompt, pick a style & aspect ratio — get an AI-generated image | `Text-to-Image Gen API v2` |
| 2 | **Background Removal** | Upload any photo → clean transparent PNG | `background_removal: cloudinary_ai` |
| 3 | **Photo Enhancement** | AI-improved, upscaled photo | `e_improve` + `e_upscale` |
| 4 | **Generative Replace** | Swap any object via text prompts | `e_gen_replace` |
| 5 | **Generative Recolor** | Recolor a specific object to any color | `e_gen_recolor` |
| 6 | **Generative Fill / Outpainting** | Extend an image to a new aspect ratio with AI | `b_gen_fill` + `c_pad` |
| 7 | **AI Unblur / Deblur** | Sharpen blurry photos (Standard / Motion / Face) | `e_unsharp_mask` + `e_sharpen` |
| 8 | **Media History & Gallery** | Browse, filter, delete all past creations | `cloudinary.search` |
| 9 | **Usage Analytics Dashboard** | Per-feature credit consumption breakdown | `cloudinary.search` sync |
| 10 | **Credit Purchase** | Buy credits via Razorpay; credits never expire | Razorpay SDK |

---

## 🏗 Architecture

### High-Level Overview

ImaGod follows a **decoupled client–server monorepo** pattern with three external service integrations:

```
┌──────────────┐        REST / JSON        ┌──────────────┐
│   React SPA  │ ◄─────────────────────►  │  Express API │
│   (Vite)     │       axios / JWT         │  (Node.js)   │
└──────┬───────┘                           └──┬───┬───┬───┘
       │                                      │   │   │
       │  Razorpay.js                         │   │   │
       ▼                                      │   │   │
┌──────────────┐                              │   │   │
│   Razorpay   │◄─────────────────────────────┘   │   │
│  (Payments)  │  create order / verify           │   │
└──────────────┘                                  │   │
                                                  │   │
┌──────────────┐                                  │   │
│  MongoDB     │◄─────────────────────────────────┘   │
│  Atlas       │  Mongoose ODM                        │
└──────────────┘                                      │
                                                      │
┌──────────────────────────────────────┐              │
│           Cloudinary                 │◄─────────────┘
│                                      │  SDK v2 / REST API
│  • Generation API (text-to-image)    │
│  • Upload API (upload_stream)        │
│  • URL Transforms (on-the-fly)       │
│  • Search API (user history)         │
│  • Admin API (polling / ownership)   │
│  • Tag Management (per-user assets)  │
│  • CDN Delivery (global edge)        │
└──────────────────────────────────────┘
```

| Layer | Runs On | Responsibility |
|---|---|---|
| **Client** | Browser (port `5173`) | UI rendering, routing, state, API calls |
| **Server** | Node.js (port `4000`) | Auth, business logic, Cloudinary orchestration, payments |
| **MongoDB Atlas** | Cloud | Persistent user data, credits, transactions, usage logs |
| **Cloudinary** | Cloud + CDN | All AI processing, media storage, transformations, delivery |
| **Razorpay** | Cloud | Payment processing (test mode supported) |

---

### System Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                        CLIENT  (React 19 + Vite 7)                   │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                         React Router v7                        │  │
│  │                                                                │  │
│  │  /              → Home        (Landing — Hero, Features)       │  │
│  │  /result        → Result      (Text-to-Image chat studio)      │  │
│  │  /remove-bg     → RemoveBg    (Background removal)             │  │
│  │  /enhance       → Enhance     (Photo enhancement)              │  │
│  │  /unblur        → Unblur      (AI deblur)                      │  │
│  │  /ai-editor     → AiEditor    (Gen Replace & Recolor)          │  │
│  │  /gen-fill      → GenFill     (Outpainting)                    │  │
│  │  /history       → History     (Media gallery)                  │  │
│  │  /usage         → Usage       (Analytics dashboard)            │  │
│  │  /buycredit     → BuyCredit   (Purchase credits)               │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │ AppContext    │  │ Framer Motion│  │ GSAP + Lenis │               │
│  │ (global state │  │ (page trans- │  │ (scroll-     │               │
│  │  & API calls) │  │  itions)     │  │  animations) │               │
│  └──────┬───────┘  └──────────────┘  └──────────────┘               │
│         │  axios                                                     │
└─────────┼────────────────────────────────────────────────────────────┘
          │  REST API (JSON + multipart/form-data)
          │  Authorization: Bearer <JWT>
          ▼
┌──────────────────────────────────────────────────────────────────────┐
│                    SERVER  (Express 5 + Node.js)                     │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │  Middleware Pipeline                                           │  │
│  │  express.json() → cors() → multer (memory) → JWT auth         │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌─────────────────┐  ┌──────────────────┐  ┌─────────────────┐    │
│  │  userController  │  │  imageController  │  │ historyController│   │
│  │                  │  │                   │  │                  │   │
│  │ • register       │  │ • generateImage   │  │ • getUserHistory │   │
│  │ • login          │  │ • removeBg        │  │ • deleteHistory  │   │
│  │ • userCredits    │  │ • enhanceImage    │  │   Item           │   │
│  │ • paymentRazor   │  │ • genReplace      │  │                  │   │
│  │ • verifyRazor    │  │ • genRecolor      │  │  (Cloudinary     │   │
│  │ • getUserUsage   │  │ • genFill         │  │   Search API)    │   │
│  │                  │  │ • unblurImage     │  │                  │   │
│  └──────┬───────────┘  └──────┬───────────┘  └──────┬───────────┘  │
│         │                     │                      │              │
│  ┌──────┴─────────────────────┴──────────────────────┴───────────┐  │
│  │                    Route Layer                                 │  │
│  │  /api/user/*   → userRoutes.js    (auth, credits, payments,   │  │
│  │                                    history)                    │  │
│  │  /api/image/*  → imageRoutes.js   (all 7 AI image endpoints)  │  │
│  └───────────────────────────────────────────────────────────────┘  │
└──────────┬─────────────────┬─────────────────┬───────────────────────┘
           │                 │                 │
           ▼                 ▼                 ▼
   ┌──────────────┐  ┌──────────────────┐  ┌──────────────┐
   │   MongoDB    │  │    Cloudinary    │  │   Razorpay   │
   │   Atlas      │  │                  │  │              │
   │              │  │  ┌────────────┐  │  │  • Create    │
   │  Models:     │  │  │ Gen API v2 │  │  │    Order     │
   │  • User      │  │  │ (text →    │  │  │  • Verify    │
   │    - name    │  │  │  image)    │  │  │    Payment   │
   │    - email   │  │  ├────────────┤  │  │              │
   │    - password│  │  │ Upload API │  │  └──────────────┘
   │    - credits │  │  │ (stream +  │  │
   │    - usage   │  │  │  bg_remove)│  │
   │  • Txn       │  │  ├────────────┤  │
   │    - amount  │  │  │ URL Trans- │  │
   │    - credits │  │  │ formations │  │
   │    - payment │  │  ├────────────┤  │
   │    - date    │  │  │ Search API │  │
   │              │  │  │ (history)  │  │
   │              │  │  ├────────────┤  │
   │              │  │  │ Admin API  │  │
   │              │  │  │ (poll/own) │  │
   │              │  │  ├────────────┤  │
   │              │  │  │ CDN Edge   │  │
   │              │  │  │ (delivery) │  │
   │              │  │  └────────────┘  │
   └──────────────┘  └──────────────────┘
```

---

### Data Flow

Below are the two primary request patterns in ImaGod:

#### Flow A — Text-to-Image Generation

```
User types prompt
       │
       ▼
  AppContext.jsx ──► POST /api/image/generate-image { prompt }
                          │
                          ▼  (imageController.js)
                    Cloudinary Generation API v2
                    POST /v2/generate/{cloud}/text_to_image
                          │
                          ▼
                    Cloudinary stores result in
                    ImaGod/generated/{timestamp}
                          │
                          ▼
                    Tag asset with user's MongoDB _id
                          │
                          ▼
                    Deduct 1 credit (MongoDB)
                          │
                          ▼
                    Return CDN URL to client
```

#### Flow B — Upload-Based Transformation (BG Remove, Enhance, Replace, Recolor, Fill, Unblur)

```
User uploads image + params
       │
       ▼
  AppContext.jsx ──► POST /api/image/{feature}  (multipart FormData)
                          │
                          ▼  (imageController.js)
                    Multer stores file in memory buffer
                          │
                          ▼
                    Upload to Cloudinary via upload_stream
                    → folder: ImaGod/{feature}/
                    → tags: [userId]
                          │
                 ┌────────┴─────────┐
                 │                  │
          (bg-removal only)    (all others)
                 │                  │
          Poll resource()      Build transformation URL
          until status ===     via cloudinary.url()
          'complete'           with chained effects
                 │                  │
                 └────────┬─────────┘
                          │
                          ▼
                    Deduct 1 credit (MongoDB)
                          │
                          ▼
                    Return CDN URL to client
```

---

### Frontend Architecture

```
src/
├── App.jsx                  ← Root: routes + layout (Navbar, Footer, Login modal)
├── main.jsx                 ← Entry point: BrowserRouter + AppContext provider
├── index.css                ← Global styles (Tailwind base + custom tokens)
│
├── context/
│   └── AppContext.jsx       ← Global state: auth, credits, all API methods
│
├── pages/                   ← One component per route (self-contained features)
│   ├── Home.jsx             ← Landing page (hero, features, testimonials)
│   ├── Result.jsx           ← Text-to-Image chat studio (prompt → image)
│   ├── RemoveBg.jsx         ← Background removal (upload → result)
│   ├── Enhance.jsx          ← Photo enhancement (upload → result)
│   ├── Unblur.jsx           ← AI deblur (upload + mode select → result)
│   ├── AiEditor.jsx         ← Gen Replace & Recolor (upload + prompts → result)
│   ├── GenFill.jsx          ← Outpainting (upload + aspect ratio → result)
│   ├── History.jsx          ← Media gallery (Cloudinary search → grid)
│   ├── Usage.jsx            ← Analytics dashboard (per-feature breakdown)
│   └── BuyCredit.jsx        ← Credit purchase plans (Razorpay checkout)
│
├── components/              ← Shared / presentational components
│   ├── Navbar.jsx           ← Navigation with auth state & credit display
│   ├── Footer.jsx           ← Site footer with links
│   ├── Login.jsx            ← Auth modal (login ↔ register toggle)
│   ├── ScrollMorphHero.jsx  ← Animated landing hero section
│   ├── VideoHero.jsx        ← Video showcase section
│   ├── Description.jsx      ← Feature description blocks
│   ├── Steps.jsx            ← "How it works" steps
│   ├── GenerateBtn.jsx      ← CTA button component
│   ├── CloudShader.jsx      ← WebGL cloud shader effect
│   ├── SoftGradientBackground.jsx
│   ├── InteractiveHoverButton.jsx
│   ├── ui/                  ← Reusable primitives (Button, Card, Input, etc.)
│   └── effects/             ← Visual effects (timeline animations)
│
├── assets/                  ← Static images, icons, videos (logo, samples, etc.)
└── lib/
    └── utils.ts             ← Shared utility functions
```

**Key Patterns:**
- **AppContext** centralises all API calls and auth state — pages consume via `useContext(AppContext)`
- **Framer Motion** handles page transitions and micro-animations
- **GSAP + Lenis** power scroll-driven animations on the landing page
- **Tailwind CSS 3** for all styling with custom design tokens in `index.css`

---

### Backend Architecture

```
server/
├── server.js                ← Entry: Express app, middleware, route mounting
│
├── config/
│   ├── cloudinary.js        ← Cloudinary SDK v2 initialisation (cloud_name, api_key, api_secret)
│   └── mongodb.js           ← Mongoose connection to MongoDB Atlas
│
├── middlewares/
│   └── auth.js              ← JWT verification middleware (extracts userId from token)
│
├── models/
│   ├── userModel.js         ← User schema: name, email, password, creditBalance, usageLogs
│   └── transactionModel.js  ← Transaction schema: userId, plan, amount, credits, payment, date
│
├── controllers/
│   ├── imageController.js   ← 7 AI endpoints — orchestrates Cloudinary SDK calls
│   │   ├── generateImage()  ← Text-to-Image Gen API v2
│   │   ├── removeBg()       ← upload_stream + background_removal: cloudinary_ai + polling
│   │   ├── enhanceImage()   ← upload → URL transform (e_improve, e_upscale, q_auto:best)
│   │   ├── genReplace()     ← upload → URL transform (e_gen_replace:from_X;to_Y)
│   │   ├── genRecolor()     ← upload → URL transform (e_gen_recolor:prompt_X;to-color_Y)
│   │   ├── genFill()        ← upload → URL transform (c_pad, g_center, b_gen_fill)
│   │   └── unblurImage()    ← upload → URL transform (e_unsharp_mask, e_sharpen, e_improve)
│   │
│   ├── userController.js    ← Auth, credits & payments
│   │   ├── registerUser()   ← bcrypt hash + JWT sign + 5 free credits
│   │   ├── loginUser()      ← bcrypt compare + JWT sign
│   │   ├── userCredits()    ← Return balance
│   │   ├── getUserUsage()   ← Return usage stats (auto-sync from Cloudinary if needed)
│   │   ├── paymentRazorpay()← Create Razorpay order
│   │   └── verifyRazorpay() ← Verify signature + add credits
│   │
│   └── historyController.js ← Media history via Cloudinary Search API
│       ├── getUserHistory() ← Search assets by tag (userId) + folder prefix
│       └── deleteHistoryItem() ← Verify tag ownership → cloudinary.uploader.destroy
│
└── routes/
    ├── imageRoutes.js       ← /api/image/* — multer upload + auth middleware
    └── userRoutes.js        ← /api/user/* — auth, credits, payments, history
```

**Key Patterns:**
- **Cloudinary as the source of truth** for all media — no duplicate asset metadata in MongoDB
- **Tag-based ownership** — every asset is tagged with `userId` for per-user isolation
- **Upload → URL Transform** pattern — source images are uploaded, then transformed via URL construction (no intermediate files)
- **Polling for async ops** — background removal uses `cloudinary.api.resource()` polling until processing completes

---

### Cloudinary Integration Summary

| Cloudinary API / Feature | Where Used | Code Location |
|---|---|---|
| **Text-to-Image Generation API v2** | AI image generation from prompts | `imageController.js → generateImage()` |
| **Upload API** (`upload_stream`) | BG removal, enhance, replace, recolor, fill, unblur | `imageController.js` (all upload endpoints) |
| **Background Removal Add-on** (`cloudinary_ai`) | AI background eraser | `imageController.js → removeBg()` |
| **URL-based Transformations** | Enhance, Gen Replace, Gen Recolor, Gen Fill, Unblur | `imageController.js` (URL construction) |
| **Admin API** (`api.resource`) | Polling async status, ownership verification | `imageController.js`, `historyController.js` |
| **Search API** (`cloudinary.search`) | User history, usage analytics sync | `historyController.js`, `userController.js` |
| **Tag Management** (`uploader.add_tag`) | Per-user asset ownership | `imageController.js` |
| **Asset Deletion** (`uploader.destroy`) | History cleanup | `historyController.js` |
| **CDN Delivery** (automatic) | All images served via global CDN | All endpoints |

---

## 🛠 Tech Stack

### Frontend

| Technology | Version | Purpose |
|---|---|---|
| **React** | 19 | UI framework |
| **Vite** | 7 | Build tool & dev server |
| **Tailwind CSS** | 3 | Utility-first styling |
| **Framer Motion** | 14 | Page transitions & micro-animations |
| **GSAP** | 3.15 | Advanced scroll-driven animations |
| **Lenis** | 1.3 | Smooth scrolling |
| **Lucide React** | 1.49 | Icon library |
| **React Router** | v7 | Client-side routing |
| **Axios** | 1.13 | HTTP client |
| **React Toastify** | 11 | Toast notifications |

### Backend

| Technology | Version | Purpose |
|---|---|---|
| **Node.js + Express** | 5 | REST API server |
| **Cloudinary SDK** | v2.11 | Image AI, transforms, search, uploads |
| **Mongoose** | 9 + MongoDB Atlas | User data, credits, transactions |
| **Multer** | 2.3 | Multipart file upload (memory storage) |
| **bcrypt** | 6 | Password hashing |
| **jsonwebtoken** | 9 | JWT-based authentication |
| **Razorpay SDK** | 2.9 | Payment gateway |
| **dotenv** | 17 | Environment variable management |

---

## 📂 Project Structure

```
pixels-to-products-cloudinary-ai-hackathon-2026-resq/
│
├── client/                              # React frontend (Vite)
│   ├── public/                          # Static public assets
│   ├── src/
│   │   ├── assets/                      # Images, icons, videos, sample images
│   │   ├── components/
│   │   │   ├── ui/                      # Reusable UI primitives (Button, Card, Input, etc.)
│   │   │   ├── effects/                 # Visual effects (timeline animations)
│   │   │   ├── Navbar.jsx               # Navigation with auth state
│   │   │   ├── Footer.jsx               # Site footer
│   │   │   ├── Login.jsx                # Auth modal (login / register)
│   │   │   ├── ScrollMorphHero.jsx      # Animated landing hero
│   │   │   ├── VideoHero.jsx            # Video showcase section
│   │   │   ├── Description.jsx          # Feature description blocks
│   │   │   ├── Steps.jsx                # "How it works" section
│   │   │   ├── GenerateBtn.jsx          # CTA button component
│   │   │   ├── CloudShader.jsx          # WebGL cloud shader effect
│   │   │   ├── SoftGradientBackground.jsx
│   │   │   └── InteractiveHoverButton.jsx
│   │   ├── context/
│   │   │   └── AppContext.jsx           # Global state & all API methods
│   │   ├── lib/
│   │   │   └── utils.ts                 # Shared utilities
│   │   ├── pages/
│   │   │   ├── Home.jsx                 # Landing page
│   │   │   ├── Result.jsx               # Text-to-Image studio
│   │   │   ├── RemoveBg.jsx             # Background removal
│   │   │   ├── Enhance.jsx              # Photo enhancement
│   │   │   ├── Unblur.jsx               # AI deblur
│   │   │   ├── AiEditor.jsx             # Gen Replace & Recolor
│   │   │   ├── GenFill.jsx              # Outpainting / generative fill
│   │   │   ├── History.jsx              # Media gallery
│   │   │   ├── Usage.jsx                # Usage analytics dashboard
│   │   │   └── BuyCredit.jsx            # Credit purchase & pricing
│   │   ├── App.jsx                      # Root component with routes
│   │   ├── main.jsx                     # Entry point
│   │   └── index.css                    # Global styles + Tailwind
│   ├── .env.example                     # Frontend env template
│   ├── tailwind.config.js               # Tailwind configuration
│   ├── vite.config.js                   # Vite configuration
│   └── package.json
│
├── server/                              # Express backend
│   ├── config/
│   │   ├── cloudinary.js                # Cloudinary SDK initialisation
│   │   └── mongodb.js                   # Mongoose connection
│   ├── controllers/
│   │   ├── imageController.js           # All 7 image AI endpoints
│   │   ├── userController.js            # Auth, credits, payments, usage
│   │   └── historyController.js         # Cloudinary Search API for history
│   ├── middlewares/
│   │   └── auth.js                      # JWT verification middleware
│   ├── models/
│   │   ├── userModel.js                 # User schema (credits, usageLogs)
│   │   └── transactionModel.js          # Payment transaction schema
│   ├── routes/
│   │   ├── imageRoutes.js               # /api/image/* routes
│   │   └── userRoutes.js                # /api/user/* routes
│   ├── .env.example                     # Backend env template
│   ├── package.json
│   └── server.js                        # Express app entry point
│
├── .env.example                         # Root env reference (points to client/ & server/)
├── .gitignore
├── LICENSE                              # MIT License
└── README.md                            # ← You are here
```

---

## 🚀 Getting Started

### Prerequisites

Before you begin, make sure you have the following installed and configured:

| Prerequisite | Details |
|---|---|
| **Node.js** | v18 or later ([download](https://nodejs.org/)) |
| **npm** | Comes with Node.js |
| **MongoDB Atlas** | Free cluster ([sign up](https://www.mongodb.com/cloud/atlas/register)) |
| **Cloudinary** | Free account ([sign up](https://cloudinary.com/users/register_free)) |
| **Razorpay** | Test mode account ([sign up](https://dashboard.razorpay.com/signup)) |

> [!IMPORTANT]
> You **must** enable the **Cloudinary AI Background Removal** add-on in your Cloudinary dashboard for the background removal feature to work. Go to **Settings → Add-ons** in your Cloudinary console.

---

### Step 1 — Clone the Repository

```bash
git clone https://github.com/HackIndiaXYZ/pixels-to-products-cloudinary-ai-hackathon-2026-resq.git
cd pixels-to-products-cloudinary-ai-hackathon-2026-resq
```

---

### Step 2 — Install Dependencies

Install both server and client dependencies:

```bash
# Server dependencies
cd server
npm install

# Client dependencies
cd ../client
npm install

# Return to project root
cd ..
```

---

### Step 3 — Configure Environment Variables

The project uses two `.env` files — one for the server and one for the client. Templates are provided:

```bash
# Copy templates
cp server/.env.example server/.env
cp client/.env.example client/.env
```

Now open each `.env` file and fill in your credentials:

#### `server/.env`

```env
# Server Port (optional, defaults to 4000)
PORT=4000

# MongoDB Atlas connection string
MONGODB_URI=mongodb+srv://<username>:<password>@<cluster-url>

# Secret key for signing JWT tokens (use any strong random string)
JWT_SECRET=your_jwt_secret_key_here

# Cloudinary credentials (from your Cloudinary Console → Dashboard)
CLOUDINARY_CLOUD_NAME=your_cloudinary_cloud_name
CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_API_SECRET=your_cloudinary_api_secret

# Razorpay credentials (from Razorpay Dashboard → API Keys)
RAZORPAY_KEY_ID=rzp_test_your_key_id
RAZORPAY_KEY_SECRET=your_razorpay_key_secret
CURRENCY=INR
```

#### `client/.env`

```env
# Backend API URL
VITE_BACKEND_URL=http://localhost:4000

# Razorpay public key (same Key ID as server, used for client-side checkout)
VITE_RAZORPAY_KEY_ID=rzp_test_your_key_id
```

> [!TIP]
> **Where to find your Cloudinary credentials:**
> Log into [console.cloudinary.com](https://console.cloudinary.com/) → Dashboard → copy **Cloud Name**, **API Key**, and **API Secret**.

---

### Step 4 — Start the Development Servers

Open **two terminal windows** and run:

**Terminal 1 — Backend:**
```bash
cd server
npm run server
```
This starts the Express server with `nodemon` on **port 4000** (auto-restarts on file changes).

**Terminal 2 — Frontend:**
```bash
cd client
npm run dev
```
This starts the Vite dev server on **port 5173** (with hot module replacement).

---

### Step 5 — Verify Everything Works

1. Open **[http://localhost:5173](http://localhost:5173)** in your browser
2. You should see the ImaGod landing page with animations
3. Click **Login** → switch to **Sign Up** → create an account
4. You'll receive **5 free credits** on signup
5. Try any feature:
   - Navigate to **Generate** → type a prompt → get an AI image
   - Navigate to **Remove BG** → upload a photo → get a transparent PNG
   - Check **History** → see all your creations from Cloudinary

> [!NOTE]
> To verify Cloudinary is working, log into your [Cloudinary Console](https://console.cloudinary.com/) → **Media Library** and look for the `ImaGod/` folder structure with your generated assets.

**For Razorpay test payments**, use:
- Card number: `4111 1111 1111 1111`
- Any future expiry date
- Any CVV

---

## 🔑 Environment Variables Reference

### Server (`server/.env`)

| Variable | Required | Default | Description |
|---|---|---|---|
| `PORT` | No | `4000` | Express server port |
| `MONGODB_URI` | ✅ | — | MongoDB Atlas connection string |
| `JWT_SECRET` | ✅ | — | Secret for JWT token signing |
| `CLOUDINARY_CLOUD_NAME` | ✅ | — | Cloudinary cloud name |
| `CLOUDINARY_API_KEY` | ✅ | — | Cloudinary API key |
| `CLOUDINARY_API_SECRET` | ✅ | — | Cloudinary API secret |
| `RAZORPAY_KEY_ID` | ✅ | — | Razorpay Key ID (`rzp_test_*` for dev) |
| `RAZORPAY_KEY_SECRET` | ✅ | — | Razorpay Key Secret |
| `CURRENCY` | No | `INR` | Payment currency code |

### Client (`client/.env`)

| Variable | Required | Default | Description |
|---|---|---|---|
| `VITE_BACKEND_URL` | ✅ | — | Backend API base URL (e.g., `http://localhost:4000`) |
| `VITE_RAZORPAY_KEY_ID` | ✅ | — | Razorpay public key for checkout widget |

---

## 📡 API Reference

### Auth & User (`/api/user`)

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/register` | ✗ | Create account (returns JWT + 5 free credits) |
| `POST` | `/login` | ✗ | Login (returns JWT) |
| `GET` | `/credits` | ✓ | Get credit balance & user info |
| `GET` | `/usage` | ✓ | Get per-feature usage stats (auto-syncs from Cloudinary) |
| `POST` | `/pay-razor` | ✓ | Create Razorpay payment order |
| `POST` | `/verify-razor` | ✓ | Verify payment signature & add credits |
| `GET` | `/history` | ✓ | Fetch media history via Cloudinary Search API |
| `POST` | `/history/delete` | ✓ | Delete a history item (ownership-verified) |

### Image AI (`/api/image`) — all require auth + 1 credit

| Method | Endpoint | Body | Cloudinary Feature |
|---|---|---|---|
| `POST` | `/generate-image` | `{ prompt }` | Text-to-Image Gen API v2 |
| `POST` | `/remove-bg` | `FormData: image` | `background_removal: cloudinary_ai` |
| `POST` | `/enhance` | `FormData: image` | `e_improve` + `e_upscale` |
| `POST` | `/gen-replace` | `FormData: image, from, to` | `e_gen_replace` |
| `POST` | `/gen-recolor` | `FormData: image, prompt, color` | `e_gen_recolor` |
| `POST` | `/gen-fill` | `FormData: image, aspectRatio` | `b_gen_fill` + `c_pad` |
| `POST` | `/unblur` | `FormData: image, mode` | `e_unsharp_mask` + `e_sharpen` |

---

## 💳 Credits & Monetisation

ImaGod uses a **credit-based pay-per-use** model. Each AI operation costs **1 credit**:

| Plan | Credits | Price (INR) |
|---|---|---|
| **Free** (on signup) | 5 | ₹0 |
| **Basic** | 100 | ₹10 |
| **Advanced** | 500 | ₹50 |
| **Business** | 5,000 | ₹250 |

Credits **never expire**. Payments are processed via Razorpay (test mode supported for development).

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

<p align="center">
  <b>Built with ❤️ and Cloudinary</b><br/>
  <i>ImaGod — Where every pixel is powered by Cloudinary.</i>
</p>
