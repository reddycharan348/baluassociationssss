# Balu Associates - Tax & GST Solutions

Complete website with secure client portal for file sharing.

## 🌟 Features

### Main Website
- Professional business website
- Services showcase
- GST Calculator
- Consultation booking
- Contact forms
- Testimonials
- Mobile responsive

### Client Portal
- **Admin Dashboard**: Manage companies, upload files, send notifications
- **Company Dashboard**: View files, download documents, request files
- **File Management**: Upload to Google Drive, preview PDFs/images
- **Activity Tracking**: Complete audit log
- **Notifications**: Send messages to companies
- **Security**: JWT auth, session timeout, password strength

## 📁 Project Structure

```
BaluAssociates/
├── index.html              # Main website
├── client-portal.html      # Portal interface
├── portal-app.js           # Portal logic
├── portal-api.js           # API client
├── styles.css              # Main styles
├── portal-styles.css       # Portal styles
├── script.js               # Main website JS
├── backend/                # Node.js API
│   ├── config/            # Supabase + Google Drive
│   ├── routes/            # API endpoints
│   └── server.js          # Express server
├── DEPLOYMENT_GUIDE.md    # Step-by-step setup
├── QUICK_START.md         # 5-minute guide
└── README.md              # This file
```

## 🚀 Quick Start

```bash
# 1. Backend
cd backend
npm install
cp .env.example .env
# Edit .env with your credentials
npm run dev

# 2. Frontend (new terminal)
python3 -m http.server 8000

# 3. Open browser
http://localhost:8000/client-portal.html
```

**Login:**
- Admin: `admin@baluassociates.net` / `admin123`

## 📖 Documentation

- **DEPLOYMENT_GUIDE.md** - Complete setup instructions
- **QUICK_START.md** - Get running in 5 minutes
- **backend/SETUP_GUIDE.md** - Backend configuration
- **backend/README.md** - API documentation
- **PORTAL_FEATURES_GUIDE.md** - Feature list

## 🛠️ Technology Stack

**Frontend:**
- HTML5, CSS3, JavaScript
- Font Awesome icons
- Responsive design

**Backend:**
- Node.js + Express
- Supabase (PostgreSQL + Auth)
- Google Drive API
- JWT authentication

## 📞 Contact

**Balu Associates**
- Email: baluassociates.net@gmail.com
- Phone: +91 9535725179
- Address: Bengaluru, Karnataka 560068

## 📄 License

© 2025 Balu Associates. All rights reserved.
