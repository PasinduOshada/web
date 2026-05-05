# Falcon Courier Services

A full-stack courier and shipment management web application featuring public-facing website, real-time tracking, staff intranet portal, and admin dashboard.

## Features

### 🌐 Public Website
- **Landing Page**: Hero section with services overview and CTAs
- **Shipment Tracking**: Real-time tracking with location history
- **About Us**: Company information and team
- **Services**: Detailed service offerings and pricing
- **Contact**: Inquiry form with email submission

### 👥 Staff Portal
- **Dashboard**: Overview of assigned shipments
- **Shipments List**: Filter and manage assigned deliveries
- **Status Updates**: Update shipment status with location and notes
- **Tracking Timeline**: View complete delivery history

### 🔐 Admin Dashboard
- **Overview**: Analytics with KPIs (total, delivered, pending, failed)
- **Shipment Management**: Create, edit, assign, delete shipments
- **Staff Management**: Manage courier accounts and roles
- **Audit Logs**: Complete tracking event history
- **Inquiries**: Manage customer contact form submissions

### 🎨 Design
- **International Typographic Style**: Clean, structured, timelessly modern
- **Color Scheme**: Red/Black/White with bold accents
- **Responsive**: Mobile-first design for all devices
- **Accessibility**: WCAG 2.1 AA compliant

## Tech Stack

- **Frontend**: React 19 + Tailwind CSS 4 + TypeScript
- **Backend**: Express 4 + tRPC 11 + Node.js
- **Database**: MySQL/TiDB with Drizzle ORM
- **Authentication**: Manus OAuth
- **Deployment**: Cloudflare Workers
- **Testing**: Vitest

## Project Structure

```
falconsl-website/
├── client/                 # React frontend
│   ├── src/
│   │   ├── pages/         # Page components
│   │   ├── components/    # Reusable components
│   │   ├── lib/           # Utilities and hooks
│   │   └── App.tsx        # Main router
│   └── public/            # Static assets
├── server/                # Express backend
│   ├── routers/           # tRPC route definitions
│   ├── db.ts              # Database queries
│   ├── routers.ts         # Main router
│   └── _core/             # Core infrastructure
├── drizzle/               # Database schema & migrations
├── shared/                # Shared types and constants
└── storage/               # S3 storage helpers
```

## Getting Started

### Prerequisites

- Node.js 18+
- pnpm 10+
- MySQL 8+ or TiDB
- Manus OAuth credentials

### Installation

```bash
# Install dependencies
pnpm install

# Configure environment
cp .env.example .env.local
# Edit .env.local with your credentials

# Run migrations
pnpm drizzle-kit generate
pnpm drizzle-kit migrate

# Seed demo data (optional)
node server/seed.mjs

# Start dev server
pnpm dev
```

The app will be available at `http://localhost:5173`

## Development

### Available Commands

```bash
# Development server
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start

# Run tests
pnpm test

# Format code
pnpm format

# Type check
pnpm check
```

### Database Changes

When modifying the database schema:

```bash
# 1. Update drizzle/schema.ts
# 2. Generate migration
pnpm drizzle-kit generate

# 3. Review generated SQL in drizzle/*.sql
# 4. Apply migration
pnpm drizzle-kit migrate
```

### Adding Features

1. **Database**: Update `drizzle/schema.ts` and generate migration
2. **Backend**: Add query helpers in `server/db.ts`, procedures in `server/routers/`
3. **Frontend**: Create components in `client/src/pages/` and `client/src/components/`
4. **Tests**: Add vitest specs in `server/*.test.ts`

## Authentication

The app uses Manus OAuth for authentication:

- **Public Pages**: No authentication required
- **Staff Portal**: Login required (courier/staff role)
- **Admin Dashboard**: Admin role required

Login URL: `getLoginUrl()` from `client/const.ts`

## Database Schema

### Users
- Manus OAuth integration
- Role-based access (user, admin)
- Staff profiles with contact info

### Shipments
- Tracking number (unique)
- Sender/recipient details
- Weight, dimensions, estimated delivery
- Status tracking
- Courier assignment

