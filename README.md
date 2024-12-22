
# Setting Up Enhanced E-commerce in Google Analytics for Shopify

Shopify makes setting up Google Analytics very simple. Even tasks that are generally complicated, such as configuring Enhanced E-commerce, can be done in no time. This is highly recommended for anyone using Shopify.

## How to Set Up Google Analytics
For detailed instructions on setting up Google Analytics and Universal Analytics (UA) Enhanced E-commerce, refer to Shopify’s [Set up Google Analytics](https://help.shopify.com/en/manual/reports-and-analytics/google-analytics/google-analytics-setup) guide.

---

# Setting Up E-commerce for GA4

While Shopify automatically collects data for UA Enhanced E-commerce, as of December 2020, GA4’s Enhanced E-commerce is not yet supported. You’ll need to implement it yourself. The following instructions use the “Debut” theme as an example; adapt these instructions for your own theme if necessary.

## Tracking Product/List Views and Impressions
### Modifying `collection.liquid`
To track impressions of product lists, you need to insert product data into the `dataLayer`. Modify the `collection.liquid` file by adding the following script between `{% endcomment %}` and `{% section 'collection-template' %}`:

```javascript
var items = [];
var item_num = 1;
{% for product in collection.products %}
  {% assign current_variant = product.selected_or_first_available_variant %}
  var item = {
    'item_name': "{{ product.title }}",
    'item_id': "{{ current_variant.sku }}",
    'price': "{{ current_variant.price | divided_by: 100.00 }}",
    'item_brand': "{{ product.vendor }}",
    'item_category': "{{ product.collections[0].title }}",
    'item_list_name': "{{ collection.title }}",
    'item_list_id': "{{ collection.id }}",
    'index': item_num,
    'quantity': '1'
  };
  items.push(item);
  item_num += 1;
{% endfor %}

window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
  'event': 'view_item_list',
  'ecommerce': {
    'items': items
  }
});
```

Note: If your Shopify store doesn’t use SKU codes, replace `current_variant.sku` with `current_variant.id`.

### Setting Up GTM
Next, configure Google Tag Manager (GTM) to retrieve product information from the `dataLayer`. Create a variable:

- **Variable Name:** DLV – ecommerce.items
- **Variable Type:** Data Layer Variable
- **Data Layer Variable Name:** ecommerce.items
- **Version:** 2

Then, set up a custom event trigger:

- **Trigger Name:** Event – view_item_list
- **Trigger Type:** Custom Event
- **Event Name:** view_item_list

Finally, create an event tag:

- **Tag Name:** GA4 ecommerce – view_item_list
- **Measurement ID:** Your GA4 Measurement ID
- **Event Name:** view_item_list
- **Event Parameters:** 
  - **Name:** items, **Value:** `{{ DLV – ecommerce.items }}`
- **Trigger:** Event – view_item_list

---

## Tracking Product Detail Views
### Modifying `product.liquid`
To track product detail views, add the following script between `{% endcomment %}` and `{% section 'product-template' %}` in `product.liquid`:

```javascript
<script>
var item = {
  'item_name': "{{ product.title }}",
  'item_id': "{{ product.selected_or_first_available_variant.sku }}",
  'price': "{{ product.selected_or_first_available_variant.price | divided_by: 100.00 }}",
  'item_brand': "{{ product.vendor }}",
  'item_category': "{{ product.collections[0].title }}"
};

window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
  'event': 'view_item',
  'ecommerce': {
    'items': [item]
  }
});
</script>
```

### Configuring GTM
In GTM, reuse the variable `DLV – ecommerce.items`. Set up a custom event:

- **Trigger Name:** Event – view_item
- **Trigger Type:** Custom Event
- **Event Name:** view_item

Create an event tag:

- **Tag Name:** GA4 ecommerce – view_item
- **Measurement ID:** Your GA4 Measurement ID
- **Event Name:** view_item
- **Event Parameters:** 
  - **Name:** items, **Value:** `{{ DLV – ecommerce.items }}`
- **Trigger:** Event – view_item

---

## Measuring Add-to-Cart Events
### Modifying `product-template.liquid`
To capture SKU data when adding to cart, modify the `product-template.liquid` file by adding `data-sku="{{ variant.sku }}"` to the `<option>` element:

```html
<select name="id" id="ProductSelect-{{ section.id }}" class="product-form__variants no-js">
  {% for variant in product.variants %}
    <option value="{{ variant.id }}" data-sku="{{ variant.sku }}">{{ variant.title }}</option>
  {% endfor %}
</select>
```

### Configuring GTM
Create a custom HTML tag to send `add_to_cart` events to the `dataLayer`:

```javascript
<script>
(function addToCart(selectedSku) {
  window.dataLayer = window.dataLayer || [];
  window.dataLayer.push({
    'event': 'add_to_cart',
    'ecommerce': {
      'items': getItem(selectedSku)
    }
  });

  function getItem(selectedSku) {
    // Your logic to retrieve the item
  }
})();
</script>
```

Set up click triggers for “Add to Cart” buttons and configure a custom event for `add_to_cart` in GTM. Create the GA4 tag with appropriate event parameters.

---

## Measuring Purchases
### Adding Checkout Scripts in Shopify
In the Shopify Admin, navigate to **Settings → Checkout → Additional Scripts** and add this script:

```javascript
<script>
var ga4_ecommerce_items = [];
{% for item in checkout.line_items %}
  var item = {
    'item_name': "{{ item.title }}",
    'item_id': "{{ item.sku }}",
    'price': "{{ item.final_price | divided_by: 100.00 }}",
    'quantity': {{ item.quantity }}
  };
  ga4_ecommerce_items.push(item);
{% endfor %}

window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
  'event': 'purchase',
  'ecommerce': {
    'transaction_id': '{{ checkout.order_number }}',
    'value': '{{ checkout.total_price | money_without_currency }}',
    'currency': '{{ shop.currency }}',
    'items': ga4_ecommerce_items
  }
});
</script>
```

### Configuring GTM
Set up a custom event named `purchase` in GTM and create a GA4 tag to track purchase events with the `items` parameter.

---

## Bonus: Google Ads Conversion Tag
To track Google Ads conversions on the thank-you page, use this script:

```javascript
<script>
gtag('event', 'conversion', {
  'send_to': 'AW-XXXXXXXXX/ABCDEFGHIJKLMNOPQR',
  'value': '{{ checkout.total_price | money_without_currency }}',
  'currency': '{{ shop.currency }}',
  'transaction_id': '{{ checkout.order_number }}'
});
</script>
```
