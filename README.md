# ARC & THREAD — AI-Powered E-Commerce Platform

An e-commerce storefront with an n8n-powered backend and role-aware AI chatbot agents (separate flows for the store owner and for customers). Built on Supabase (Postgres + pgvector) for products, semantic search, and orders.

## Overview

ARC & THREAD is a clothing storefront where an AI assistant is embedded on both the customer-facing shop and the admin dashboard:

- **Customers** get a chat assistant that can search products, check stock, and add items to their cart.
- **Owners/Admins** get a chat assistant that can add, update, delete, and report on inventory — alongside a full manual table-based inventory manager.

Checkout itself is deterministic (not AI-driven): a dedicated workflow validates stock, records the order, and decrements inventory.

## Screenshots

| | |
|---|---|
| ![Shop homepage](screenshots/01-shop-homepage.png) | ![Catalogue](screenshots/02-catalogue.png) |
| ![Admin inventory](screenshots/03-admin-inventory.png) | ![Checkout modal](screenshots/04-checkout-modal.png) |

Backend workflow graphs (n8n):

| | |
|---|---|
| ![Owner product tool workflow](screenshots/05-owner-workflow.png) | ![Chat backend workflow](screenshots/06-chat-backend-workflow.png) |

## Architecture

The backend is 4 n8n workflows:

1. **Ecommerce Chat Backend** — chat webhook → role router → Owner Agent / Customer Agent (LLM via Groq) → shapes and returns the reply.
2. **Owner Product Tool** — full CRUD on products (add / update / delete / search / stock report), callable both as an AI tool and via a direct admin webhook.
3. **Customer Product Tool** — read-only-ish tool for the customer agent: search, check stock, add-to-cart intent. No destructive actions.
4. **Checkout + Products List** — non-AI, deterministic: re-validates stock, writes the order, decrements stock, and serves the plain product list.

### Data layer (Supabase)

- `products` — name, price, stock, category, description, image_url
- `product_chunks` — text chunks + embeddings (pgvector) for semantic product search
- `orders`, `order_items` — order records written at checkout

### Frontend

- `shop.html` — storefront: product grid, search, cart drawer, checkout modal, floating chat widget
- `admin.html` — single-page inventory manager (editable table + stats) with a docked AI ops assistant panel
- `admin-login.html` — placeholder client-side login gate for the admin panel

## Setup

This repo does **not** include any live credentials, instance IDs, or endpoints — you'll need to supply your own:

1. Create a Supabase project and set up the `products`, `product_chunks`, `orders`, `order_items` tables (with pgvector enabled for `product_chunks`).
2. Import the 4 workflow JSONs from `Backend/` into your own n8n instance.
3. In n8n, create your own Supabase and Groq credentials and attach them to the relevant nodes (they are **not** included in the exported JSON).
4. Replace the placeholder values in the workflow JSONs:
   - `YOUR_SUPABASE_SERVICE_ROLE_KEY`
   - `YOUR_SUPABASE_SECRET_KEY`
   - `YOUR_PROJECT_REF.supabase.co`
   - `YOUR_N8N_INSTANCE_ID` (n8n regenerates this automatically on import — you can leave it as-is)
5. In `Frontend/admin.html` and `Frontend/shop.html`, replace `https://YOUR_N8N_INSTANCE` in the URL constants with your own n8n webhook base URL.
6. In `Frontend/admin-login.html`, replace the placeholder password (`CHANGE_ME_BEFORE_DEPLOY`) with your own, or better — replace this client-side check entirely with a real auth webhook before deploying anywhere public.

## ⚠️ Security notes

- `admin-login.html` is a **placeholder** login (hardcoded username/password checked client-side). It is not real authentication — do not deploy as-is.
- The admin product webhook (`admin-products`) currently has no server-side auth check beyond what your n8n instance enforces — add authentication before exposing it publicly.
- Cart state lives in `localStorage` until checkout; only the checkout workflow writes to the database.

## Tech Stack

- **Backend:** n8n (workflow automation), Groq (LLM), Supabase (Postgres + pgvector)
- **Frontend:** Plain HTML/CSS/JS, Tailwind CSS (CDN)
- **Fonts:** Clash Display, Satoshi
