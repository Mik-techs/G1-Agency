# G1 Creative Agency — Print Order System

A modern, mobile-first print order management platform for G1 Creative Agency. Features a customer-facing order form, public portfolio showcase, and staff dashboard for order management.

## 🚀 Features

### Customer-Facing
- **Interactive Order Form** (`print-order.html`)
  - Job category selection with subtypes (cards, t-shirts, stickers, etc.)
  - Copy quantity stepper
  - File upload or "bring your own" option
  - Dynamic question flow based on job type
  - Automatic ticket generation and WhatsApp integration
  - Real-time file upload to Catbox with progress tracking

- **Portfolio Gallery** (`portfolio.html`)
  - Public showcase of past work
  - Filter by job category
  - Lightbox image viewer
  - Direct link to order form

### Staff Dashboard
- **Order Management** (`dashboard-1.html`)
  - Real-time order list with 15-second auto-refresh
  - Status tracking (New → In Progress → Done)
  - Profile-aware field display (apparel, cards, banners, stickers, standard print)
  - Direct links to ticket images and uploaded files
  - Session-based authentication

- **Portfolio Management**
  - Add portfolio items with title, category, and image upload
  - Delete items
  - Reorder display
  - Real-time sync with public gallery

## 🏗️ Architecture

```
├── print-order.html          # Customer order form
├── portfolio.html            # Public portfolio gallery
├── dashboard-1.html          # Staff dashboard (orders + portfolio mgmt)
├── supabase-setup.sql        # Database schema & RLS policies
├── README.md                 # This file
├── .env.example              # Environment variable template
└── vercel.json              # Vercel deployment config
```

### Tech Stack
- **Frontend**: Vanilla JavaScript, HTML5, CSS3 (no build step)
- **Backend**: Supabase (PostgreSQL + Auth + RLS)
- **File Storage**: Catbox.moe (free file upload)
- **Communication**: WhatsApp Business API
- **Deployment**: Vercel (or any static host)

## 🔧 Setup Instructions

### 1. Clone & Configure

```bash
git clone https://github.com/Mik-techs/G1-Agency.git
cd G1-Agency
```

### 2. Set Up Supabase

