# Mzali Bakery — Comprehensive PRD & 13-Phase Specification

## Project Overview
**Application Name:** Mzali Bakery  
**Domain:** Artisanal Bakery Customization, Planning & Booking System  
**Platform:** Mobile-First Web Application (TanStack Start, React, Tailwind CSS, Lovable Cloud PostgreSQL)  
**Currency:** South African Rand (ZAR / R)  

---

## Phase 1: Project Initiation
### Purpose
Define what Mzali Bakery is, why it is needed, and what the project should achieve.

### Problem Statement
Customers frequently struggle with:
1. Deciding which cake suits their event.
2. Knowing the exact portion size needed for their guest count.
3. Tedious back-and-forth messaging to obtain quotes.
4. Keeping track of order specifications, delivery dates, and status updates.

### Solution
An AI-powered mobile-first web application that provides:
- A visual cake catalogue with filtering and search.
- An interactive 6-step Cake Planner with live price estimation.
- An AI Cake Assistant providing portion guidance and catalogue recommendations.
- A streamlined booking submission and confirmation flow.
- A bakery staff administrative dashboard to manage production.

---

## Phase 2: Requirements Gathering and Analysis
### 1. Functional Requirements
- **Customer:**
  - Browse and search categorized cakes (Birthday, Wedding, Graduation, Anniversary, Baby Shower, Chocolate, Red Velvet, Cupcakes, Custom).
  - Customize cake specifications: occasion, sponge/filling flavour, size, decorations, and custom piping text.
  - Calculate dynamic price estimates in ZAR.
  - Chat with AI for sizing recommendations, stock checks, and ordering FAQs.
  - Submit bookings with event date, collection vs delivery selection, and address.
  - View digital booking receipts and track status in "My Bookings".
