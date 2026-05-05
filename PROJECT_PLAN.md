# Falcon Courier Management System — Project Plan

## Overview

This document outlines the architecture, database schema, and implementation strategy for the Falcon Courier Management System. The application combines a public-facing website with an internal operations platform, built using React 19, Express 4, tRPC 11, and MySQL via Drizzle ORM.

## Design Philosophy

The visual design follows the **International Typographic Style**, emphasizing:

- **Pristine white canvas** as the primary background
- **Bold red square accents** for visual hierarchy and call-to-action elements
- **Crisp black sans-serif typography** (system fonts for performance)
- **Strict yet dynamic grid system** with mathematical precision
- **Fine black divider lines** and generous negative space
- **Functional, structured, and timelessly modern** aesthetic

Color palette:
- Primary: Red (#DC143C for accents, #B22222 for hover states)
- Secondary: Black (#1A1A1A for text)
- Tertiary: White (#FFFFFF for backgrounds)
- Neutral: Light gray (#F5F5F5 for subtle backgrounds)

## Database Schema

### Tables

#### `users`
Stores staff and admin accounts with role-based access control.

| Column | Type | Notes |
|--------|------|-------|
| id | INT PRIMARY KEY | Auto-increment |
| openId | VARCHAR(64) UNIQUE | Manus OAuth identifier |
| name | TEXT | User's full name |
| email | VARCHAR(320) | User's email address |
| role | ENUM('user', 'admin', 'courier', 'staff') | Access level |
| department | VARCHAR(100) | e.g., "Delivery", "Admin", "Support" |
| createdAt | TIMESTAMP | Account creation |
| updatedAt | TIMESTAMP | Last update |
| lastSignedIn | TIMESTAMP | Last login |

#### `shipments`
Core shipment records with tracking information.

| Column | Type | Notes |
|--------|------|-------|
| id | INT PRIMARY KEY | Auto-increment |
| trackingNumber | VARCHAR(50) UNIQUE | Public tracking ID |
| senderName | VARCHAR(255) | Sender's name |
| senderAddress | TEXT | Sender's full address |
| recipientName | VARCHAR(255) | Recipient's name |
| recipientAddress | TEXT | Recipient's full address |
| weight | DECIMAL(10, 2) | Package weight in kg |
| dimensions | VARCHAR(100) | e.g., "30x20x15 cm" |
| status | ENUM('pending', 'picked_up', 'in_transit', 'out_for_delivery', 'delivered', 'failed', 'returned') | Current status |
| estimatedDelivery | DATE | Expected delivery date |
| actualDelivery | DATE | Actual delivery date (NULL if not delivered) |
| assignedCourier | INT | Foreign key to users.id |
| createdAt | TIMESTAMP | Shipment creation |
| updatedAt | TIMESTAMP | Last update |

#### `trackingEvents`
Audit trail of all status changes with timestamps and responsible staff.

| Column | Type | Notes |
|--------|------|-------|
| id | INT PRIMARY KEY | Auto-increment |
| shipmentId | INT FOREIGN KEY | Reference to shipments.id |
| status | VARCHAR(50) | Status at this event |
| location | VARCHAR(255) | Geographic location or facility |
| notes | TEXT | Additional details or notes |
| updatedBy | INT FOREIGN KEY | Reference to users.id (staff member) |
| createdAt | TIMESTAMP | Event timestamp |

#### `inquiries`
Contact form submissions from the public website.

| Column | Type | Notes |
|--------|------|-------|
| id | INT PRIMARY KEY | Auto-increment |
| name | VARCHAR(255) | Sender's name |
| email | VARCHAR(320) | Sender's email |
| phone | VARCHAR(20) | Sender's phone |
| subject | VARCHAR(255) | Inquiry subject |
| message | TEXT | Inquiry message |
| status | ENUM('new', 'read', 'responded') | Processing status |
| createdAt | TIMESTAMP | Submission time |

## Page Structure & Routes

### Public Pages

| Route | Component | Purpose |
|-------|-----------|---------|
| `/` | Home | Landing page with hero, services overview, CTA |
| `/track` | Tracking | Public shipment tracking interface |
| `/about` | About | Company information and mission |
| `/services` | Services | Service offerings and pricing |
| `/contact` | Contact | Contact form and inquiry submission |

### Staff Portal (Protected: role = 'courier' or 'staff')

| Route | Component | Purpose |
|-------|-----------|---------|
| `/staff/dashboard` | StaffDashboard | Overview of assigned shipments |
| `/staff/shipments` | StaffShipments | List and manage assigned shipments |
| `/staff/shipment/:id` | ShipmentDetail | Update delivery status, add notes |

### Admin Dashboard (Protected: role = 'admin')

| Route | Component | Purpose |
|-------|-----------|---------|
| `/admin/dashboard` | AdminDashboard | Analytics, KPIs, system overview |
| `/admin/shipments` | AdminShipments | Full shipment management (CRUD) |
| `/admin/shipment/:id` | AdminShipmentDetail | Edit, assign, delete shipments |
| `/admin/staff` | StaffManagement | Manage staff accounts and roles |
| `/admin/logs` | AuditLogs | View all tracking events and changes |
| `/admin/inquiries` | InquiriesManagement | View and respond to contact inquiries |

## tRPC Procedures

### Public Procedures

```
trpc.shipments.getByTrackingNumber(trackingNumber: string)
  → Returns shipment + full tracking event history

trpc.inquiries.submit(name, email, phone, subject, message)
  → Submits contact form inquiry
```

### Staff Procedures (Protected)

```
trpc.staff.getAssignedShipments()
  → Returns shipments assigned to current courier

trpc.staff.updateShipmentStatus(shipmentId, status, location, notes)
  → Updates shipment status and creates tracking event
```

### Admin Procedures (Protected)

```
trpc.admin.shipments.list(filters?)
  → Returns all shipments with pagination

trpc.admin.shipments.create(shipmentData)
  → Creates new shipment

trpc.admin.shipments.update(id, shipmentData)
  → Updates shipment details

trpc.admin.shipments.delete(id)
  → Deletes shipment

trpc.admin.shipments.assign(id, courierId)
  → Assigns shipment to courier

trpc.admin.staff.list()
  → Returns all staff members

trpc.admin.staff.update(id, userData)
  → Updates staff member details

trpc.admin.logs.list(filters?)
  → Returns all tracking events with pagination

trpc.admin.analytics.getMetrics()
  → Returns delivery metrics, on-time rates, etc.
```

## Seed Data Strategy

The database will be pre-populated with realistic dummy data:

- **5 staff members** with roles: 2 couriers, 2 staff, 1 admin
- **20 shipments** with varied statuses (pending, in-transit, delivered, failed)
- **50+ tracking events** showing realistic status progression
- **5 sample inquiries** in the contact form submissions

This enables immediate demo use without manual data entry.

## Component Architecture

### Layout Components

- **PublicLayout**: Header with logo, navigation, footer (public pages)
- **StaffLayout**: Sidebar navigation, authenticated header (staff portal)
- **AdminLayout**: Full dashboard layout with sidebar (admin panel)

### Shared Components

- **ShipmentCard**: Displays shipment summary
- **TrackingTimeline**: Visualizes tracking event history
- **StatusBadge**: Color-coded status indicator
- **FormInput**: Reusable form field with validation
- **DataTable**: Paginated table for lists

### Feature Components

- **TrackingForm**: Input field for tracking number lookup
- **ShipmentForm**: Create/edit shipment modal
- **StatusUpdateForm**: Update delivery status
- **ContactForm**: Inquiry submission form

## Implementation Phases

1. **Database & Backend**: Schema, migrations, seed data, tRPC procedures
2. **Public Pages**: Landing, tracking, about, services, contact
3. **Staff Portal**: Dashboard, shipment list, status updates
4. **Admin Dashboard**: Full CRUD, staff management, analytics, logs
5. **Styling & Responsive Design**: Apply International Typographic Style, mobile optimization
6. **Testing & Deployment**: Vitest coverage, checkpoint, deployment guide

## Key Decisions

- **Authentication**: Manus OAuth for staff/admin; public tracking requires no login
- **Styling**: Tailwind CSS 4 with custom design tokens for red/black/white theme
- **Data Persistence**: MySQL with Drizzle ORM for type-safe queries
- **Real-time Updates**: tRPC mutations with optimistic updates for status changes
- **Responsive Design**: Mobile-first approach with breakpoints at 640px, 1024px, 1280px

## Success Criteria

- All pages render correctly on desktop and mobile
- Public tracking works without authentication
- Staff can update shipment statuses
- Admin can perform full CRUD operations
- Audit trail captures all changes with timestamps
- Contact form submissions are stored and retrievable
- Design adheres to International Typographic Style principles
- Application is ready for immediate demo use with seed data
