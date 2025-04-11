# Shopify App: Fraud Prevention Suite

## Project Overview
Comprehensive fraud prevention solution for Shopify stores including email validation, geo-location rules, IP filtering, email domain filtering, and first-order rules.

## Current Setup
- Framework: Remix
- Database: SQLite
- App Type: Shopify App

## Database Schema
```sql
// Blacklisted Emails
model BlacklistedEmail {
  id        String   @id @default(uuid())
  user_id   String
  email     String
  reason    String?
  matches   Int      @default(0)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

// Geo Location Rules
model GeoRule {
  id          String   @id @default(uuid())
  user_id     String
  countryCode String
  action      String   // ALLOW, BLOCK
  status      String   // ACTIVE, INACTIVE
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}

// IP Filtering
model BlockedIp {
  id        String   @id @default(uuid())
  user_id   String
  ip        String
  reason    String?
  matches   Int      @default(0)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

// Email Domain Filtering
model EmailDomainFilter {
  id        String   @id @default(uuid())
  user_id   String
  domain    String
  filterType String  // BLACKLIST, WHITELIST
  reason    String?
  matches   Int      @default(0)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

// First Order Rules
model FirstOrderRule {
  id        String   @id @default(uuid())
  user_id   String
  ruleType  String   // MAX_ORDER_VALUE, REQUIRED_FIELDS, etc.
  value     String
  status    String   // ACTIVE, INACTIVE
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

## Features Required

### 1. Email Validation
- Block specific email addresses
- Real-time validation during checkout
- Store blacklisted emails in database

### 2. Geo-Location Rules
- Allow/block orders by country
- Multiple rule support
- Rule priority system

### 3. IP Filtering
- Block suspicious IP addresses
- Track IP match counts
- Support for IP ranges

### 4. Email Domain Filtering
- Block/allow specific email domains
- Support for wildcard domains
- Track domain match counts

### 5. First Order Rules
- Maximum order value limits
- Required field validation
- Custom rule conditions

## Implementation Approach

### 1. App Proxy Endpoints
```typescript
// Fetch all validation rules
export async function loader({ request }) {
  const { session } = await authenticate.admin(request);
  
  const [
    blacklistedEmails,
    geoRules,
    blockedIps,
    domainFilters,
    firstOrderRules
  ] = await Promise.all([
    db.$queryRaw`SELECT * FROM blacklisted_emails WHERE user_id = ${session.id}`,
    db.$queryRaw`SELECT * FROM geo_rules WHERE user_id = ${session.id}`,
    db.$queryRaw`SELECT * FROM blocked_ips WHERE user_id = ${session.id}`,
    db.$queryRaw`SELECT * FROM email_domain_filters WHERE user_id = ${session.id}`,
    db.$queryRaw`SELECT * FROM first_order_rules WHERE user_id = ${session.id}`
  ]);

  return json({
    blacklistedEmails,
    geoRules,
    blockedIps,
    domainFilters,
    firstOrderRules
  });
}
```

### 2. Cart Attributes Update
```typescript
// Update cart with all validation rules
async function updateCartAttributes(rules) {
  await fetch('/cart/update.js', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      attributes: {
        'fraudguard_rules': JSON.stringify(rules)
      }
    })
  });
}
```

### 3. Checkout Validation Function
```typescript
export function run(input) {
  const rules = JSON.parse(input.cart.attributes.fraudguard_rules || '{}');
  const errors = [];

  // Email validation
  if (rules.blacklistedEmails?.includes(input.cart.buyerIdentity?.email)) {
    errors.push({ message: "Email not allowed", target: "cart" });
  }

  // Geo validation
  const countryCode = input.cart.buyerIdentity?.countryCode;
  if (countryCode && rules.geoRules?.some(rule => 
    rule.countryCode === countryCode && rule.action === 'BLOCK')) {
    errors.push({ message: "Orders not allowed from your country", target: "cart" });
  }

  // IP validation
  const ip = input.cart.buyerIdentity?.ip;
  if (ip && rules.blockedIps?.includes(ip)) {
    errors.push({ message: "IP address not allowed", target: "cart" });
  }

  // Domain validation
  const emailDomain = input.cart.buyerIdentity?.email?.split('@')[1];
  if (emailDomain && rules.domainFilters?.some(filter => 
    filter.domain === emailDomain && filter.filterType === 'BLACKLIST')) {
    errors.push({ message: "Email domain not allowed", target: "cart" });
  }

  // First order rules
  if (input.cart.buyerIdentity?.isFirstOrder) {
    const orderValue = input.cart.cost.totalAmount.amount;
    if (rules.firstOrderRules?.maxOrderValue && 
        orderValue > rules.firstOrderRules.maxOrderValue) {
      errors.push({ 
        message: "First order value exceeds limit", 
        target: "cart" 
      });
    }
  }

  return { errors };
}
```

## Technical Requirements
1. Secure data storage and access
2. Real-time validation
3. Comprehensive error handling
4. User-friendly error messages
5. Performance optimization
6. Scalable architecture

## Deliverables
1. Complete fraud prevention suite
2. Admin dashboard for rule management
3. Real-time validation system
4. Detailed documentation
5. Testing suite

## Timeline
- Expected completion: [Your timeline]
- Budget: [Your budget]

## Questions for Developer
1. Experience with Shopify Functions and Remix?
2. Experience with fraud prevention systems?
3. Approach to handling multiple validation rules?
4. Strategy for performance optimization?
5. Experience with SQLite and database optimization?

## Additional Notes
- Security is top priority
- System should be scalable
- Performance should be optimized
- Code should be well-documented
- Regular updates and maintenance plan
