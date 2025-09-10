# Segment Analytics Implementation Guide
## Enterprise Web Application

---

## Page 1: Architecture & Core Implementation

### Overview

This document outlines a comprehensive multi-platform analytics tracking system with **Segment** as the primary customer data platform (CDP). The system provides unified data collection, customer behavior analysis, and conversion funnel tracking across the entire user journey for enterprise web applications.

### Architecture Components

**Multi-Platform Approach:**
- **Segment Analytics** - Primary CDP for user behavior, product interactions, and conversion funnel analysis
- **Tealium** - Tag management system handling Google Analytics 4 (GA4) and third-party vendor tracking
- **Datadog RUM** - Real User Monitoring for performance and error tracking
- **Custom Tracking Utilities** - Application-specific tracking logic

**Key Design Principle:** Separation of concerns where Segment handles customer data and user identification, while Tealium manages GA4 and conversion pixels.

### Core Implementation Structure

**Primary Tracking Function:** `sendSegmentTracking()`
```typescript
function sendSegmentTracking({
  type: 'page' | 'track' | 'identify',
  eventLabel: string,
  params: object,
  context: object
})
```

**Essential Tracking Hooks:**
- **`useSegmentTracking`** - Automatic page view tracking with time-on-page calculation
- **`useProductActionTracking`** - Real-time product addition/removal monitoring
- **`useProductListTracking`** - Product list interaction tracking

### Event Classification System

**Three Core Event Types:**

1. **PAGE Events** - Page view tracking with navigation analytics
   - Automatic route change detection
   - Time spent calculation on previous pages
   - Checkout step progression tracking

2. **TRACK Events** - Custom user interaction tracking
   - Product Added/Removed
   - Cart Created/Viewed
   - Checkout Started/Completed
   - Promotion Clicked/Viewed
   - Experiment Enrolled/Treated

3. **IDENTIFY Events** - User identification and trait management
   - Logged-in user identification with customer ID
   - Anonymous user trait association
   - Email validation and PII protection

### Data Structure & Properties

**Common Event Properties (Auto-injected):**
```typescript
{
  category: string,           // Flow purpose (Q1, Q2, crossell, checkout)
  graph_type: string,         // Entity type (LLC, Corp, DBA, etc.)
  cart_id: string,           // Shopping cart identifier
  processing_order_id: string, // Order processing reference
  account_id: string,         // Customer account identifier
  package_id: string,         // Selected package/product ID
  topline_product_group: string, // Product categorization
  screen_name: string,        // Current page path (track events only)
  process_id: string,         // Business process identifier
  process_name: string        // Human-readable process name
}
```

**Context Enhancement:**
- **Answers Store** - User form responses (PII-filtered for non-EP flows)
- **Session Data** - Cart state, processing status, user authentication
- **Flow Overrides** - Custom tracking configuration per business flow

### Technical Integration

**Segment Snippet Integration:**
- Embedded directly in `index.html` for immediate availability
- Environment-specific proxy URLs for data routing
- OneTrust integration for privacy compliance
- Asynchronous loading with fallback handling

**State Management Integration:**
- **Flow Store** - Current graph type, tracking configuration, user answers
- **Session Store** - Cart data, processing orders, authentication state
- **Answer Store** - User form responses with PII protection

---

## Page 2: Advanced Features & Best Practices

### Advanced Tracking Features

**Intelligent Product Tracking:**
The system automatically detects product selection changes using a sophisticated diffing algorithm:
```typescript
// Automatically detects adds/removes without manual tracking calls
const productActions = determineAction({
  current: previousProducts,
  incoming: currentProducts
});
```

**Time-Based Analytics:**
- **Page Duration Tracking** - Automatic calculation of time spent on each page
- **Session Timeline** - Complete user journey mapping with timestamps
- **Funnel Progression** - Checkout step advancement with abandonment detection

**Smart User Identification:**
```typescript
// Handles both logged-in and anonymous users
const handleUserIdentification = (traits, context) => {
  const userId = extractUserIdFromCookies();
  const isLoggedIn = checkAuthenticationStatus();

  if (isLoggedIn) {
    analytics.identify(userId, traits, context);
  } else {
    analytics.identify(traits, context); // Anonymous identification
  }
};
```

### Privacy & Data Protection

**PII Protection Mechanisms:**
- **Automatic Email Validation** - Invalid emails are stripped from tracking data
- **EP Flow Exclusion** - Estate Planning flows exclude user answers from context
- **PII Filtering** - `getAnswersWithoutPII()` removes sensitive information
- **Context Sanitization** - User input marked as 'NOT_TRACKED' for sensitive flows

**Flow-Specific Overrides:**
```typescript
{
  user_input: 'NOT_TRACKED',  // Disable input tracking for sensitive flows
  process_id: 'custom_id',    // Override process identification
  process_name: 'Custom Flow' // Custom process naming
}
```

### Event Catalog & Standards

**E-commerce Event Standards:**
- **Cart Management** - Cart Created, Cart Viewed, Cart Updated
- **Product Interactions** - Product Added, Product Removed, Product Clicked, Product List Viewed
- **Checkout Funnel** - Checkout Started, Checkout Step Viewed, Checkout Completed
- **Order Management** - Order Completed, Order Updated, Payment Info Entered

**Engagement Tracking:**
- **Content Interaction** - Content Viewed, Surface Viewed, Alert Viewed
- **Promotional Events** - Promotion Clicked, Promotion Viewed
- **Experimentation** - Experiment Enrolled, Experiment Treated
- **Form Completion** - Questionnaire Completed, Interaction events

### Integration with Business Logic

**Flow Type Determination:**
The system automatically categorizes user flows:
- **Q1 Flows** - Initial business formation questionnaires
- **Q2 Flows** - Secondary confirmation and add-on selection
- **Checkout Flows** - Payment and order completion
- **Cross-sell Flows** - Additional product recommendations

**Process Identification:**
Each user journey is mapped to specific business processes with standardized naming conventions for consistent analysis across different flow types.

### Implementation Best Practices

**Developer Guidelines:**
1. **Hook Usage** - Include `useSegmentTracking()` in main application components for automatic page tracking
2. **Product Pages** - Add `useProductActionTracking()` to pages with product selection capabilities
3. **Manual Events** - Use `sendSegmentTracking()` for custom interaction tracking
4. **Type Safety** - Leverage TypeScript definitions for event structure validation

**Performance Considerations:**
- **Lazy Loading** - Tracking scripts load asynchronously to prevent blocking
- **Batch Processing** - Multiple events can be queued and sent efficiently
- **Error Handling** - Graceful degradation when analytics services are unavailable

**Data Quality Assurance:**
- **Validation Layer** - All event properties validated before transmission
- **Standardized Schemas** - Consistent data structure across all tracking events
- **Testing Integration** - Comprehensive test coverage for tracking implementation

### Monitoring & Analytics

**Real-time Capabilities:**
- **Event Stream Monitoring** - Live tracking event validation
- **Funnel Analysis** - Real-time conversion rate tracking
- **User Journey Mapping** - Complete customer experience analysis
- **A/B Testing Integration** - Experiment enrollment and treatment tracking

This implementation provides a robust, scalable, and privacy-compliant analytics foundation that supports comprehensive business intelligence while maintaining high data quality standards.
