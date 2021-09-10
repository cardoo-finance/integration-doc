# Introduction
Our solution implements all the crucial steps, which are: 
* Calculating the price of our product for a specific car rental offer
* Receiving a payment from a customer (incl. some prepay amount for a car rental)
* Identification and verification of a customer
* Communicating with your system via API regarding the status of verification/approval process

These steps will be described in following sections from the points of view of a customer and your system.

## Customer’s Point of View
A customer is browsing a website of one of our partners and looking for a car to rent for a specified time interval. 
When they open a detailed view of one of the rental offers there’s a section with additional services which includes our product - Deposit Free - with a price. 
If a customer chooses to use our solution and proceeds to book, they get redirected to our Customer Flow page where they: 
* pay for an order 
* upload their ID document. 
After completion of these steps the order undergoes the approval process which includes verification of a customer.
When the decision is made to either approve or reject the order, the corresponding email will be sent to a customer. In case of approval a customer will also receive an issued guarantee agreement. In case of rejection we will unhold the money on a customer's card within 24 hours.

## Your System’s Point of View
Your system forms the list of additional services for a specific car rental offer. To get a price for Cardoo’s Deposit Free your system has to make the price request to our system.
When a customer chooses our solution and starts the booking process, your system will need to make the order creation request to our system and in response will get an URL to the Customer Flow. Your system will need to redirect a customer to that URL instead of your usual payment page.
After our order approval process has been completed, your system will receive a signal which will not contain the actual decision. Upon receiving that signal your system should make a request to our system to get our decision. 


