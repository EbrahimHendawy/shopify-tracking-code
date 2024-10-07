# Shopify Multi-Platform Tracking Script

This script provides a unified solution for tracking customer events across multiple platforms (Facebook, TikTok, and Snapchat) in your Shopify store.

## Features

- Tracks key events: PageView, ViewContent, AddToCart, Search, AddPaymentInfo, InitiateCheckout, and Purchase
- Supports Facebook Pixel, TikTok Pixel, and Snapchat Pixel
- Utilizes Shopify's analytics events for accurate tracking
- Includes debug mode for easier troubleshooting

## Installation

1. Log in to your Shopify admin panel
2. Go to Settings > Customer events
3. In the "Custom JavaScript" section, paste the entire script provided below

## Configuration

Before using the script, you need to replace the placeholder pixel IDs with your actual pixel IDs:

```javascript
const FACEBOOK_PIXEL_ID = 'YOUR_FACEBOOK_PIXEL_ID';
const TIKTOK_PIXEL_ID = 'YOUR_TIKTOK_PIXEL_ID';
const SNAPCHAT_PIXEL_ID = 'YOUR_SNAPCHAT_PIXEL_ID';
```

## Debug Mode

The script includes a debug mode that logs events to the console. To enable or disable debug mode, set the `DEBUG` constant:

```javascript
const DEBUG = true; // Set to false to disable debug mode
```

## Customization

You can customize the event mapping or add new events by modifying the `fireEvent` function. Make sure to handle the new events in the appropriate analytics.subscribe calls.

## Troubleshooting

If you're not seeing events in your pixel dashboards:

1. Enable debug mode and check the browser console for any error messages
2. Verify that your pixel IDs are correct
3. Ensure that you've placed the script in the correct location in your Shopify settings
4. Check if there are any ad-blockers or privacy tools preventing the pixels from loading

## Support

For issues or questions about this script, please contact your development team or Shopify support.

Remember to test thoroughly in a development environment before deploying to your live store.

## Full Script

Here's the full script to be pasted into your Shopify Custom JavaScript section:

