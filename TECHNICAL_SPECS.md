# The Lazy App - Technical Specifications

## System Architecture Overview

```
┌─────────────────┐         ┌──────────────────┐         ┌─────────────────┐
│   Frontend      │         │   Backend API    │         │   Firebase      │
│  (React/Native) │◄───────►│   (Node.js)      │◄───────►│  (Auth & DB)    │
└─────────────────┘         └──────────────────┘         └─────────────────┘
                                     │
                                     │
                            ┌────────▼────────┐
                            │  Stripe API     │
                            │  (Payments)     │
                            └─────────────────┘
```

## Technology Stack

### Backend
- **Runtime**: Node.js v18+
- **Framework**: Express.js / Fastify
- **Database**: Firebase Firestore
- **Authentication**: Firebase Auth
- **Payment Processing**: Stripe API
- **Real-time Messaging**: Firebase Realtime DB / Socket.io
- **API Documentation**: OpenAPI/Swagger

### Frontend (Web)
- **Framework**: React 18+
- **State Management**: Redux Toolkit / Zustand
- **Styling**: Tailwind CSS (Green palette)
- **HTTP Client**: Axios / Fetch API
- **Maps**: Google Maps API (ZIP code geolocation)

### Mobile
- **Framework**: React Native
- **State Management**: Redux Toolkit / Zustand
- **Maps**: React Native Maps

### DevOps & Infrastructure
- **Hosting**: Firebase Hosting / Google Cloud Run
- **CI/CD**: GitHub Actions
- **Version Control**: Git/GitHub
- **Monitoring**: Firebase Crashlytics / Sentry

## Database Schema

### Users Collection
```javascript
{
  id: string (UUID),
  email: string,
  phone: string,
  role: enum ["employer", "gig_worker"],
  profile: {
    firstName: string,
    lastName: string,
    avatar: string (URL),
    bio: string,
    zipCode: string,
    latitude: number,
    longitude: number,
    verificationStatus: enum ["pending", "verified", "rejected"],
    verificationDocument: string (URL)
  },
  ratings: {
    averageRating: number (1-5),
    totalReviews: number,
    asEmployer?: {
      averageRating: number,
      totalReviews: number
    },
    asWorker?: {
      averageRating: number,
      totalReviews: number
    }
  },
  financials: {
    stripeCustomerId: string,
    stripeAccountId: string (for workers),
    bankAccount?: object,
    pendingBalance: number,
    totalEarnings: number (for workers)
  },
  preferences: {
    categoriesInterested: array[string],
    radiusInMiles: number,
    notificationsEnabled: boolean
  },
  createdAt: timestamp,
  updatedAt: timestamp
}
```

### Jobs Collection
```javascript
{
  id: string (UUID),
  employerId: string,
  title: string,
  description: string,
  category: string,
  status: enum ["open", "in_progress", "completed", "cancelled"],
  pricing: {
    type: enum ["fixed", "bidding"],
    amount: number,
    currency: string
  },
  location: {
    zipCode: string,
    latitude: number,
    longitude: number,
    address: string
  },
  scheduling: {
    preferredDate: timestamp,
    preferredTime: string,
    estimatedDuration: number (minutes)
  },
  assignedWorkerId?: string,
  bids: array[{
    workerId: string,
    bidAmount: number,
    message: string,
    status: enum ["pending", "accepted", "rejected"],
    createdAt: timestamp
  }],
  timeline: {
    createdAt: timestamp,
    startedAt?: timestamp,
    completedAt?: timestamp,
    updatedAt: timestamp
  },
  images: array[string] (URLs)
}
```

### Ratings Collection
```javascript
{
  id: string (UUID),
  jobId: string,
  ratedBy: string (userId),
  ratedUser: string (userId),
  role: enum ["employer_rating_worker", "worker_rating_employer"],
  rating: number (1-5),
  review: string,
  categories: {
    communication: number,
    professionalism: number,
    quality: number,
    timeliness: number
  },
  createdAt: timestamp
}
```

### Transactions Collection
```javascript
{
  id: string (UUID),
  jobId: string,
  employerId: string,
  workerId: string,
  amount: number,
  fee: number (15% of amount),
  tip: number,
  status: enum ["pending", "processing", "completed", "failed", "refunded"],
  paymentMethod: {
    type: enum ["card", "bank_transfer"],
    last4: string
  },
  stripePaymentIntentId: string,
  timeline: {
    createdAt: timestamp,
    processedAt?: timestamp,
    completedAt?: timestamp
  }
}
```