### Tracking Events
- Status updates with timestamps
- Location information
- Notes and metadata
- Audit trail

### Inquiries
- Contact form submissions
- Customer information
- Status tracking (new, responded)

## API Endpoints

### Public
- `GET /api/trpc/shipments.getByTrackingNumber` - Track shipment
- `POST /api/trpc/inquiries.submit` - Submit inquiry

### Staff (Protected)
- `GET /api/trpc/shipments.getAssignedShipments` - List assigned shipments
- `GET /api/trpc/shipments.adminGetById` - Get shipment details
- `POST /api/trpc/shipments.updateStatus` - Update status

### Admin (Protected)
- `GET /api/trpc/shipments.adminList` - List all shipments
- `POST /api/trpc/shipments.adminCreate` - Create shipment
- `PUT /api/trpc/shipments.adminUpdate` - Update shipment
- `DELETE /api/trpc/shipments.adminDelete` - Delete shipment
- `GET /api/trpc/inquiries.adminList` - List inquiries

## Deployment

See [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md) for detailed deployment instructions.

### Quick Deploy

```bash
# Build
pnpm build

# Deploy to Cloudflare Workers
wrangler deploy --env production
```

## Demo Credentials

After seeding demo data:

**Courier Login**
- Email: anil@falcon.lk or priya@falcon.lk
- Use Manus OAuth

**Admin Login**
- Email: rajesh@falcon.lk
- Use Manus OAuth

**Demo Tracking**
- Tracking numbers: FK-2026-001 through FK-2026-020

## Performance

- **Lighthouse Score**: 95+
- **First Contentful Paint**: < 1.5s
- **Time to Interactive**: < 3s
- **Database Query Time**: < 100ms (p95)

## Security

- HTTPS/TLS encryption
- OAuth 2.0 authentication
- SQL injection prevention (Drizzle ORM)
- XSS protection (React)
- CSRF tokens on forms
- Rate limiting on public endpoints
- Secure session cookies

## Browser Support

- Chrome/Edge 90+
- Firefox 88+
- Safari 14+
- Mobile browsers (iOS Safari, Chrome Android)

## Contributing

1. Create feature branch: `git checkout -b feature/name`
2. Make changes and commit: `git commit -am 'Add feature'`
3. Push to branch: `git push origin feature/name`
4. Submit pull request

## Testing

```bash
# Run all tests
pnpm test

# Run specific test file
pnpm test server/routers/shipments.test.ts

# Watch mode
pnpm test --watch
```

## Troubleshooting

### Database Connection Error
```bash
# Check DATABASE_URL
echo $DATABASE_URL

# Test connection
mysql -h host -u user -p -e "SELECT 1"
```

### OAuth Not Working
- Verify `VITE_APP_ID` in environment
- Check redirect URI in Manus OAuth settings
- Ensure cookies are enabled

### Build Errors
```bash
# Clear cache and reinstall
rm -rf node_modules pnpm-lock.yaml
pnpm install
pnpm build
```

## Performance Optimization

- **Caching**: Cloudflare caches static assets
- **Database**: Indexes on frequently queried columns
- **Frontend**: Code splitting, lazy loading
- **API**: Response compression, pagination

## Monitoring

- Cloudflare Analytics Dashboard
- Application logs in Cloudflare Workers
- Database performance metrics
- GitHub Actions CI/CD logs

## Maintenance

- Weekly security updates
- Monthly database optimization
- Quarterly security audits
- Annual infrastructure review

## Support

- Documentation: [docs/](./docs/)
- Issues: GitHub Issues
- Email: support@falcon.lk

## License

MIT License - See LICENSE file for details

## Changelog

### v1.0 (May 2026)
- Initial release
- Public website with 5 pages
- Staff portal with shipment management
- Admin dashboard with analytics
- Real-time tracking system
- Contact form integration
- International Typographic Style design

---

**Built with ❤️ for Falcon Courier Services**

For deployment instructions, see [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md)