1. Create a [Supabase](https://supabase.com) project
2. Go to **SQL Editor** → **New query**
3. Copy and paste the entire contents of `supabase-setup.sql`
4. Click **Run**
5. Go to **Settings** → **API** and note:
   - `Project URL` (SUPABASE_URL)
   - `anon public` key (SUPABASE_ANON_KEY)

### 3. Create Staff Login

1. In Supabase: **Authentication** → **Users** → **Add user**
2. Enter email and password for the shop owner
3. This is the dashboard login (no public sign-up)

### 4. Environment Configuration

Create a `.env.local` file (or set in deployment platform):

```env
# Supabase
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key-here

# WhatsApp
VITE_SHOP_WHATSAPP=233546850197  # Include country code, no + symbol

# File Upload
VITE_UPLOAD_ENDPOINT=https://catbox.moe/user/api.php

# Order Form URL (for portfolio gallery)
VITE_ORDER_FORM_URL=./print-order.html
```

### 5. Update HTML Data Attributes

Each HTML file has `data-*` attributes in the root element. Update these with your actual values:

**`print-order.html`**
```html
<div class="ticket"
     data-shop-whatsapp="233546850197"
     data-upload-endpoint="https://catbox.moe/user/api.php"
     data-supabase-url="https://your-project.supabase.co"
     data-supabase-anon-key="your-anon-key">
```

**`portfolio.html` & `dashboard-1.html`**
```html
<body data-supabase-url="https://your-project.supabase.co"
      data-supabase-anon-key="your-anon-key"
      data-order-form-url="./print-order.html">
```

### 6. Configure Color Scheme

Each file has CSS custom properties. Customize in `<style>`:

```css
:root {
  --ink: #1c2b3a;           /* Primary text */
  --paper: #f7f5f0;         /* Background */
  --stub: #ffffff;          /* Card backgrounds */
  --amber: #c9a227;         /* Primary accent */
  --gold-dark: #8a6d1a;     /* Dark accent */
  --green: #3a7d44;         /* Success/action color */
  --line: #d8d2c4;          /* Borders */
  --muted: #6b7280;         /* Secondary text */
}
```

## 📦 Deployment

### Option A: Vercel (Recommended)

1. Push to GitHub
2. Go to [Vercel](https://vercel.com)
3. Click **New Project** → select this repo
4. Add environment variables (from `.env.example`)
5. Deploy

### Option B: Netlify

1. Connect GitHub repo
2. Set build command: (leave empty — static site)
3. Set publish directory: `.` (root)
4. Add environment variables
5. Deploy

### Option C: Self-Hosted

1. Copy files to any web server
2. Set environment variables via HTML data attributes
3. Serve over HTTPS

## 🔐 Security

### Row-Level Security (RLS)
- **Customers** can only INSERT orders (via public `anon` key)
- **Customers** cannot read, update, or delete orders
- **Staff** can only access dashboard with valid login
- Portfolio is read-only to public, write-restricted to authenticated staff

### WhatsApp Integration
- Orders are sent via WhatsApp, not stored in a separate delivery channel
- Ticket images auto-generate and upload to Catbox
- Failed uploads don't block submission (fallback to text summary)

### File Uploads
- User files uploaded to Catbox (external, ephemeral storage)
- Images generated server-side as canvas → blob → upload
- No files stored on the origin server

## 📊 Database Schema

### `orders` table
```sql
id              uuid (primary key)
order_code      text (e.g. "PR-260827-4521")
created_at      timestamptz
job             text (e.g. "Photocopying")
copies          int
profile         text (standardPrint | card | banner | stickers | apparel)
color           text (Color | Black & White)
paper_size      text (A4 | A3 | Not sure)
finishing       text (Plain | Stapled | Spiral bound | Laminated)
card_finish     text (Matte | Glossy | Laminated | Not sure)
banner_finish   text (Eyelets | Hemmed edges | Pole pocket | Not sure)
sticker_finish  text (Glossy | Matte | Waterproof | Not sure)
shirt_color     text (White | Black | Other color | Not sure)
shirt_size      text (S | M | L | XL | Mixed sizes | Not sure)
print_placement text (Front | Back | Front and Back | Not sure)
customer_name   text
bring_own       boolean
file_name       text
file_link       text
ticket_link     text
status          text (new | in_progress | done)
```

### `portfolio_items` table
```sql
id              uuid (primary key)
created_at      timestamptz
title           text
category        text (standardPrint | card | banner | stickers | apparel | other)
image_url       text
display_order   int
```

## 🎨 Customization

### Job Categories
Edit the `JOB_CATALOG` in `print-order.html`:
```javascript
const JOB_CATALOG = [
  { key:'photocopy', label:'Photocopy', icon:'icon-photocopy', profile:'standardPrint', job:'Photocopying' },
  { key:'card', label:'Card', icon:'icon-card', profile:'card', subtypes:[...] },
  // Add more...
];
```

### Icons
All icons are inline SVG symbols (no image files). Add new ones in the `<svg>` block and reference via `<use href="#icon-name">`.

### WhatsApp Message Format
Edit `buildMessage()` in `print-order.html` to customize the order summary sent to WhatsApp.

## 🐛 Troubleshooting

| Issue | Solution |
|-------|----------|
| "Couldn't upload file" | Check Catbox endpoint, ensure files are under 50MB |
| Orders not appearing | Verify Supabase credentials, check RLS policies in dashboard |
| Dashboard login fails | Confirm user exists in Supabase Authentication, check token expiry |
| Ticket image blank | Ensure canvas API is available, check CORS settings |
| WhatsApp link not opening | Verify phone number format (country code without +) |

## 📱 Mobile Responsiveness

All pages are mobile-first and tested on:
- iPhone SE and larger
- Android (Chrome, Samsung Internet)
- iPad and tablets

The order form dynamically sizes to fit screen width (max 420px on desktop).

## ✅ Pre-Deployment Checklist

- [ ] Supabase project created and RLS policies applied
- [ ] Staff login user created in Supabase Auth
- [ ] Environment variables set (Supabase URL, anon key, WhatsApp number)
- [ ] HTML data attributes updated with actual values
- [ ] Color scheme customized (if needed)
- [ ] Job categories and icons reviewed
- [ ] WhatsApp Business number verified
- [ ] Test order submitted end-to-end
- [ ] Dashboard login tested
- [ ] Portfolio images ready (if launching with samples)
- [ ] Domain configured (if using custom domain)
- [ ] HTTPS enabled (auto with Vercel/Netlify)

## 📞 Support

For issues or questions:
1. Check the troubleshooting section above
2. Review Supabase documentation: https://supabase.com/docs
3. Contact support via the form's "Talk to us" WhatsApp link

## 📄 License

This project is proprietary to G1 Creative Agency. All rights reserved.

---

**Version:** 1.0  
**Last Updated:** August 2026  
**Created for:** G1 Creative Agency, Sunyani
