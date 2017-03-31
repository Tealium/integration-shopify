Tealium Shopify Integration
===========================

---------------------------

Summary
-------

For Tealium iQ users with Shopify, use this integration to add the necessary Tealium specific code to your site. It provides a UDO (Universal Data Object) and loads the necessary Tealium assets on each page of your site. The integration takes the form of a theme customization.

---------------------------

Setup Instructions
------------------

You're basically just going to customize your existing Shopify theme.

1. ###### Settings
Copy the following JSON object into the JSON array contained in the Config/settings_schema.json file of your Shopify them. This will enable Tealium specific settings for your theme.

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
  Now you can edit your Tealium specific settings in the "General Settigs" tab when customizing your theme.

2. ###### Snippets
Drop the files from the "Snippets" folder of this repo into the "Snippets" folder of your Shopify theme. These files contain the actual UDO implementations for the various page types.

3. ###### Templates
You will need to include the correct snippet that implements the relevant data layer on each page. This means going to each template, or possibly section for certain pages such as the index ("home") page, and including the snippet. For example, the product.liquid template file in the default "Debut" theme that Shopify provides would look like the following, where the "product_udo" snippet has been added at the top of the file.

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

4. ###### Checkout
The checkout pages work differently than the rest of the pages. To implement Tealium on the order status page, you must add some code to the "Additional scripts" box of your order processing settings. According to the [Shopify documentation](https://help.shopify.com/themes/customization/order-status), you access it in the following way:

  > 1. From your Shopify admin, click Settings, and then click Checkout.
  > 2. Find the Order processing section. Near the bottom of this section you will find Additional content & scripts.

  And then you'll need to add the following code:

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

  Make sure to set "tealium_enabled" to true, and to set the "tealium_account", "tealium_profile", and "tealium_environment" variables to match what you configure in the settings. Unfortunately pages in the checkout don't have access to settings, so it's necessary to configure them again just for the order status page.
