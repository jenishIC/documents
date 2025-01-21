# Event Tracking SDK Implementation Guide

## Basic Implementation

Add the following script to your HTML page:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Your Website</title>
    <!-- Add Event Tracking SDK -->
    <script src="event-tracker.js"></script>
    <script>
        // SDK will automatically initialize when DOM is loaded
        document.addEventListener('DOMContentLoaded', () => {
            // Track page visit automatically
            EventTracker.page_visit();
        });
    </script>
</head>
<body>
    <!-- Your website content -->
</body>
</html>
```

## Tracking Custom Events

```html
<button onclick="EventTracker.track('button_click', { buttonId: 'submit-btn' })">
    Submit
</button>

<script>
    // Track form submissions
    document.getElementById('myForm').addEventListener('submit', async (e) => {
        await EventTracker.track('form_submit', {
            formId: 'myForm',
            formType: 'contact'
        });
    });

    // Track custom interactions
    function trackPurchase(productId, amount) {
        EventTracker.track('purchase', {
            productId,
            amount,
            currency: 'USD'
        });
    }
</script>
```

## User Identification

```html
<script>
    // Identify user after login
    async function onUserLogin(userData) {
        await EventTracker.identify({
            userId: userData.id,
            email: userData.email,
            name: userData.name,
            plan: userData.subscriptionPlan
        });
    }

    // Example login form
    document.getElementById('loginForm').addEventListener('submit', async (e) => {
        e.preventDefault();
        const userData = await loginUser(); // Your login logic
        await onUserLogin(userData);
    });
</script>
```

## Advanced Usage

### Custom Page Properties

```html
<script>
    // Track page visit with custom properties
    EventTracker.page_visit({
        category: 'Product',
        searchQuery: urlParams.get('q'),
        filters: {
            category: 'electronics',
            priceRange: '100-200'
        }
    });
</script>
```

### E-commerce Tracking

```html
<script>
    // Track product view
    function trackProductView(product) {
        EventTracker.track('product_view', {
            productId: product.id,
            name: product.name,
            price: product.price,
            category: product.category
        });
    }

    // Track add to cart
    function trackAddToCart(product, quantity) {
        EventTracker.track('add_to_cart', {
            productId: product.id,
            name: product.name,
            price: product.price,
            quantity,
            totalValue: product.price * quantity
        });
    }

    // Track purchase
    function trackPurchase(order) {
        EventTracker.track('purchase_complete', {
            orderId: order.id,
            total: order.total,
            items: order.items.map(item => ({
                productId: item.id,
                quantity: item.quantity,
                price: item.price
            })),
            paymentMethod: order.paymentMethod
        });
    }
</script>
```

### SPA Implementation

```html
<script>
    // Track route changes in Single Page Applications
    function onRouteChange(newPath) {
        EventTracker.page_visit({
            path: newPath,
            previousPath: currentPath,
            timestamp: new Date().toISOString()
        });
    }

    // React Router example
    import { useEffect } from 'react';
    import { useLocation } from 'react-router-dom';

    function RouteTracker() {
        const location = useLocation();

        useEffect(() => {
            EventTracker.page_visit({
                path: location.pathname,
                search: location.search
            });
        }, [location]);

        return null;
    }
</script>
```

### Error Tracking

```html
<script>
    // Track JavaScript errors
    window.addEventListener('error', (event) => {
        EventTracker.track('js_error', {
            message: event.message,
            filename: event.filename,
            lineNo: event.lineno,
            colNo: event.colno,
            stack: event.error?.stack
        });
    });

    // Track API errors
    async function apiCall() {
        try {
            const response = await fetch('/api/data');
            if (!response.ok) {
                throw new Error('API Error');
            }
        } catch (error) {
            EventTracker.track('api_error', {
                endpoint: '/api/data',
                status: error.status,
                message: error.message
            });
        }
    }
</script>
```

### Performance Tracking

```html
<script>
    // Track page load performance
    window.addEventListener('load', () => {
        const performance = window.performance;
        const timing = performance.timing;

        EventTracker.track('page_performance', {
            loadTime: timing.loadEventEnd - timing.navigationStart,
            domReady: timing.domContentLoadedEventEnd - timing.navigationStart,
            firstPaint: performance.getEntriesByType('paint')[0]?.startTime,
            firstContentfulPaint: performance.getEntriesByType('paint')[1]?.startTime
        });
    });
</script>
```

## Best Practices

1. **Event Naming**
   - Use snake_case for event names (e.g., 'button_click', 'form_submit')
   - Be consistent with naming conventions
   - Use descriptive but concise names

2. **Property Structure**
   - Keep properties flat when possible
   - Use consistent property names
   - Include relevant context

3. **User Identification**
   - Call identify() as soon as user data is available
   - Include all relevant user properties
   - Update user data when it changes

4. **Error Handling**
   - Implement proper error tracking
   - Include relevant error context
   - Don't expose sensitive information

5. **Performance**
   - Initialize SDK early
   - Use async/await for tracking calls
   - Batch events when possible
