# Backend Setup Guide - Supabase + Google Drive

## Prerequisites
- Node.js (v16 or higher)
- Supabase account
- Google Cloud account

## Step 1: Install Dependencies

```bash
cd backend
npm install
```

## Step 2: Set Up Supabase

### 2.1 Create Supabase Project
1. Go to [supabase.com](https://supabase.com)
2. Create a new project
3. Note down:
   - Project URL
   - Anon key
   - Service role key

### 2.2 Create Database Tables

Run this SQL in Supabase SQL Editor:

```sql
-- Enable UUID extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Note: Make sure to run this entire SQL script in one go

-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
    email TEXT UNIQUE NOT NULL,
    role TEXT NOT NULL CHECK (role IN ('admin', 'company')),
    company_id UUID,
    company_name TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Companies table
CREATE TABLE companies (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    phone TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Files table
CREATE TABLE files (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    company_id UUID REFERENCES companies(id) ON DELETE CASCADE,
    name TEXT NOT NULL,
    type TEXT NOT NULL,
    size BIGINT NOT NULL,
    category TEXT NOT NULL,
    expiry_date DATE,
    storage_path TEXT NOT NULL,
    uploaded_by UUID REFERENCES users(id),
    uploaded_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    read_by UUID[] DEFAULT '{}'
);

-- Notifications table
CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    company_id UUID REFERENCES companies(id) ON DELETE CASCADE,
    subject TEXT NOT NULL,
    message TEXT NOT NULL,
    sent_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    read BOOLEAN DEFAULT FALSE
);

-- Document requests table
CREATE TABLE document_requests (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    company_id UUID REFERENCES companies(id) ON DELETE CASCADE,
    doc_type TEXT NOT NULL,
    description TEXT NOT NULL,
    status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'completed')),
    requested_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    completed_at TIMESTAMP WITH TIME ZONE
);

-- Activity log table
CREATE TABLE activity_log (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    action TEXT NOT NULL,
    details TEXT NOT NULL,
    timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create indexes
CREATE INDEX idx_files_company_id ON files(company_id);
CREATE INDEX idx_files_expiry_date ON files(expiry_date);
CREATE INDEX idx_notifications_company_id ON notifications(company_id);
CREATE INDEX idx_requests_company_id ON document_requests(company_id);
CREATE INDEX idx_activity_user_id ON activity_log(user_id);
CREATE INDEX idx_activity_timestamp ON activity_log(timestamp DESC);

-- Enable Row Level Security
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE companies ENABLE ROW LEVEL SECURITY;
ALTER TABLE files ENABLE ROW LEVEL SECURITY;
ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;
ALTER TABLE document_requests ENABLE ROW LEVEL SECURITY;
ALTER TABLE activity_log ENABLE ROW LEVEL SECURITY;

-- RLS Policies for users
CREATE POLICY "Users can view own data" ON users
    FOR SELECT USING (auth.uid() = id);

-- RLS Policies for companies
CREATE POLICY "Companies can view own data" ON companies
    FOR SELECT USING (
        id IN (SELECT company_id FROM users WHERE id = auth.uid())
    );

-- RLS Policies for files
CREATE POLICY "Companies can view own files" ON files
    FOR SELECT USING (company_id IN (
        SELECT company_id FROM users WHERE id = auth.uid()
    ));

-- RLS Policies for notifications
CREATE POLICY "Companies can view own notifications" ON notifications
    FOR SELECT USING (company_id IN (
        SELECT company_id FROM users WHERE id = auth.uid()
    ));

-- RLS Policies for document requests
CREATE POLICY "Companies can view own requests" ON document_requests
    FOR SELECT USING (company_id IN (
        SELECT company_id FROM users WHERE id = auth.uid()
    ));

CREATE POLICY "Companies can create requests" ON document_requests
    FOR INSERT WITH CHECK (company_id IN (
        SELECT company_id FROM users WHERE id = auth.uid()
    ));

-- RLS Policies for activity log
CREATE POLICY "Users can view own activity" ON activity_log
    FOR SELECT USING (user_id = auth.uid());
```

### 2.3 Create Admin User

Run this SQL to create admin user:

```sql
-- This will be done via Supabase Dashboard
-- Go to Authentication > Users > Add User
-- Email: admin@baluassociates.net
-- Password: admin123
-- Then run this SQL with the user ID:

INSERT INTO users (id, email, role, company_id, company_name)
VALUES (
    'YOUR_ADMIN_USER_ID_HERE',
    'admin@baluassociates.net',
    'admin',
    NULL,
    NULL
);
```

## Step 3: Set Up Supabase Storage

### 3.1 Create Storage Bucket
1. In Supabase dashboard, go to **Storage** (left sidebar)
2. Click **"New bucket"**
3. Bucket name: `company-files`
4. Make it **Public** (check the box)
5. Click **"Create bucket"**

### 3.2 Set Storage Policies
1. Click on the `company-files` bucket
2. Go to **"Policies"** tab
3. Click **"New Policy"**
4. Use this template:

**For SELECT (viewing files):**
```sql
CREATE POLICY "Allow authenticated users to view files"
ON storage.objects FOR SELECT
TO authenticated
USING (bucket_id = 'company-files');
```

**For INSERT (uploading files):**
```sql
CREATE POLICY "Allow authenticated users to upload files"
ON storage.objects FOR INSERT
TO authenticated
WITH CHECK (bucket_id = 'company-files');
```

**For DELETE (deleting files):**
```sql
CREATE POLICY "Allow authenticated users to delete files"
ON storage.objects FOR DELETE
TO authenticated
USING (bucket_id = 'company-files');
```

## Step 4: Configure Environment Variables

Create `.env` file in backend folder:

```env
PORT=5000
NODE_ENV=development

# Supabase
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_KEY=your-service-role-key

# File Upload
MAX_FILE_SIZE=10485760

# CORS
CORS_ORIGIN=http://localhost:8000

# Admin
ADMIN_EMAIL=admin@baluassociates.net
ADMIN_PASSWORD=admin123
```

## Step 5: Start the Server

```bash
npm run dev
```

Server will run on http://localhost:5000

## Step 6: Test API

### Health Check
```bash
curl http://localhost:5000/health
```

### Login
```bash
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@baluassociates.net","password":"admin123"}'
```

## API Endpoints

### Authentication
- POST `/api/auth/login` - Login
- POST `/api/auth/logout` - Logout
- POST `/api/auth/change-password` - Change password

### Companies (Admin only)
- GET `/api/companies` - Get all companies
- POST `/api/companies` - Create company
- PUT `/api/companies/:id` - Update company
- DELETE `/api/companies/:id` - Delete company

### Files
- GET `/api/files` - Get files
- POST `/api/files/upload` - Upload files (Admin)
- DELETE `/api/files/:id` - Delete file (Admin)
- POST `/api/files/:id/mark-read` - Mark as read (Company)

### Notifications
- GET `/api/notifications` - Get notifications
- POST `/api/notifications` - Send notification (Admin)

### Document Requests
- GET `/api/requests` - Get requests
- POST `/api/requests` - Create request (Company)
- PUT `/api/requests/:id` - Update request (Admin)

### Activity Log
- GET `/api/activity` - Get activity log

## Troubleshooting

### Database Connection Issues
- Check Supabase URL and keys
- Verify RLS policies are set correctly

### File Upload Issues
- Verify storage bucket is created and public
- Check storage policies are set correctly
- Ensure file size is within limits

### CORS Issues
- Update CORS_ORIGIN in .env
- Check frontend URL matches

## Production Deployment

1. Update environment variables
2. Use production Supabase project
3. Set NODE_ENV=production
4. Use proper domain for CORS
5. Enable HTTPS
6. Set up proper logging

---

**Backend is ready!** 🚀
