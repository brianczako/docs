---
title: Event
---


The [Zendesk](https://www.zendesk.com/) event integration supplements customer support by providing them additional context about their users; allowing them to provide a more personalized experience.

## Supported Features

This integration with Zendesk uses the Sunshine Events API [to track events against a Sunshine profile](https://developer.zendesk.com/rest_api/docs/sunshine/events_api#track-event-against-a-sunshine-profile).

## Prerequisites

The Zendesk integration uses the Sunshine Events API to track events against a Sunshine profile. You need to ensure your Sunshine plan is properly configured to use the integration. You can check this by accessing the subscription page of your account at h<span>ttps://</span>**your-zendesk-subdomain**.zendesk.com/admin/billing/subscription

1.  A [Zendesk](https://www.zendesk.com) Account.
2.  A Zendesk Sunshine Professional Plan or higher. 
    * Zendesk Sunshine Professional is the minimum required, but we recommend Enterprise or higher to avoid hitting the Sunshine API limit. 
    * For more information check [Plan Availability](https://developer.zendesk.com/rest_api/docs/sunshine/introduction#plan-availability).
3.  Account Verification. 
    * See [verifying an end user's email address](https://support.zendesk.com/hc/en-us/articles/203663786-Verifying-a-user-s-email-address) for more details.
4.  A Zendesk Sunshine [API token](https://developer.zendesk.com/rest_api/docs/support/introduction#api-token).

## Data Processing Notes

When you create a connection to Zendesk, forwarding for all existing data points will initially be set to 'off', and forwarding new data points by default will be disabled. You must enable all data points that you wish to forward to Zendesk.

mParticle recommends working with your Zendesk account manager before forwarding all events to your Zendesk account in the event [Filter](/guides/platform-guide/data-filter/). This will ensure that you have a sufficient plan to handle the additional events.

mParticle will only forward events to Zendesk if:

1.  An email address has been provided as identity in the event
2.  The payload is lower than ~2 Kilobytes - [Event maximum payload](https://develop.zendesk.com/hc/en-us/community/posts/360034212634-Event-maximum-payload)
3.  If a commerce payload is greater than ~2  Kilobytes, it will be split into separate requests for each product.  The description property will contain information on the set it belongs to; e.g., 1 of N. Please see specific commerce event types to learn more.

### Enabling User Profiles And Events In Zendesk

After you have finished configuring the integration in mParticle, and it has began forwarding events to Zendesk, you have to enable which of these 'user profiles' and events **that have already been forwarded** will be visible in the 'customer context'. To do this please follow the instructions on how to [Add Sunshine user profiles and events to customer context in a ticket](https://support.zendesk.com/hc/en-us/articles/360044235113)

## Forwarded JSON Payload Example
This is an example of a JSON payload generated and forwarded by mParticle to Zendesk API
~~~json
{
    "profile": {
        "source": "mParticle",
        "type": "customer",
        "identifiers": [{
                "type": "email",
                "value": "some@email.com"
            }
        ]
    },
    "event": {
        "type": "some-app-event-name",
        "source": "mParticle",
        "properties": {
            "custom_attributes": {
                "some_attribute_name": "some-attribute-value"
            }
        }
    }
}
~~~
## Event Data Mapping

The following field mappings apply for all supported events in this integration.

Profile mappings:

| Parameter | Zendesk Field | mParticle Details
|---|---|---
| [Event Source](#connection-settings) | profile.source | Taken from [Configuration Settings](#connection-settings), default is `mParticle`
| [Profile Type](#connection-settings) | profile.type | Taken from [Configuration Settings](#connection-settings), default is `customer`
| user_attributes | profile.name | If `$FirstName` and `$LastName` attributes are defined, `name` attribute will be included
| user_identities | profile.identifiers | Must include an email identity type

Event mappings:

| Parameter | Zendesk Field | mParticle Details
|---|---|---
| [Event Source](#connection-settings) | event.source | Taken from [Configuration Settings](#connection-settings), default is `mParticle`
| - | event.type | Check specific type of event for respective mapping of this field
| custom_attributes | event.properties.custom_attributes | All attribute items will be added to custom_attributes property
| - | event.description | Only set on commerce events that have been split into a set of events, it will contain a text describing the part of the set it belongs to; e.g., 1 of N.

### Screen View Events

| Parameter | Zendesk Field | mParticle Details
|---|---|---
| - | event.type | `Screen View` string will be sent as value for this field
| screen_name | event.properties.screen_name | The value sent in screen_name property in the event

Screen View (event) JSON Sample
~~~json
{
    "type": "Screen View",
    "source": "mParticle",
    "properties": {
        "screen_name": "About",
        "custom_attributes": {
            "some_attribute_name": "some-attribute-value"
        }
    }
}
~~~

### Custom Events

| Parameter | Zendesk Field | mParticle Details
|---|---|---
| event_name | event.type | The value sent in event_name property in the event
| custom_event_type | event.properties.custom_event_type | The value sent in custom_event_type property in the event. e.g.: UserPreference, Navigation, ProductCheckout

Custom Event (event) JSON Sample
~~~json
{
    "type": "app event name",
    "source": "mParticle",
    "properties": {
        "custom_event_type": "Other",
        "custom_attributes": {
            "some_attribute_name": "some-attribute-value"
        }
    }
}
~~~

### Commerce Events

#### Product-based
| Parameter | Zendesk Field | mParticle Details
|---|---|---
| event_name | event.type | The value sent in event_name property in the event
| action | event.properties.action | The value sent in action property of the product_action property in the event, e.g.: AddToCart, Checkout, RemoveFromWishlist
| transaction_id | event.properties.transaction_id | The value sent in transaction_id property of the product_action property in the event
| currency_code | event.properties.currency_code | The value sent in currency_code property in the event
| total_amount | event.properties.total_amount | The value sent in total_amount property of the product_action property in the event
| tax_amount | event.properties.tax_amount | The value sent in tax_amount property of the product_action property in the event
| shipping_amount | event.properties.shipping_amount | The value sent in shipping_amount property of the product_action property in the event
| coupon_code | event.properties.coupon_code | The value sent in coupon_code property of the product_action property in the event
| timestamp_unixtime_ms | event.properties.timestamp_unixtime_ms | The value sent in timestamp_unixtime_ms property in the event
| products | event.properties.products | The array of products sent in products property of the product_action property in the event

Product-Based JSON Sample
~~~json
{
    "type": "eCommerce - AddToCart",
    "source": "mParticle",
    "properties": {
        "action": "AddToCart",
        "transaction_id": "some-transaction-id",
        "currency_code": "USD",
        "total_amount": "1.99",
        "tax_amount": "0.50",
        "shipping_amount": "1.00",
        "coupon_code": "some-coupon-code",
        "timestamp_unixtime_ms": "1605805232479",
        "products": [{
                "id": "some-product-id",
                "name": "some-product-name",
                "brand": "some-product-brand",
                "category": "some-product-category",
                "position": "1",
                "price": "1.99",
                "quantity": "1",
                "coupon_code": "some-product-coupon-code",
                "total_product_amount": "1.99",
                "product_attributes": {
                    "some_product_attribute_name": "some-product-attribute-value"
                }
            }
        ],
        "custom_attributes": {
            "some_attribute_name": "some-attribute-value"
        }
    }
}
~~~

<aside class="notice">If request size is greater than ~2 Kilobytes, we will try to split it into multiple request messages, each containing a specific product (1 of N). If the request can't be split or split requests are still too big, the main request will be dropped</aside>

#### Promotion-based
| Parameter | Zendesk Field | mParticle Details
|---|---|---
| event_name | event.type | The value sent in event_name property in the event
| action | event.properties.action | The value sent in action property of the promotion_action property in the event, e.g.: PromotionView, PromotionClick
| timestamp_unixtime_ms | event.properties.timestamp_unixtime_ms | The value sent in timestamp_unixtime_ms property in the event
| promotions | event.properties.promotions | The array of promotions sent in promotions property of the promotion_action property in the event

Promotion-Based JSON Sample
~~~json
{
    "type": "eCommerce - PromotionView",
    "source": "mParticle",
    "properties": {
        "action": "PromotionView",
        "promotions": [{
                "id": "some-promotion-id",
                "name": "some-promotion-name",
                "creative": "some-creative-name",
                "position": "some-position-value"
            }
        ],
        "custom_attributes": {
            "some_attribute_name": "some-attribute-value"
        }
    }
}
~~~

<aside class="notice">If request size is greater than ~2 Kilobytes, the integration will try to split it into multiple request messages, each containing a specific promotion (1 of N). If the request can't be split or split requests are still too big, the main request will be dropped</aside>

#### Impression-based
| Parameter | Zendesk Field | mParticle Details
|---|---|---
| event_name | event.type | The value sent in event_name property in the event
| product_impressions | event.properties.product_impressions | The array of product impressions sent in product_impressions property in the event

Impression-Based JSON Sample
~~~json
{
    "type": "eCommerce - Impression",
    "source": "mParticle",
    "properties": {
        "product_impressions": [{
                "product_impression_list": "some-product-impresion-list-name",
                "products": [{
                        "id": "some-product-id",
                        "name": "some-product-name",
                        "brand": "some-product-brand",
                        "category": "some-product-category",
                        "position": "1",
                        "price": "1.99",
                        "quantity": "1",
                        "coupon_code": "some-product-coupon-code",
                        "total_product_amount": "1.99",
                        "product_attributes": {
                            "some_product_attribute_name": "some-product-attribute-value"
                        }
                    }
                ]
            }
        ],
        "custom_attributes": {
            "some_attribute_name": "some-attribute-value"
        }
    }
}
~~~

<aside class="notice">If request size is greater than ~2 Kilobytes, we will try to split it into multiple request messages, each containing a specific product (1 of N). If the request can't be split or split requests are still too big, the main request will be dropped</aside>

## Configuration Settings

| Setting Name |  Data Type | Default Value  | Description |
| ---|---|---|---|
| Authentication Email Address | `string` | <unset> | Email address for API authentication |
| API Token | `string` | <unset> | API tokens are auto-generated passwords in the Support admin interface on Zendesk platform |
| Subdomain | `string` | <unset> | The Subdomain can be identified from the Zendesk account's URL, example: <https://your-zendesk-subdomain.zendesk.com> | 

## Connection Settings

| Setting Name |  Data Type    | Default Value | Platform | Description |
| ---|---|---|---|-----
| Event Source | `string` | mParticle | All | Application which sent the event |
| Profile Type | `string` | customer | All | Is a user-defined name that lets you create different kinds of profiles for a given source. Example: "customer" for the customer profiles |

<aside class="notice">'zendesk' is a protected source name for Zendesk standard events. Any attempts to use this source when creating an event results in an error</aside>

<aside class="notice">When saving Connection Settings, access to Zendesk platform will be validated using the provided information (auth. email, api token and subdomain), if this process fail, an error will be shown in the configuration screen</aside>
