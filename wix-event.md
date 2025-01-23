# Event Tracking Integration Guide for Wix Sites

This guide will help you integrate event tracking into your Wix website. Follow these steps to start tracking user interactions and behaviors on your Wix site.

## Setup Instructions

### Step 1: Add the Tracking Script to Your Wix Site

1. In your Wix dashboard, go to:
   - Settings → Custom Code
   - Click "Add Custom Code"
   - Select "Add Code to Head"

2. Add the following code:

```html
<!-- Event Tracking SDK -->
<script src="https://your-tracking-server.com/event-tracker.js"></script>
```

## Basic Event Tracking

### Automatic Page Views
The SDK automatically tracks page visits. For Wix dynamic pages, add this code in Settings → Custom Code:

```html
<script>
    // Track Wix page navigation
    window.addEventListener('load', () => {
        // Initial page view
        if (window.EventTracker) {
            EventTracker.page_visit({
                pageId: window.location.pathname,
                pageName: document.title,
                pageType: 'wix'
            });
        }

        // Track dynamic page changes
        const observer = new MutationObserver(() => {
            if (window.EventTracker) {
                EventTracker.page_visit({
                    pageId: window.location.pathname,
                    pageName: document.title,
                    pageType: 'wix'
                });
            }
        });

        // Observe Wix page container for changes
        const pageContainer = document.getElementById('SITE_CONTAINER');
        if (pageContainer) {
            observer.observe(pageContainer, { 
                subtree: true, 
                childList: true 
            });
        }
    });
</script>
```

### Track Button Clicks
Add tracking to Wix buttons using the Wix Code/Velo platform:

```javascript
// In your page code (Wix Editor → Add → Element Events)
export function button1_click(event) {
    EventTracker.track('button_click', {
        buttonId: 'button1',
        buttonText: event.target.innerText,
        pageSection: 'header',
        clickType: 'primary_cta'
    });
}
```

### Track Form Submissions
For Wix forms:

```javascript
// In your page code
export function contactForm_submit(event) {
    EventTracker.track('form_submit', {
        formId: 'contactForm',
        formType: 'contact',
        formLocation: 'contact_page',
        // Don't include sensitive form data
        hasEmail: Boolean(event.target.email),
        hasPhone: Boolean(event.target.phone)
    });
}
```

## User Identification

### After Wix Member Login
```javascript
// In your Member Area page code
export function onMemberLogin(member) {
    EventTracker.identify({
        userId: member._id,
        email: member.loginEmail,
        memberSince: member.memberSince,
        // Add any public member properties
        role: member.role,
        plan: member.plan
    });
}
```

## E-commerce Tracking (Wix Stores)

### Product Views
```javascript
// In your Product Page code
export function onProductPageLoad(product) {
    EventTracker.track('product_view', {
        productId: product._id,
        name: product.name,
        price: product.price,
        currency: product.currency,
        category: product.productType,
        collections: product.collections
    });
}
```

### Add to Cart
```javascript
// In your Product Page code
export function addToCart_click(product, quantity) {
    EventTracker.track('add_to_cart', {
        productId: product._id,
        name: product.name,
        price: product.price,
        quantity: quantity,
        currency: product.currency,
        totalValue: product.price * quantity
    });
}
```

### Purchase Completion
```javascript
// In your Thank You Page code
export function onPurchaseComplete(order) {
    EventTracker.track('purchase_complete', {
        orderId: order._id,
        total: order.total,
        currency: order.currency,
        items: order.lineItems.map(item => ({
            productId: item.productId,
            name: item.name,
            price: item.price,
            quantity: item.quantity
        })),
        paymentMethod: order.paymentMethod
    });
}
```

## Best Practices for Wix Sites

1. **Script Loading**
   - Add the tracking script in the HEAD section to ensure early loading
   - Initialize after the page load event
   - Verify initialization in the browser console

2. **Page Tracking**
   - Use the MutationObserver for reliable page change detection
   - Include Wix-specific page properties
   - Track lightbox/popup views separately

3. **Form Tracking**
   - Never track sensitive form data
   - Track form interactions (focus, validation)
   - Include form context (page, section)

4. **User Privacy**
   - Respect user privacy settings
   - Don't track personally identifiable information without consent
   - Follow GDPR/CCPA guidelines

## Troubleshooting

### Common Issues

1. **Script Not Loading**
   - Verify the script URL is correct and accessible
   - Check if the script is blocked by content security policy
   - Ensure the script is added to the HEAD section

2. **Events Not Tracking**
   ```javascript
   // Add this code temporarily to debug
   window.addEventListener('load', () => {
       if (window.EventTracker) {
           console.log('Tracker initialized');
           // Test event
           EventTracker.track('test_event', {
               timestamp: new Date().toISOString()
           }).then(console.log).catch(console.error);
       } else {
           console.error('Tracker not initialized');
       }
   });
   ```

3. **Page Changes Not Detected**
   - Verify the MutationObserver is properly set up
   - Check if the SITE_CONTAINER element exists
   - Try observing different DOM elements

### Support

For technical support or questions:
1. Check the browser console for errors
2. Verify your server URL and configuration
3. Contact our support team with:
   - Your Wix site URL
   - Browser console logs
   - Steps to reproduce any issues
