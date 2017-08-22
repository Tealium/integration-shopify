Tealium Shopify Integration
===========================

Summary
-------
With this integration, you can customize your Shopify theme to use Tealium. This code provides a Universal Data Object (UDO) and loads the necessary Tealium assets on each page of your site. This integration takes the form of a theme customization.

The code snippet for the order status page will also appear on the thank you page of the checkout. It is therefore the code that provides the UDO for conversion tracking. Because the order status page can be revisited after the order is placed, care must be taken not to track conversions twice. This logic can be configured in Tealium iQ.

Only Shopify Plus customers have access to the checkout.liquid file, which is needed to load utag.js throughout the checkout funnel. This integration does not currently support the checkout funnel, however you can do so by implementing your own checkout.liquid following the patterns used on other pages.

---------------------------

Setup Instructions
------------------

#### 1. Settings

Copy the following JSON object into the JSON array contained in the [Config/settings_schema.json](https://help.shopify.com/themes/development/theme-editor/settings-schema) file of your Shopify theme. This will enable the Tealium-specific settings for your theme.

  ```json
  {
    "name": "Tealium",
    "settings": [
      {
        "type": "text",
        "id": "tealium_account",
        "label": "Account",
        "info": "Name of your acount"
      },
      {
        "type": "text",
        "id": "tealium_profile",
        "label": "Profile",
        "default": "main",
        "info": "Which profile to load your tags from",
        "placeholder": "main"
      },
      {
        "type": "text",
        "id": "tealium_environment",
        "label": "Environment",
        "default": "prod",
        "info": "Which environment to load your tags from",
        "placeholder": "prod"
      },
      {
        "type": "checkbox",
        "id": "tealium_enabled",
        "label": "Enable",
        "default": false,
        "info": "Check to enable Tealium"
      }
    ]
  }
  ```
  Now you can edit your Tealium-specific settings in the **General Settings** tab when customizing your theme.

#### 2. Snippets

Drop the files from the **Snippets** folder of this GitHub repository into the **Snippets** folder of your Shopify theme. These files contain the actual UDO implementations for the various page types.

See [https://help.shopify.com/themes/development/templates#snippets](https://help.shopify.com/themes/development/templates#snippets) for information on theme snippets.

#### 3. Templates

On each page, you will need to include the correct snippet corresponding to the data layer of that page. To do that, go to the page's [template](https://help.shopify.com/themes/development/templates), or possibly the section for certain pages such as the index ("home") page, and insert the relevant snippet. For example, the **product.liquid** template file in the default **Debut** theme that Shopify provides would look like the following, where the **product_udo** snippet has been added at the top of the file.

  ```

  {% include 'product_udo' %}

  {% comment %}
    The contents of the product.liquid template can be found in /sections/product-template.liquid
  {% endcomment %}

  {% section 'product-template' %}

  <script>
    // Override default values of shop.strings for each template.
    // Alternate product templates can change values of
    // add to cart button, sold out, and unavailable states here.
    theme.productStrings = {
      addToCart: {{ 'products.product.add_to_cart' | t | json }},
      soldOut: {{ 'products.product.sold_out' | t | json }},
      unavailable: {{ 'products.product.unavailable' | t | json }}
    }
  </script>
  ```

#### 4. Order Status

The order status page works differently than the rest of the pages. Unlike other pages, the order status page doesn't have access to the admin Settings, so you must reconfigure your Tealium account information. To do that, define the settings in the **Additional scripts** box of the **Order processing** section. According to the [Shopify documentation](https://help.shopify.com/themes/customization/order-status), you can access it in the following way:


  > 1. From your Shopify admin, click Settings, then click Checkout.
  > 2. Find the Order processing section. Near the bottom of this section you will find Additional content & scripts.

  And then you'll need to add the following code. Before you begin, make sure to set `tealium_enabled` to `true`, and to set the `tealium_account`, `tealium_profile`, and `tealium_environment` variables to match what you configure in the settings.

  ```
  {% assign tealium_enabled = false %}
  {% assign tealium_account = "" %}
  {% assign tealium_profile = "" %}
  {% assign tealium_environment = "" %}

  {% if tealium_enabled and tealium_account != "" and tealium_profile != "" and tealium_environment != "" %}
  <script type="text/javascript">
    var utag_data = {
      "site_section": "order",
      "product_id": [
      {% for line_item in checkout.line_items %}
        "{{ line_item.product_id }}"{% unless forloop.last %},{% endunless %}
      {% endfor %}
      ],
      "product_quantity": [
      {% for line_item in checkout.line_items %}
        "{{ line_item.quantity }}"{% unless forloop.last %},{% endunless %}
      {% endfor %}
      ],
      "product_price": [
      {% for line_item in checkout.line_items %}
        "{{ line_item.price | money_without_currency }}"{% unless forloop.last %},{% endunless %}
      {% endfor %}
      ],
      "product_name": [
      {% for line_item in checkout.line_items %}
        "{{ line_item.title }}"{% unless forloop.last %},{% endunless %}
      {% endfor %}
      ],
      "product_sku": [
      {% for line_item in checkout.line_items %}
        "{{ line_item.sku }}"{% unless forloop.last %},{% endunless %}
      {% endfor %}
      ],

      "page_name": "{{ page_title }}",
      "language_code": "{{ shop.locale }}",

      {% if customer %}
        "customer_logged_in": "true",
        "customer_id": "{{ customer.id }}",
        "customer_email": "{{ customer.email }}",
        "customer_first_name": "{{ customer.first_name }}",
        "customer_last_name": "{{ customer.last_name }}",

        {% if customer.default_address %}
          "country_code": "{{ customer.default_address.country_code }}",
        {% endif %}

      {% else %}
        "customer_loggedin": "false",
      {% endif %}

      {% if cart %}
        "cart_total_items": "{{ cart.item_count }}",
        "cart_total_value": "{{ cart.total_price | money_without_currency }}",
      {% endif %}

      "page_type": "order"
    }
  </script>

  <!-- Loading script asynchronously -->
  <script type="text/javascript">
      (function(a,b,c,d){
      a='//tags.tiqcdn.com/utag/{{ settings.tealium_account }}/{{ settings.tealium_profile }}/{{ settings.tealium_environment }}/utag.js';
      b=document;c='script';d=b.createElement(c);d.src=a;d.type='text/java'+c;d.async=true;
      a=b.getElementsByTagName(c)[0];a.parentNode.insertBefore(d,a);
      })();
  </script>
  {% else %}
  <!-- Please configure your Tealium account information for the order confirmation page. -->
  {% endif %}
  ```

----------------------------

## Change Log

- 1.0.2
    - Updated README to more clearly reflect the capabilities of the integration
    - Updated order_status snippet to include more data and better variable names
    - Renamed checkout.liquid to order_status.liquid to better reflect its functionality
- 1.0.1
    - Fixed misspelled variable name in the "global_udo_vars" snippet. "customer_loggedin" (wrong) => "customer_logged_in" (right)
- 1.0.0 Initial Release
    - All snippets and configuration required to customize a Shopify theme to integrate Tealium
    - Instructions for customizing a theme

----------------------------

## License

Use of this software is subject to the terms and conditions of the license agreement contained in the file titled "LICENSE.txt".  Please read the license before downloading or using any of the files contained in this repository. By downloading or using any of these files, you are agreeing to be bound by and comply with the license agreement.


---
Copyright (C) 2012-2017, Tealium Inc.
