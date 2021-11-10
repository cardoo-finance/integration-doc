# Cardoo Integration Guide

## Table of Contents

- [Definitions](#definitions)
- [Flow Overview](#flow-overview)
  - [Customer’s Point of View](#customers-pov)
  - [Your System’s Point of View](#your-systems-pov)
- [Testing Guideline](#testing-guideline)
- [API Reference](#api-reference)
  - [Overview](#api-overview)
  - [One-time orders](#api-orders-onetime)
    - [Deposit Free Price Request (POST)](#api-orders-onetime-price-request)
    - [Creating an Order (POST)](#api-orders-onetime-post)
    - [Getting Order States Details (GET)](#api-orders-onetime-get)
  - [Callbacks](#api-callbacks)

## Definitions <a name="definitions"></a>
- **Acquiring** - Receiving car rental fee from customers' bank payment cards and transferring it to the car rental company in scope of the sales agency contract between Cardoo and the car rental company. 
- **Customer** - A person renting a car 
- **Deposit Free** - A service enabling a customer to rent a car without making a security deposit 
- **Issued Guarantee** - Cardoo’s legally binding commitment to pay to the car rental company the value of possible losses arisen from the respective car rental
- **Guarantee Status** - Cardoo’s guarantee status indicating whether such guarantee has been issued or denied 
- **Order** - Cardoo’s Deposit Free service request by a customer with respect to an underlying car rental booking
- **One-time Order** - A type of order with fixed pickup and drop-off dates and no recurring payments.
- **Order Approval Process** - Cardoo’s internal procedures related to the customer assessment and resolution on the Guarantee Status 
- **Deposit Free Price Request** - A request to calculate the price of the Deposit Free service with respect to the specified car rental parameters 


## Flow Overview <a name="flow-overview"></a>
Our solution implements all the crucial steps, which are: 
* Calculating the price of our product for a specific car rental offer
* Receiving a payment from a customer (incl. some prepay amount for a car rental)
* Identification and verification of a customer
* Communicating with your system via API regarding the status of verification/approval process

These steps will be described in the following sections from the points of view of a customer and your system.

### Customer Experience  <a name="customers-pov"></a>

A customer may browse our partner car rental company’s web site for a specified time interval. When they open a detailed view of a rental offer, they find a section with additional services which includes our product Deposit Free and its price. If a customer chooses to use our solution and proceeds to book, they get redirected to our Customer Flow page where they:
* pay for an order 
* upload their ID document. 

After completion of these steps the order undergoes the approval process which includes KYC procedure. 

When the decision is made to either approve or reject the order, the corresponding email will be sent to the customer. 

In case of approval, the customer will also receive an issued guarantee agreement. 

In case of rejection, we will unblock the money on the customer’s payment card within 24 hours.

### Your System Routine <a name="your-systems-pov"></a>
Your system creates the list of additional services for a specific car rental offer. To get a price for Cardoo’s Deposit Free your system has to make the price request to our system.

When a customer chooses our solution and starts the booking process, your system will need to make an order creation request to our system and in response it will get an URL to the Customer Flow. Your system will redirect the customer to that URL instead of your usual payment page.

After our order approval process has been completed, your system will receive a signal which will not contain the actual decision. Upon receiving that signal your system should make a request to our system to get our decision. 

## Testing Guideline <a name="testing-guideline"></a>
Please use the sandbox mode to test integrations.
There are a few key differences from the production mode:
- No real bank card will be accepted, please use a test card data. 
- Automatic customer identification and verification process is not available, but you can still experience this part of the process interacting as a customer.
- You will have to manually issue or deny the Guarantee (this is available only in the sandbox mode). 

To use the sandbox mode, please, use the following URL to send HTTP requests: <https://sandbox-api.cardoo.finance>

### Bank Payment Card Test Data
You can enter the following payment card details on the payment page of our Customer Flow in order to successfully pay:

**Bank card number**: 4111 1111 1111 1111

**Expiry date, MM/YY**: any date later or equal to the present

**CSC/CVV**: 000

**Card holder name**: any combination of English letters and spaces


### Issuing or Denying the Guarantee

**Note** Described API endpoints are only available in Sandbox.

#### POST /partner_api/sandbox/orders/:order_id/guarantee_issuance/issue
For emulating issuance of the guarantee by Cardoo, send a POST request with empty body to this URL.

When the guarantee issuance status has been successfully changed to "issued", the response would have status `200 OK` and an empty body. This would also trigger callback `guarantee_status_changed`, if you have specified it.


#### POST /partner_api/sandbox/orders/:order_id/guarantee_issuance/deny
For emulating denial of the guarantee issuance by Cardoo, send a POST request with empty body to this URL.

When the guarantee issuance status has been successfully changed to "denied", the response would have status `200 OK` and an empty body. This would also trigger callback `guarantee_status_changed`, if you have specified it.



# API Reference <a name="api-reference"></a>
## Overview  <a name="api-overview"></a>
The Cardoo API allows you to manage orders and request prices for our products in a simple way using conventional HTTP requests.

#### Requests
All requests should be made using the HTTPS protocol so that traffic is encrypted.

When passing parameters in a POST request, parameters must be passed as a JSON object containing attribute names and values as key-value pairs. You must also set `Content-Type` header to `application/json`.

#### Authentication
All requests to Cardoo API must be authenticated.

In order to make an authenticated request, please include a header containing your token as follows: `Access-Token: YOUR_TOKEN`.

To receive the access token, please, contact us via <sales@cardoo.finance>. 

#### Responses
In general, if the status of the response is in the 200 range, it indicates that the request was fulfilled successfully and no error was encountered.

Return codes in the 400 range indicate some issue with the request that was sent. This could mean that you did not authenticate, or an object with given attributes could not be created.

A response status in 500 range indicates a server-side problem.

#### Error Object
When you get a response in 400 range, it will contain an error object with code and a message, for example:

```json
{
  "errors": {
    "partner_order_id": [
      "is empty"
    ]
  }
}
```

#### Date and Time Values
Date and time values must be formatted according ISO 8601.

#### Money Object

Monetary values are represented as an object with value in cents and currency in ISO format. For example, 11.54 euros must be passed as 1154 cents:

```json
{
  "cents": 1154,
  "currency_iso": "eur"
}
```

## Deposit Free Price Request <a name="api-orders-onetime-price-request"></a>
Price request endpoint allows you to request a price of our product for a customer.


### POST /partner_api/orders/onetime/deposit_free_price_request
To create a deposit free price request that can later be used for creating a one-time order, send a `POST` request with parameters of a rental.

When the price was successfully calculated, the response status would be `201 Created`.

#### Request Body Schema

Example request body:
```json
{
  "vehicle": "Tesla Roadster",
  "session_cookie_id": 1,
  "deposit_amount": {
    "cents": 500000,
    "currency_iso": "EUR"
  },
  "pickup_time": "2021-12-24T10:00:00Z",
  "dropoff_time": "2021-12-27T10:00:00Z"
}
```

###### vehicle

Vehicle name as a string.

###### session_cookie_id

Session cookie ID of the customer in your system.

###### deposit_amount
Deposit amount for the vehicle as a [Money Object](#money-object).

###### pickup_time

Time when the rental is supposed to begin.

###### dropoff_time

Time when the rental is supposed to end.

#### Response Body Schema
Example response body:
```json
{
  "deposit_free_price_request_id": "76aed270-4b39-44ca-b6c2-a1b8d0b67288",
  "timestamp": 1631556435543,
  "price": {
    "cents": 12000,
    "currency_iso": "EUR"
  }
}
```

###### deposit_free_price_request_id
UUID for the deposit free price request. Must be used for [creating an order](#api-orders-onetime-post).

###### price
Price for Cardoo product for the customer as a [Money Object](#money-object).

###### timestamp
Time when the price was calculated.



## One-time orders <a name="api-orders-onetime"></a>
One-time orders endpoints allow you to create orders for customers.


### POST /partner_api/orders/onetime <a name="api-orders-onetime-post"></a>
To create a one-time order for a customer, send a `POST` request with parameters of a rental, price request ID, and required callbacks.

When the one-time order was successfully calculated, the returned response status would be `201 Created`.

#### Request Body Schema

Example request body:
```json
{
  "partner_order_id": "X666",
  "customer": {
    "first_name": "Jane",
    "last_name": "Doe",
    "email": "jane@gmail.com",
    "phone_number": "+1-541-754-3010"
  },
  "pickup_time": "2021-12-24T10:00:00+02:00",
  "dropoff_time": "2021-12-28T10:00:00+02:00",
  "city": "Florence",
  "country": "Italy",
  "vehicle": "BMW I5",
  "callbacks": {
    "guarantee_status_changed": "https://callback_url"
  },
  "redirects": {
    "success_url": "https://success",
    "failure_url": "https://failure_url"
  },
  "deposit_free": {
    "deposit_free_price_request_id": "76aed270-4b39-44ca-b6c2-a1b8d0b67288",
    "deposit_amount": {
      "cents": 500000,
      "currency_iso": "EUR"
    }
  },
  "acquiring": {
    "prepay": "full",
    "price": {
      "cents": 10000,
      "currency_iso": "EUR"
    }
  }
}
```

###### partner_order_id

Order ID in your system

###### customer

Customer object

 `first_name`
 *Optional* Customer first name

 `last_name` 
 *Optional* Customer last name

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

*Optional* Vehicle name as a string.

###### callbacks

*Optional* Object with callbacks URLs. You can find more details about callbacks logic in [Callbacks](#callbacks) section.

 `guarantee_status_changed`
URL that will be triggered with a POST request when Cardoo guarantee status changes.

###### redirects

*Optional* Object with URLs where the customer will be redirected to.

`success_url`
The URL for redirecting the customer in case of successfully completing a transaction with Cardoo.

`failure_url`
The URL for redirecting the customer in case they fail to complete a transaction with Cardoo.

###### acquiring

*Optional* See [Definitions](#definitions) for the description of our Acquiring service.

`prepay`
*Optional* Indicates whether the full rental price is subject for our acquiring or just a part of it. Value can either be `full` or `partial`.

 `price`
Amount that must be charged from a customer and transferred to the car rental company [Money Object](#money-object).

**deposit_free**

`deposit_free_price_request_id`
This is the ID that you got by [requesting the Deposit Free price](#api-orders-onetime-price-request)

`deposit_amount`
Deposit amount for the vehicle as a [Money Object](#money-object).

#### Response Body Schema
Example response body:
```json
{
  "order_id": "3a58ed01-9fab-45b4-b22b-c84478e68f1d",
  "customer_flow_url": "https://customer-flow.cardoo.finance/orders/3a58ed01-9fab-45b4-b22b-c84478e68f1d/flow"
}
```

###### order_id
UUID of the created one-time order.

###### customer_flow_url
URL where the customer must be redirected for starting a transaction with Cardoo.



## Getting Order States Details <a name="api-orders-onetime-get"></a>
Orders endpoint allows you to get order information, along with its guarantee issuance state and payment state.


### GET /partner_api/orders/onetime/:id
To get order details, send a `GET` request with order ID.


#### Response Body Schema
Example response body:
```json
{
  "order_id": "3a58ed01-9fab-45b4-b22b-c84478e68f1d",
  "guarantee_issuance_status": "issued",
  "invoice_state": "paid"
}
```

###### order_id
UUID of the one-time order.

###### guarantee_issuance_status
Cardoo guarantee issuance status of the order. It may be one of the following:

 - `pending` - decision hasn’t been made yet
 - `issued` - guarantee has been successfully issued
 - `denied` - guarantee has been denied

###### invoice_state
Payment state of the order. It may be one of the following:

- `issued` - invoice was issued, but not yet paid
- `paid` - invoice was paid
- `hold` - invoice amount is pu on hold on the customer payment card
- `cancelled` - invoice was cancelled


## Callbacks <a name="api-callbacks"></a>
Cardoo uses callbacks for notifying partners about changes in orders.

When the guarantee status changes, we will make a POST request with a JSON body to a URL that you specified when creating an order.

In JSON body we send you 
```JSON
{
  "order_id": "3a58ed01-9fab-45b4-b22b-c84478e68f1d",
  "partner_order_id": "X666"
}
```

You can use a distinct URL per each order or a universal one and rely on `order_id` (our internal ID) or `partner_order_id` (an ID you've specified during an order creation) value in a request's body.

If the URL that you provided as a callback URL is not available when we trigger the callback request, or returns a 500 status code, we will retry the request with increasing time intervals.

#### Guarantee status changed
This callback is triggered, when Cardoo issues or denies guarantee to the customer.