### Messages Collection
```javascript
{
  id: string (UUID),
  conversationId: string,
  senderId: string,
  recipientId: string,
  jobId: string,
  message: string,
  attachments?: array[{
    type: enum ["image", "document"],
    url: string,
    name: string
  }],
  read: boolean,
  createdAt: timestamp
}
```

### Hour Logs Collection
```javascript
{
  id: string (UUID),
  workerId: string,
  jobId: string,
  date: date,
  startTime: time,
  endTime: time,
  totalHours: number,
  notes: string,
  status: enum ["draft", "submitted", "verified"],
  createdAt: timestamp,
  updatedAt: timestamp
}
```

## API Endpoints

### Authentication
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - Login user
- `POST /api/auth/logout` - Logout user
- `POST /api/auth/verify-email` - Verify email address
- `POST /api/auth/forgot-password` - Request password reset

### Users
- `GET /api/users/:id` - Get user profile
- `PUT /api/users/:id` - Update user profile
- `GET /api/users/:id/ratings` - Get user ratings
- `POST /api/users/:id/verify-identity` - Upload verification documents

### Jobs
- `GET /api/jobs` - List jobs (with filters: zipCode, category, status)
- `POST /api/jobs` - Create new job
- `GET /api/jobs/:id` - Get job details
- `PUT /api/jobs/:id` - Update job
- `DELETE /api/jobs/:id` - Cancel job
- `POST /api/jobs/:id/bids` - Submit bid
- `PUT /api/jobs/:id/bids/:bidId` - Accept/reject bid

### Messaging
- `GET /api/conversations` - List conversations
- `GET /api/conversations/:conversationId/messages` - Get messages
- `POST /api/conversations/:conversationId/messages` - Send message

### Ratings & Reviews
- `POST /api/ratings` - Submit rating
- `GET /api/ratings/user/:userId` - Get user ratings

### Transactions & Payments
- `GET /api/transactions` - List transactions
- `POST /api/transactions/:jobId/process` - Process payment
- `POST /api/transactions/:jobId/tip` - Add tip
- `GET /api/payouts` - Get payout history
- `POST /api/payouts/request` - Request payout

### Hour Logs
- `GET /api/hours` - Get hour logs
- `POST /api/hours` - Create hour log entry
- `PUT /api/hours/:id` - Update hour log
- `GET /api/hours/:id/export-pdf` - Export as PDF

### AI Concierge
- `POST /api/concierge/chat` - Send message to AI chatbot
- `GET /api/concierge/suggestions` - Get service suggestions

## Security Considerations

### Authentication & Authorization
- Firebase Authentication for secure login
- JWT tokens for API calls
- Role-based access control (RBAC)
- Session timeout after 30 minutes of inactivity

### Data Protection
- End-to-end encryption for messaging
- SSL/TLS for all API communications
- PCI DSS compliance for payment data
- GDPR compliance for user data

### Verification & Trust
- Identity verification before first transaction
- Background checks for gig workers (future enhancement)
- Phone number verification
- Email verification

## Performance & Scalability

### Caching Strategy
- Redis for session management
- Browser caching for static assets
- CDN for image delivery

### Database Optimization
- Firestore indexes on frequently queried fields
- Query pagination (limit 20-50 results)
- Real-time listener optimization

### Rate Limiting
- API rate limiting: 100 requests per minute per user
- Payment API calls: 10 per second

## Testing Strategy

### Unit Tests
- Jest for backend and frontend
- 80%+ code coverage target

### Integration Tests
- Supertest for API testing
- Firebase emulator for local testing

### E2E Tests
- Cypress for web application
- Detox for mobile application

## Deployment & CI/CD

### GitHub Actions Workflows
1. **PR Checks**: Lint, unit tests, code coverage
2. **Staging Deploy**: Auto-deploy to staging on main branch
3. **Production Deploy**: Manual approval required

### Environments
- **Development**: Local Firebase emulator
- **Staging**: Firebase test project
- **Production**: Firebase production project

## Monitoring & Analytics

### Metrics
- User acquisition and retention
- Job completion rates
- Average rating trends
- Revenue and transaction volume
- API response times
- Error rates

### Tools
- Firebase Analytics
- Sentry for error tracking
- Google Analytics for web
- Amplitude for user behavior