```javascript
// Pixel IDs
const FACEBOOK_PIXEL_ID = 'YOUR_FACEBOOK_PIXEL_ID';
const TIKTOK_PIXEL_ID = 'YOUR_TIKTOK_PIXEL_ID';
const SNAPCHAT_PIXEL_ID = 'YOUR_SNAPCHAT_PIXEL_ID';

// Debug mode
const DEBUG = true;

// Logging functions
const log = DEBUG ? (message) => console.log(`[Tracking Debug] ${message}`) : () => {};
const logError = DEBUG ? (message, error) => console.error(`[Tracking Error] ${message}`, error) : () => {};

// Initialize Facebook Pixel
!function(f,b,e,v,n,t,s){if(f.fbq)return;n=f.fbq=function(){n.callMethod?n.callMethod.apply(n,arguments):n.queue.push(arguments)};if(!f._fbq)f._fbq=n;n.push=n;n.loaded=!0;n.version='2.0';n.queue=[];t=b.createElement(e);t.async=!0;t.src=v;s=b.getElementsByTagName(e)[0];s.parentNode.insertBefore(t,s)}(window,document,'script','https://connect.facebook.net/en_US/fbevents.js');
fbq('init', FACEBOOK_PIXEL_ID);

// Initialize TikTok Pixel
!function(w,d,t){w.TiktokAnalyticsObject=t;var ttq=w[t]=w[t]||[];ttq.methods=["page","track","identify","instances","debug","on","off","once","ready","alias","group","enableCookie","disableCookie"],ttq.setAndDefer=function(t,e){t[e]=function(){t.push([e].concat(Array.prototype.slice.call(arguments,0)))}};for(var i=0;i<ttq.methods.length;i++)ttq.setAndDefer(ttq,ttq.methods[i]);ttq.instance=function(t){for(var e=ttq._i[t]||[],n=0;n<ttq.methods.length;n++)ttq.setAndDefer(e,ttq.methods[n]);return e},ttq.load=function(e,n){var i="https://analytics.tiktok.com/i18n/pixel/events.js";ttq._i=ttq._i||{},ttq._i[e]=[],ttq._i[e]._u=i,ttq._t=ttq._t||{},ttq._t[e]=+new Date,ttq._o=ttq._o||{},ttq._o[e]=n||{};var o=document.createElement("script");o.type="text/javascript",o.async=!0,o.src=i+"?sdkid="+e+"&lib="+t;var a=document.getElementsByTagName("script")[0];a.parentNode.insertBefore(o,a)};
ttq.load(TIKTOK_PIXEL_ID)}(window,document,'ttq');

// Initialize Snapchat Pixel
!function(e,t,n){if(e.snaptr)return;var a=e.snaptr=function(){a.handleRequest?a.handleRequest.apply(a,arguments):a.queue.push(arguments)};a.queue=[];var s='script';r=t.createElement(s);r.async=!0;r.src=n;var u=t.getElementsByTagName(s)[0];u.parentNode.insertBefore(r,u)}(window,document,'https://sc-static.net/scevent.min.js');
snaptr('init', SNAPCHAT_PIXEL_ID);

// Helper function to safely access nested properties
const safeGet = (obj, path) => path.split('.').reduce((acc, part) => acc && acc[part], obj);

// Function to fire events for all platforms
function fireEvent(eventName, eventData) {
  let snaptrData;
  switch(eventName) {
    case 'PageView':
      fbq('track', 'PageView');
      ttq.page();
      snaptr('track', 'PAGE_VIEW');
      break;
    case 'ViewContent':
    case 'AddToCart':
      fbq('track', eventName, eventData);
      ttq.track(eventName, eventData);
      snaptrData = {
        item_ids: [eventData.content_ids[0]],
        item_names: [eventData.content_name],
        currency: eventData.currency,
        price: eventData.value
      };
      snaptr('track', eventName === 'ViewContent' ? 'VIEW_CONTENT' : 'ADD_CART', snaptrData);
      break;
    case 'Search':
      fbq('track', 'Search', eventData);
      ttq.track('Search', { query: eventData.search_string });
      snaptr('track', 'SEARCH', eventData);
      break;
    case 'AddPaymentInfo':
      fbq('track', 'AddPaymentInfo');
      ttq.track('AddPaymentInfo');
      snaptr('track', 'ADD_BILLING');
      break;
    case 'InitiateCheckout':
      fbq('track', 'InitiateCheckout', eventData);
      ttq.track('InitiateCheckout', eventData);
      snaptr('track', 'START_CHECKOUT', {
        currency: eventData.currency,
        price: eventData.value,
        number_items: eventData.num_items
      });
      break;
    case 'Purchase':
      fbq('track', 'Purchase', eventData);
      ttq.track('PlaceAnOrder', eventData);
      snaptr('track', 'PURCHASE', {
        currency: eventData.currency,
        price: eventData.value,
        transaction_id: eventData.transaction_id,
        item_ids: eventData.content_ids,
        number_items: eventData.num_items
      });
      break;
  }
  log(`Fired ${eventName} event for all platforms`);
}

// Event handlers
analytics.subscribe("page_viewed", () => {
  log('Page viewed event received');
  fireEvent('PageView');
});

analytics.subscribe("product_viewed", (event) => {
  log('Product viewed event received');
  const product = safeGet(event, 'data.productVariant');
  if (product) {
    fireEvent('ViewContent', {
      content_type: 'product',
      content_ids: [product.id],
      content_name: product.title,
      currency: product.price.currencyCode,
      value: product.price.amount,
    });
  } else {
    logError('Product data not available for product_viewed event');
  }
});

analytics.subscribe("search_submitted", (event) => {
  log('Search submitted event received');
  const query = safeGet(event, 'searchResult.query');
  query ? fireEvent('Search', { search_string: query }) : logError('Search query not available for search_submitted event');
});

analytics.subscribe("product_added_to_cart", (event) => {
  log('Product added to cart event received');
  const product = safeGet(event, 'data.cartLine.merchandise.productVariant');
  if (product) {
    fireEvent('AddToCart', {
      content_type: 'product',
      content_ids: [product.id],
      content_name: product.title,
      currency: product.price.currencyCode,
      value: product.price.amount,
    });
  } else {
    logError('Product data not available for product_added_to_cart event');
  }
});

analytics.subscribe("payment_info_submitted", () => {
  log('Payment info submitted event received');
  fireEvent('AddPaymentInfo');
});

analytics.subscribe("checkout_started", (event) => {
  log('Checkout started event received');
  const cart = safeGet(event, 'data.checkout');
  if (cart) {
    fireEvent('InitiateCheckout', {
      content_type: 'product',
      contents: cart.lineItems.map(item => ({
        id: item.variant.id,
        quantity: item.quantity
      })),
      currency: cart.currencyCode,
      value: cart.totalPrice.amount,
      num_items: cart.lineItems.reduce((total, item) => total + item.quantity, 0)
    });
  } else {
    logError('Cart data not available for checkout_started event');
  }
});

analytics.subscribe("checkout_completed", (event) => {
  log('Checkout completed event received');
  const checkout = safeGet(event, 'data.checkout');
  if (checkout) {
    fireEvent('Purchase', {
      content_type: 'product',
      contents: checkout.lineItems.map(item => ({
        id: item.variant.id,
        quantity: item.quantity
      })),
      content_ids: checkout.lineItems.map(item => item.variant.id),
      currency: checkout.currencyCode,
      value: checkout.totalPrice.amount,
      num_items: checkout.lineItems.reduce((total, item) => total + item.quantity, 0),
      transaction_id: checkout.order.id
    });
  } else {
    logError('Checkout data not available for checkout_completed event');
  }
});

log('Multi-platform tracking script loaded and initialized');
```

Make sure to replace `YOUR_FACEBOOK_PIXEL_ID`, `YOUR_TIKTOK_PIXEL_ID`, and `YOUR_SNAPCHAT_PIXEL_ID` with your actual pixel IDs before using this script.
