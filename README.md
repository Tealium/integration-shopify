Tealium Shopify Integration
===========================

---------------------------

Summary
-------
With this integration, you can customize your Shopify theme to use Tealium. This code provides a Universal Data Object (UDO) and loads the necessary Tealium assets on each page of your site. This integration takes the form of a theme customization.

---------------------------

Setup Instructions
------------------

#### 1. Settings

Copy the following JSON object into the JSONarray contained in theConfig/settings_schema.json file of your Shopify theme. This will enable the Tealium-specific settings for your theme.

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

#### 3. Templates

On each page, you will need to include the correct snippet corresponding to the data layer of that page. To do that, go to the page's template, or possibly the section for certain pages such as the index ("home") page, and insert the relevant snippet. For example, the **product.liquid** template file in the default **Debut** theme that Shopify provides would look like the following, where the **product_udo** snippet has been added at the top of the file.

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

#### 4. Checkout

The checkout pages work differently than the rest of the pages. Unlike other pages, the checkout pages don't have access to the admin Settings, so you must reconfigure your Tealium account information. To do that, define the settings in the **Additional scripts** box of the **Order processing** section. According to the [Shopify documentation](https://help.shopify.com/themes/customization/order-status), you can access it in the following way:


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
