# Cardoo API
The Cardoo API allows you to manage orders and request prices for our products in a simple way using conventional HTTP requests.

#### Requests
All requests should be made using the HTTPS protocol so that traffic is encrypted.

When passing parameters in a POST request, parameters must be passed as a JSON object containing attribute names and values as key-value pairs. You must also set `Content-Type` header to `application/json`.

#### Authentication
All requests to Cardoo API must be authenticated.

In order to make an authenticated request, include an  `Authorization` header containing your token as follows: `Access-Token: YOUR_TOKEN`.

**TODO:** Add information on how to receive the Token.

#### Responses
In general, if the status of the response is in the 200 range, it indicates that the request was fulfilled successfully and no error was encountered.

Return codes in the 400 range indicate some issue with the request that was sent. This could mean that you did not authenticate, or an object with given attributes could not be created.

A response status in 500 range indicates a server-side problem.

#### Error Objects
When you get a response in 400 range, it will contain an error object with code and a message:

```json
{
  "status": 422,
  "message": "Deposit amount has to be greater than zero"
}
```

#### Other Conventions
Date and time values must be formatted according ISO 8601.

Monetary values are represented as an object with value in cents and currency in ISO format. For example, 11.54 euros must be passed as 1154 cents:

```json
{
  "cents": 1154,
  "currency_iso": "eur"
}
```



## Price Request
Price request endpoint allows you to request a price of our product for a customer.


### POST /partner_api/price_request
To create a price request that can later be used for creating an order, send a `POST` request with parameters of a rental.

When the price was successfully calculated, the response status would be `201 Created`.

#### Request Body Schema

Example request body:
```json
{
  "vehicle": "Tesla Roadster",
  "deposit_amount": {
    "cents": 500000,
    "currency_iso": "EUR"
  },
  "pickup_time": "2021-12-24T10:00:00Z",
  "dropoff_time": "2021-12-38T10:00:00Z"
}
```

###### vehicle

Vehicle name as a string.

###### deposit_amount
Deposit amount for the vehicle as a [Money Object](#other-conventions).

###### pickup_time

Time when the rental is supposed to begin.

###### dropoff_time

Time when the rental is supposed to end.

#### Response Body Schema
Example response body:
```json
{
  "price_request_id": "bbeab7d5-c80c-4cba-a3ad-2fd6574baa8b",
  "timestamp": "2021-12-20T12:15:38Z",
  "price": {
    "cents": 2800,
    "currency_iso": "EUR"
  }
}
```

###### price_request_id
UUID for the price request. Must be used for [creating an order](#post-partner_apiordersonetime).

###### price
Price for Cardoo product for the customer as a [Money Object](#other-conventions).

###### timestamp
Time when the price was calculated.



## One-time orders
One-time orders endpoints allow you to create orders for customers that don’t have a subscription at Cardoo.


### POST /partner_api/orders/onetime
To create a one-time order for a customer, send a `POST` request with parameters of a rental, price request ID, and required callbacks.

When the one-time order was successfully calculated, the returned response status would be `201 Created`.

#### Request Body Schema

Example request body:
```json
{
  "partner_order_id": "X666",
  "price_request_id": "bbeab7d5-c80c-4cba-a3ad-2fd6574baa8b",
  "customer": {
    "first_name": "Jane",
    "last_name": "Doe",
    "email": "jane@gmail.com",
    "phone_number": "+1-541-754-3010"
  },
  "pickup_time": "2021-12-24T10:00:00Z",
  "dropoff_time": "2021-12-38T10:00:00Z",
  "city": "Italy",
  "country": "Florence",
  "vehicle": "BMW I5",
  "callbacks": {
    "guarantee_state_changed": "https://callback_url"
  },
  "redirects": {
    "success_url": "https://success",
    "failure_url": "https://failure_url"
  },
  "acquiring": {
    "prepay": "full",
    "price": {
      "cents": 10000,
      "currency_iso": "EUR"
    }
  },
  "deposit_free": {
    "price": {
      "cents": 2800,
      "currency_iso": "EUR"
    },
    "deposit_amount": {
      "cents": 500000,
      "currency_iso": "EUR"
    }
  }
}
```

###### partner_order_id

Order ID in your system

###### price_request_id

This is the ID that you got when [requesting the price](#post-partner_apiprice_request)

###### customer

Customer object

 `first_name`
Customer first name

 `last_name`
Customer last name

 `email`
Customer email

 `phone_number`
Customer phone number with country code in E.164 format.

###### pickup_time

Time when the rental is supposed to begin.

###### dropoff_time

Time when the rental is supposed to end.

###### country

Country name as in ISO_3166.

###### city

City name.

###### vehicle

Vehicle name as a string.

###### callbacks

*Optional* Object with callbacks URLs. You can find more details about callbacks logic in [Callbacks](#callbacks) section.

 `guarantee_state_changed`
URL that will be triggered with a POST request when Cardoo guarantee state changes.

###### redirects

Object with URLs where the customer will be redirected to.
TODO: check why it is required

`success_url`
The URL for redirecting the customer in case of successfully completing a transaction with Cardoo.

`failure_url`
The URL for redirecting the customer in case he fails to complete a transaction with Cardoo.

###### acquiring

*Optional* Object with the amount that must be acquired from the customer for the rental by Cardoo.

`prepay`
*Optional* Indicates whether the acquiring sum is full rental price or just part of it. Can be  `full` or `partial`.

 `price`
Amount that must be acquired [Money Object](#other-conventions).

**deposit_fee**

`price`
Price for Cardoo product for the customer as a [Money Object](#other-conventions).

`deposit_amount`
Deposit amount for the vehicle as a [Money Object](#other-conventions).

#### Response Body Schema
Example response body:
```json
{
  "order_id": "878246a6-b2cb-41fa-80d0-6cb97e569836",
  "invoice_id": "56efa4bd-2e26-4fcf-98a7-bc2bd6339a59",
  "customer_flow_url": "https://cardoo.finance/orders/878246a6-b2cb-41fa-80d0-6cb97e569836/flow"
}
```

###### order_id
UUID of the created one-time order.

###### invoice_id
UUID of the invoice for the created one-time order.
TODO: check why do we need to expose this

###### customer_flow_url
URL where the customer must be redirected for starting a transaction with Cardoo.
