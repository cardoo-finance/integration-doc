# Cardoo Integration Guide

## Introduction
Our solution implements all the crucial steps, which are: 
* Calculating the price of our product for a specific car rental offer
* Receiving a payment from a customer (incl. some prepay amount for a car rental)
* Identification and verification of a customer
* Communicating with your system via API regarding the status of verification/approval process

These steps will be described in following sections from the points of view of a customer and your system.

### Customer’s Point of View
A customer is browsing a website of one of our partners and looking for a car to rent for a specified time interval. 
When they open a detailed view of one of the rental offers there’s a section with additional services which includes our product - Deposit Free - with a price. 
If a customer chooses to use our solution and proceeds to book, they get redirected to our Customer Flow page where they: 
* pay for an order 
* upload their ID document. 

After completion of these steps the order undergoes the approval process which includes verification of a customer.
When the decision is made to either approve or reject the order, the corresponding email will be sent to a customer. In case of approval a customer will also receive an issued guarantee agreement. In case of rejection we will unhold the money on a customer's card within 24 hours.

### Your System’s Point of View
Your system forms the list of additional services for a specific car rental offer. To get a price for Cardoo’s Deposit Free your system has to make the price request to our system.

When a customer chooses our solution and starts the booking process, your system will need to make the order creation request to our system and in response will get an URL to the Customer Flow. Your system will need to redirect a customer to that URL instead of your usual payment page.

After our order approval process has been completed, your system will receive a signal which will not contain the actual decision. Upon receiving that signal your system should make a request to our system to get our decision. 


# API Documentation
The Cardoo API allows you to manage orders and request prices for our products in a simple way using conventional HTTP requests.

#### Requests
All requests should be made using the HTTPS protocol so that traffic is encrypted.

When passing parameters in a POST request, parameters must be passed as a JSON object containing attribute names and values as key-value pairs. You must also set `Content-Type` header to `application/json`

#### Authentication
All requests to Cardoo API must be authenticated.

In order to make an authenticated request, include an  `Authorization` header containing your token as follows: `Access-Token: YOUR_TOKEN`

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

`vehicle`
Vehicle name as a string.
**TODO:** Specify the format???

`deposit_amount`
Deposit amount for the vehicle as a [Money Object]().

`pickup_time`
Time when the rental is supposed to begin.

`dropoff_time`
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

`price_request_id`
UUID for the price request.

`price`
Price for the customer as a [Money Object]().

`timestamp`
Time when the price was calculated.

