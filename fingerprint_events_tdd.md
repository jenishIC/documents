# Technical Design Documentation - Event Tracking SDK

## System Architecture

### 1. Components Overview

```
Event Tracking System
├── Client SDK (event-tracker.js)
│   ├── Core Tracking
│   ├── Session Management
│   └── Identity Management
├── Server API
│   ├── Event Processing
│   ├── Identity Stitching
│   └── Data Storage
└── FingerprintJS Integration
    ├── Visitor Identification
    └── Browser Fingerprinting
```

### 2. Data Flow

```
[Client Browser] → [FingerprintJS] → [Event Tracker SDK] → [Server API] → [Database]
     ↑                                      ↓
     └──────────────────[Cache/Storage]─────┘
```

## Component Details

### 1. Client SDK (event-tracker.js)

#### Initialization Flow
```javascript
1. Load SDK script
2. Initialize FingerprintJS
3. Get visitor identification
4. Create/resume session
5. Start automatic tracking
```

#### Session Management
- Uses crypto-secure session IDs
- Maintains session in localStorage
- Auto-renews on expiry
- Handles cross-tab synchronization

#### Identity Resolution
```javascript
Identity Resolution Flow:
1. FingerprintJS visitor ID (primary)
2. IP + Browser fingerprint correlation
3. IP-only fallback
4. Local storage backup
```

### 2. Server Architecture

#### API Endpoints
```
POST /api/init
├── Validate visitor data
├── Create/resume session
└── Return session context

POST /api/track
├── Validate session
├── Process event data
└── Store with identity context

POST /api/identify
├── Update identity mappings
├── Merge related identities
└── Update historical events
```

#### Identity Stitching Logic
```sql
-- Primary matching (Fingerprint)
SELECT * FROM identity_mappings 
WHERE visitor_id = $1

-- Secondary matching (IP + Browser)
SELECT * FROM identity_mappings 
WHERE ip_address = $1 
  AND browser_details->>'fingerprint' = $2

-- Fallback matching (IP only)
SELECT * FROM identity_mappings 
WHERE ip_address = $1
```

### 3. Database Schema

#### Tables Structure
```sql
identity_mappings
├── visitor_id (unique)
├── ip_address
├── browser_details (JSONB)
├── confidence_score
├── first_seen_at
├── last_seen_at
└── identity (JSONB)

session_mappings
├── session_id (primary key)
├── visitor_id
├── ip_address
└── browser_details (JSONB)

events
├── id (serial)
├── session_id
├── visitor_id
├── event_name
├── properties (JSONB)
└── identity (JSONB)
```

## Core Functionalities

### 1. Page Visit Tracking

```javascript
Flow:
1. Capture page metadata
2. Get current session
3. Enrich with visitor data
4. Send to server
5. Handle response/errors
```

### 2. Custom Event Tracking

```javascript
Flow:
1. Validate event data
2. Add session context
3. Add identity context
4. Send to server
5. Queue if offline
```

### 3. User Identification

```javascript
Flow:
1. Validate user data
2. Update local identity
3. Send to server
4. Update related sessions
5. Merge historical events
```

## Technical Implementation Details

### 1. Fingerprint Integration

```javascript
// Initialize FingerprintJS
const fpPromise = import('https://fpjscdn.net/v3/your-api-key')
    .then(FingerprintJS => FingerprintJS.load({
        apiKey: 'your-api-key'
    }));

// Get visitor data
const fp = await fpPromise;
const result = await fp.get();
const visitorId = result.visitorId;
```

### 2. Session Management

```javascript
class SessionManager {
    async initialize() {
        this.sessionId = this.getStoredSession();
        if (!this.sessionId || this.isExpired()) {
            await this.createNewSession();
        }
    }

    async createNewSession() {
        const response = await this.api.init({
            visitorId: this.visitorId,
            requestId: this.requestId
        });
        this.sessionId = response.sessionId;
        this.storeSession();
    }
}
```

### 3. Event Processing

```javascript
class EventProcessor {
    async track(eventName, properties) {
        const event = {
            sessionId: this.sessionId,
            eventName,
            properties,
            timestamp: new Date().toISOString(),
            visitorId: this.visitorId,
            identity: this.currentIdentity
        };

        await this.processEvent(event);
    }

    async processEvent(event) {
        // Add device/browser context
        event.context = await this.getEventContext();

        // Handle offline mode
        if (!navigator.onLine) {
            return this.queueEvent(event);
        }

        // Send to server
        return this.api.track(event);
    }
}
```

### 4. Identity Stitching

```javascript
class IdentityStitcher {
    async findRelatedIdentities(visitorId, ip) {
        // Find exact fingerprint matches
        const fingerprintMatches = await this.findByVisitorId(visitorId);
        
        // Find IP + browser matches
        const ipMatches = await this.findByIpAndBrowser(ip, browserDetails);
        
        // Merge identities with confidence scoring
        return this.mergeIdentities([
            ...fingerprintMatches,
            ...ipMatches
        ]);
    }

    calculateConfidence(match) {
        if (match.type === 'fingerprint') return 1.0;
        if (match.type === 'ip_browser') return 0.8;
        if (match.type === 'ip') return 0.5;
        return 0.3;
    }
}
```

## Error Handling & Recovery

### 1. Network Failures
```javascript
- Queue failed events in IndexedDB
- Implement exponential backoff
- Retry on connection restore
```

### 2. Data Consistency
```javascript
- Transaction-based updates
- Atomic identity merges
- Session validation
```

### 3. Edge Cases
```javascript
- Handle incognito mode
- Cross-domain tracking
- Bot detection
```

## Performance Considerations

### 1. Client-Side
```javascript
- Batch event processing
- Minimize storage operations
- Lazy loading of features
```

### 2. Server-Side
```javascript
- Index optimization
- Query caching
- Connection pooling
```

## Security Measures

### 1. Data Protection
```javascript
- HTTPS only
- API key rotation
- Data encryption
```

### 2. Request Validation
```javascript
- Session validation
- Origin checking
- Rate limiting
