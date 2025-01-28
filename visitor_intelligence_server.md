# Server-Side Implementation Details

## Database Schema

### 1. Identity Mappings Table
```sql
CREATE TABLE ${tenantId}.identity_mappings (
    id SERIAL PRIMARY KEY,
    visitor_id TEXT UNIQUE,
    ip_address TEXT,
    browser_details JSONB,
    confidence_score FLOAT,
    first_seen_at TIMESTAMP WITH TIME ZONE,
    last_seen_at TIMESTAMP WITH TIME ZONE,
    geolocation JSONB,
    asn JSONB,
    identification_method TEXT,
    identity JSONB
);
```

### 2. Session Mappings Table
```sql
CREATE TABLE ${tenantId}.session_mappings (
    id SERIAL PRIMARY KEY,
    session_id TEXT UNIQUE,
    visitor_id TEXT,
    ip_address TEXT,
    browser_details JSONB,
    confidence_score FLOAT,
    identification_method TEXT
);
```

### 3. Events Table
```sql
CREATE TABLE ${tenantId}.events (
    id SERIAL PRIMARY KEY,
    session_id TEXT,
    visitor_id TEXT,
    event_name TEXT,
    properties JSONB,
    identity JSONB,
    ip_address TEXT,
    browser_details JSONB,
    confidence_score FLOAT,
    identification_method TEXT,
    geolocation JSONB,
    asn JSONB,
    first_seen_at TIMESTAMP WITH TIME ZONE,
    last_seen_at TIMESTAMP WITH TIME ZONE,
    timestamp TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

## API Endpoints

### 1. Initialize Session (/eventTracking/init)
- **Method**: POST
- **Purpose**: Initialize visitor tracking session
- **Request Body**:
  ```json
  {
    "requestId": "string",
    "visitorId": "string",
    "tenantId": "string"
  }
  ```
- **Process Flow**:
  1. Validate required parameters
  2. Get verified visitor data from FingerprintJS Pro
  3. Verify visitor ID authenticity
  4. Generate unique session ID
  5. Check for existing identity
  6. Create/update identity mapping
  7. Store session mapping
- **Response**:
  ```json
  {
    "sessionId": "string",
    "identity": "object | null",
    "identityMatch": {
      "type": "string",
      "confidence": "number"
    },
    "visitorData": {
      "visitorId": "string",
      "browserDetails": "object",
      "device": "object",
      "os": "object",
      "geolocation": "object",
      "firstSeen": "datetime",
      "lastSeen": "datetime"
    }
  }
  ```

### 2. Track Event (/eventTracking/track)
- **Method**: POST
- **Purpose**: Record user events with context
- **Request Body**:
  ```json
  {
    "sessionId": "string",
    "eventName": "string",
    "properties": "object",
    "requestId": "string",
    "visitorId": "string",
    "tenantId": "string"
  }
  ```
- **Process Flow**:
  1. Validate session existence
  2. Verify visitor ID if provided
  3. Get existing identity information
  4. Retrieve latest geolocation data
  5. Store event with full context
- **Response**:
  ```json
  {
    "event": "object",
    "identityMatch": {
      "type": "string",
      "confidence": "number"
    }
  }
  ```

### 3. Identify User (/eventTracking/identify)
- **Method**: POST
- **Purpose**: Associate user identity with visitor
- **Request Body**:
  ```json
  {
    "sessionId": "string",
    "userData": "object",
    "requestId": "string",
    "visitorId": "string",
    "tenantId": "string"
  }
  ```
- **Process Flow**:
  1. Validate session
  2. Verify visitor ID
  3. Update primary identity mapping
  4. Find and update related identities
  5. Update historical events
  6. Track identify event
- **Response**:
  ```json
  {
    "success": "boolean",
    "identity": "object",
    "relatedIdentitiesUpdated": "number"
  }
  ```

## Identity Resolution Algorithm

### 1. Finding Existing Identity
```javascript
async function findExistingIdentity(tenantId, visitorId, ip, browserDetails) {
    // Step 1: Try exact fingerprint match (highest confidence)
    const fingerprintMatch = await findByVisitorId(visitorId);
    if (fingerprintMatch) {
        return {
            ...fingerprintMatch,
            matchType: 'fingerprint',
            confidence: Math.max(fingerprintMatch.confidence_score || 0.8, 0.8)
        };
    }

    // Step 2: Try IP + browser correlation
    const ipMatches = await findByIpAddress(ip);
    if (ipMatches.length > 0 && browserDetails) {
        const browserMatch = findMatchingBrowser(ipMatches, browserDetails);
        if (browserMatch) {
            return {
                ...browserMatch,
                matchType: 'ip_browser',
                confidence: Math.max(browserMatch.confidence_score || 0.6, 0.6)
            };
        }
    }

    // Step 3: Use IP-only match as last resort
    if (ipMatches.length > 0) {
        return {
            ...ipMatches[0],
            matchType: 'ip',
            confidence: Math.max(ipMatches[0].confidence_score || 0.4, 0.4)
        };
    }

    return null;
}
```

### 2. Identity Propagation Logic
```javascript
// During identify operation
async function propagateIdentity(tenantId, sessionData, userData) {
    // Step 1: Update primary identity
    await updateIdentityMapping(sessionData.visitor_id, userData);

    // Step 2: Find related identities
    const relatedIdentities = await findRelatedIdentities(
        sessionData.ip_address,
        sessionData.visitor_id,
        sessionData.browser_details
    );

    // Step 3: Update related identities
    if (relatedIdentities.length > 0) {
        await updateRelatedIdentities(relatedIdentities, userData);
        await updateHistoricalEvents(relatedIdentities, userData);
    }

    // Step 4: Track identify event
    await trackIdentifyEvent(sessionData, userData);
}
```

## Security Implementation

### 1. Visitor Verification
```javascript
async function verifyVisitor(requestId, visitorId) {
    const visitorData = await getVisitorData(requestId);
    return visitorData.visitorId === visitorId;
}
```

### 2. Session Validation
```javascript
async function validateSession(tenantId, sessionId) {
    const session = await getSessionMapping(tenantId, sessionId);
    if (!session) {
        throw new Error('Invalid session');
    }
    return session;
}
```

### 3. Tenant Isolation
- Each tenant has isolated database schema
- All queries are scoped to tenant
- Tenant ID validation on every request

## Data Enrichment Process

### 1. Event Context Enrichment
```javascript
async function enrichEventData(event, session, identity) {
    return {
        ...event,
        visitor_id: session.visitor_id,
        ip_address: session.ip_address,
        browser_details: session.browser_details,
        confidence_score: identity?.confidence || session.confidence_score,
        identification_method: session.identification_method,
        geolocation: await getGeolocationData(session.ip_address),
        asn: await getAsnData(session.ip_address),
        timestamp: new Date(),
        identity: identity?.data
    };
}
```

### 2. Identity Context Management
```javascript
async function updateIdentityContext(tenantId, visitorId, newData) {
    // Update identity mapping
    await updateIdentityMapping(tenantId, visitorId, {
        ...newData,
        last_seen_at: new Date()
    });

    // Update related sessions
    await updateSessionMappings(tenantId, visitorId, newData);

    // Update recent events
    await updateRecentEvents(tenantId, visitorId, newData);
}
```

## Transaction Management

### 1. Identity Update Transaction
```javascript
async function executeIdentityUpdate(client, updates) {
    try {
        await client.query('BEGIN');
        
        // Execute all updates in transaction
        for (const update of updates) {
            await client.query(update.query, update.params);
        }
        
        await client.query('COMMIT');
    } catch (error) {
        await client.query('ROLLBACK');
        throw error;
    }
}
```

### 2. Event Recording Transaction
```javascript
async function recordEventWithContext(client, eventData) {
    try {
        await client.query('BEGIN');
        
        // Store event
        await client.query(insertEventQuery, eventData);
        
        // Update last seen timestamps
        await client.query(updateLastSeenQuery, {
            visitorId: eventData.visitor_id
        });
        
        await client.query('COMMIT');
    } catch (error) {
        await client.query('ROLLBACK');
        throw error;
    }
}