- **Bakery Administrator:**
  - View incoming booking pipeline and KPI overview (Today's Bookings, Pending, Confirmed).
  - Update booking status (`Pending` -> `Confirmed` -> `Ready` -> `Completed` -> `Cancelled`).
  - Add, edit, and toggle stock availability for cakes and flavours.
  - Set bakery lead times and blackout dates.

### 2. Non-Functional Requirements
- **Mobile-First UX:** Minimum 44px touch targets, sticky bottom action bar, fast loading.
- **Performance:** Instant recalculation of pricing without full-page reloads.
- **Security:** Strict Row-Level Security (RLS) on PostgreSQL and validated inputs.

---

## Phase 3: System Planning
### Architecture
- **Client Tier:** React + TanStack Start, Tailwind CSS, shadcn/ui.
- **Backend Tier:** Lovable Cloud with Postgres database and REST/RPC server functions.
- **AI Engine:** Lovable AI grounded in active database catalogue records.

### Sizing Matrix
- **Small (15cm):** 6–8 servings (+R0)
- **Medium (20cm):** 12–16 servings (+R100)
- **Large (25cm):** 20–25 servings (+R220)
- **Extra Large (2-Tier):** 30+ servings (+R450)

---

## Phase 4: System Design
### Brand Identity & Aesthetic
- **Palette:** Terracotta Rose (`#D97757`), Vanilla Cream (`#FAF6F0`), Espresso Brown (`#2D2321`), Sage Green (`#E3ECE5`).
- **Typography:** Serif headings (*Playfair Display*) paired with clean sans-serif body (*Inter*).
- **Navigation:** Persistent bottom navigation (Home, Cakes, Planner, My Bookings).

---

## Phase 5: Database Design
### PostgreSQL Schema (DDL)

```sql
-- 1. Profiles (Authentication & Roles)
CREATE TABLE public.profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT NOT NULL UNIQUE,
  full_name TEXT NOT NULL,
  phone TEXT,
  role TEXT NOT NULL DEFAULT 'customer' CHECK (role IN ('customer', 'admin')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 2. Cakes Catalogue
CREATE TABLE public.cakes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  category TEXT NOT NULL CHECK (category IN (
    'birthday', 'wedding', 'graduation', 'anniversary', 
    'baby_shower', 'chocolate', 'red_velvet', 'cupcakes', 'custom'
  )),
  default_flavour TEXT NOT NULL,
  description TEXT NOT NULL,
  image_url TEXT NOT NULL,
  base_price NUMERIC(10,2) NOT NULL CHECK (base_price >= 0),
  is_available BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 3. Cake Customization Options
CREATE TABLE public.cake_options (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  type TEXT NOT NULL CHECK (type IN ('size', 'flavour', 'decoration', 'topping', 'fulfillment')),
  name TEXT NOT NULL,
  additional_price NUMERIC(10,2) NOT NULL DEFAULT 0.00,
  is_available BOOLEAN NOT NULL DEFAULT TRUE
);

-- 4. Bookings
CREATE TABLE public.bookings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  booking_ref TEXT NOT NULL UNIQUE,
  user_id UUID REFERENCES public.profiles(id) ON DELETE SET NULL,
  customer_name TEXT NOT NULL,
  customer_email TEXT NOT NULL,
  customer_phone TEXT NOT NULL,
  cake_id UUID REFERENCES public.cakes(id) ON DELETE SET NULL,
  occasion TEXT NOT NULL,
  flavour TEXT NOT NULL,
  size TEXT NOT NULL,
  design_details JSONB NOT NULL DEFAULT '{}'::jsonb,
  custom_message TEXT,
  event_date DATE NOT NULL,
  fulfillment_type TEXT NOT NULL CHECK (fulfillment_type IN ('collection', 'delivery')),
  delivery_address TEXT,
  special_instructions TEXT,
  estimated_price NUMERIC(10,2) NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN (
    'pending', 'confirmed', 'ready', 'completed', 'cancelled'
  )),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## Phase 6: User Interface Development
- **CakePlannerWizard.tsx:** 6-step state machine with real-time price calculator.
- **AIChatDrawer.tsx:** Bottom sheet chat assistant with quick action pills.
- **ConfirmationCard.tsx:** Shareable digital receipt card with reference number and live status indicator.
- **AdminDashboard.tsx:** Metrics overview and pipeline management table.

---

## Phase 7: Cake Planning and Booking Development
### Dynamic Price Calculation Formula
$$\text{Total} = \text{Base Price} + \text{Size Tier Markup} + \sum(\text{Decoration Add-ons}) + \text{Piping Fee (R50)} + \text{Delivery (R80)}$$

### Lead Time Enforcement
- Standard orders require a **minimum 48-hour advance notice**.
- Multi-tier and wedding cakes require a **minimum 7-day advance notice**.

---

## Phase 8: AI Chatbot Development
### Prompt Grounding & Guardrails
- Grounds responses in active database inventory.
- Suggests portions accurately based on guest count.
- Intercepts completed planning discussions to pre-fill the booking checkout.

---

## Phase 9: Administrator System
- Order management with one-click status transitions:
  $$\text{Pending} \rightarrow \text{Confirmed} \rightarrow \text{Ready} \rightarrow \text{Completed}$$
- Catalogue management with instant stock toggling (syncs to AI and catalogue).

---

## Phase 10: Testing Matrix
- T1–T12 QA test coverage: keyword search, filter combinations, state persistence, pricing calculations, date validation, booking submissions, AI sizing recommendations, and admin authorization gating.

---

## Phase 11: Security and Validation
- **Zod Validation:** Schema validation for names, phone numbers (+27 / South Africa format), email addresses, and character caps.
- **Row-Level Security (RLS):** Customers can only read and modify their own orders; admins have full operational access.
- **Server-Side Price Verification:** All pricing recalculated server-side upon booking creation to prevent tampering.

---

## Phase 12: Deployment
- Hosted on edge architecture via Lovable Cloud with automatic SSL/TLS.
- Custom domain routing (e.g. `orders.mzalibakery.co.za`).
- Mobile PWA configuration enabled for home screen installation.

---

## Phase 13: Maintenance and Improvement
- **Roadmap:**
  - Online payment gateway integration (Stripe / SnapScan / Ozow).
  - Automated WhatsApp order confirmations and dispatch notifications.
  - AI image generation for custom cake mockups.
  - Printable daily kitchen baking sheets.
