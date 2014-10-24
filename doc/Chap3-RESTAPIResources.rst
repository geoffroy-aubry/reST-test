.. _Chap3-RESTAPIResources:

==============================
Chapter 3 - REST API Resources
==============================

The following table lists the REST API resources used.

==================================================================  =======================================================
Resource        		                                            Description
==================================================================  =======================================================
**POST**  /rest/v1/order	                                        Request a new order.
**POST**  /rest/v1/maintenance/transaction/{transaction_reference}  Perform a maintenance on an existing transaction
**POST**  /rest/v1/hpayment		                                    Request an order and initialize a hosted payment page
**GET**   /rest/v1/transaction						                Request information about an existant transaction.
==================================================================  =======================================================

-------------------
Request a New Order
-------------------
Overview
  To request a new order, make an HTTP POST request to the following resource URL.
  POST /rest/v1/order


Order Parameters
----------------

.. table:: Table: Order-related parameters

  ====================  ===========  =======  ========  =====================================================================================================================================================================================================================================================================
  Field Name        	Format [1]_  Length   Req [2]_  Description
  ====================  ===========  =======  ========  =====================================================================================================================================================================================================================================================================
  orderid               AN           32       M         Unique order id
  operation             AN                              Transaction type.
                                                        Indicates how you want to process the payment. The default transaction type is set in the Merchant Interface (Default payment procedure in the Integration section). A transaction type sent along with the transaction will overwrite the default payment procedure.
                                                        - **Sale** indicates transaction is sent for authorization, and if approved, is automatically submitted for capture.
                                                        - **Authorization** indicates this transaction is sent for authorization only. The transaction will not be sent for settlement until the transaction is submitted for capture manually by the Merchant
  payment_product       AN                    M         The payment product (e.g., visa, mastercard, ideal).
                                                        Depending on the payment product, elements specific to the payment method are required (see following tables).
                                                        Refer to the appendices —*"Appendix A. Payment Products”*— for the full list of available payment products.
  description           AN           255      M         The order short description.
  long_description      AN                              Additional order description.
  currency              A            3        M         Base currency for this order (Default to EUR).
                                                        This three-character currency code complies with ISO 4217.
  amount                R                     M         The total order amount. It should be calculated as a sum of the items purchased, plus the shipping fee (if present), plus the tax fee (if present).
  shipping              R                               The order shipping fee (Default to zero).
                                                        It can be omitted if the shipping fee value is zero.
  tax                   R                               The order tax fee (Default to zero).
                                                        It can be omitted if the order tax value is zero.
  cid                   AN                    M         Unique customer id.
                                                        *For fraud detection reasons.*
  ipaddr                AN                    M         The IP address of your customer making a purchase.
  accept_url            AN                              The URL to return your customer to once the payment process is completed successfully.
  decline_url           AN                              The URL to return your customer to after the acquirer declines the payment.
  pending_url           AN                              The URL to return your customer to when the payment request was submitted to the acquirer but response is not yet available.
  exception_url         AN                              The URL to return your customer to after a system failure.
  cancel_url            AN                              The URL to return your customer to when he or her decides to abort the payment.
  http_accept           AN                              This element should contain the exact content of the HTTP "Accept" header as sent to the merchant from the customer's browser (Default to "*/*").
  http_user_agent       AN                              This element should contain the exact content of the HTTP "User-Agent" header as sent to the merchant from the customer's browser (Default to "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.0)").
  device_fingerprint    AN                              This element should contain the value of the “ioBB” hidden field. (Please refer to *“Chapter 8: Device fingerprint integration”*)
  language              AN                              Locale code of your customer (Default to **en_GB** – English – Great Britain).
                                                        It may be used for sending confirmation emails to your customer or for displaying payment pages.

                                                        Examples:
                                                        - en_GB
                                                        - fr_FR
                                                        - es_ES
                                                        - it_IT
                                                        - …
  cdata1                AN                              Custom data. You may use these parameters to submit values you wish to receive back in the API response messages or in the notifications, e.g. you can use these parameters to get back session data, order content or user info.
  cdata2
  cdata3
  cdata4
  ====================  ===========  =======  ========  =====================================================================================================================================================================================================================================================================


Customer Parameters
-------------------
Overview
  The merchant can/must send the following customer information along with the transaction details.

The following table lists the customer related parameters

.. table:: Table: Customer-related parameter

  ====================  ===========  =======  ========  =====================================================================================================================================================================
  Field Name            Format [1]_  Length   Req [2]_  Description
  ====================  ===========  =======  ========  =====================================================================================================================================================================
  email                 AN                    M         The customer's e-mail address.
  phone                 AN                              The customer's phone number.
  birthdate             N            8                  Birth date of the customer (YYYYMMDD).
                                                        **For fraud detection reasons.**
  birthdate             A            1                  Gender of the customer (M=male, F=female, U=unknown).
  firstname	            AN                    M         The customer's first name.
  lastname              AN                    M         The customer's last name.
  recipientinfo         AN                              Additional information about the customer (e.g., quality or function, company name, department, etc.).
  streetaddress         AN                              Street address of the customer.
                                                        It can be omitted if the shipping fee value is zero.
  streetaddress2        AN                              Additional address information of the customer (e.g., building, floor, flat, etc.).
  city                  AN                              The customer's city.
  state                 AN                              The USA state or the Canada state of the customer making the purchase. Send this information only if the address country of the customer is US (USA) or CA (Canada).
  zipcode               AN                              The zip or postal code of the customer.
  country               A            2        M         The country code of the customer.
                                                        This two-letter country code complies with ISO 3166-1 (alpha 2).
  ====================  ===========  =======  ========  =====================================================================================================================================================================

The following table lists the Parameters specific to shipping information

.. table:: Table: Parameters specific to shipping information

  ======================  =========  =======  =====================================================================================================================================================================
  Field Name        	  Format     Length   Description
  ======================  =========  =======  =====================================================================================================================================================================
  shipto_firstname        AN                  The first name of the order recipient.
  shipto_lastname         AN                  The last name of the order recipient.
  shipto_recipientinfo    AN                  Additional information about the order recipient (e.g., quality or function, company name, department, etc.).
  shipto_streetaddress    AN                  Street address to which the order is to be shipped.
  shipto_streetaddress2   AN                  The additional information about address to which the order is to be shipped (e.g., building, floor, flat, etc.).
  shipto_city             AN                  The city to which the order is to be shipped.
  shipto_state            AN                  The USA state or Canada state to which the order is being shipped. Send this information only if the shipping country is US (USA) or CA (Canada).
  shipto_zipcode          AN                  The zip or postal code to which the order is being shipped
  shipto_country          A           2       Country code to which the order is being shipped.This two-letter country code complies with ISO 3166-1 (alpha 2).
  ======================  =========  =======  =====================================================================================================================================================================


Parameters specific to the payment product
------------------------------------------
Overview
  Depending on the payment product, the Merchant is supposed to send additional request parameters.

The following table lists the Parameters specific to credit or debit card payments.

.. table:: Table: Parameters specific to credit or debit card payments

  =========================  ===========  =======  ========  =====================================================================================================================================================================
  Field Name        	     Format [1]_  Length   Req [2]_  Description
  =========================  ===========  =======  ========  =====================================================================================================================================================================
  cardtoken                  AN           40       M         Card token.
                                                             For further details about the card token and its integration, refer to the Secure Vault API documentation.
  eci                        N            1                  Electronic Commerce Indicator (ECI).
                                                             The ECI indicates the security level at which the payment information is processed between the cardholder and merchant.
                                                             Possible values:
                                                             1 = MO/TO (Card Not Present)
                                                             2 = MO/TO – Recurring
                                                             3 = Instalment Payment
                                                             4 = Manually Keyed (Card Present)
                                                             7 = E-commerce with SSL/TLS Encryption
                                                             9 = Recurring E-commerce
                                                             A default ECI value can be set in the preferences page. An ECI value sent along in the transaction will overwrite the default ECI value. Refer to the appendices (Appendix C) to get further information.

  authentication_indicator   N            1                  Indicates if the 3DS authentication should be performed. Can be used to overrule the merchant level configuration.
                                                             0 = Bypass authentication
                                                             1 = Continue if possible (Default)
  =========================  ===========  =======  ========  =====================================================================================================================================================================

The following table lists the Parameters specific to Qiwi Wallet

.. table:: Table: Parameters specific to Qiwi Wallet

  =========================  ===========  =======  ========  ===============================================================================
  Field Name        	     Format [1]_  Length   Req [2]_  Description
  =========================  ===========  =======  ========  ===============================================================================
  qiwiuser                   AN           12       M         The Qiwi user's ID, to whom the invoice is issued.
                                                             It is the user's phone number, in international format. Example: +79263745223
  =========================  ===========  =======  ========  ===============================================================================

The following table lists the Parameters specific to iDeal

.. table:: Table: Parameters specific to iDeal

  =========================  =======  =======  ====  =================================
  Field Name        	     Format   Length   Req   Description
  =========================  =======  =======  ====  =================================
  issuer_bank_id             AN        4       M     Issuers' bank Id list [ref1]_
  =========================  =======  =======  ====  =================================

.. [ref1] Table:Issuers’ bank Id list

===========  ===================
Field Name   Bank description
===========  ===================
ABNANL2A     ABN AMRO
INGBNL2A     ING
RABONL2U     Rabobank
SNSBNL2A     SNS Bank
ASNBNL21     ASN Bank
FRBKNL2L     Friesland Bank
KNABNL2H     Knab
RBRBNL21     SNS Regio Bank
TRIONL2U     Triodos bank
FVLBNL22     Van Lanschot
===========  ===================

Response Fields
---------------

Overview
  Depending on the payment product, the Merchant is supposed to send additional request parameters.

The following table lists and describes the response fields.

============================  =====================================================================================================================================================================
Field Name                    Description
============================  =====================================================================================================================================================================
state                         transaction state.

                              Value must be a member of the following list.

                              - completed
                              - forwarding
                              - pending
                              - declined
                              - error

                              Please report to the following section below — Transaction Workflow — for further details.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
reason                        optional element. Reason why transaction was declined.
code                          reason code as described in the appendices.
message                       reason description.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
forwardUrl (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
forward_url (xml)             optional element. Merchant must redirect the customer's browser to this URL.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
test                          true if the transaction is a testing transaction, otherwise false.
mid                           your merchant account number (issued to you by HiPay TPP).
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
attemptId (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
attempt_id (xml)              attempt id of the payment.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
authorizationCode (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
authorization_code (xml)      an authorization code (up to 35 characters) generated for each approved or pending transaction by the acquiring provider.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
transactionReference (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
transaction_reference (xml)   the unique identifier of the transaction.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
referenceToPay (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
reference_to_pay (xml)        In some payment methods the customer can receive a reference to pay, at this point, the customer has the option to physically paying with cash at any bank branch, or at authorized processors such as drugstores, supermarkets or post offices, or paying electronically at an electronic banking point.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
dateCreated (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
date_created (xml)            time when transaction was created.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
dateUpdated (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
date_updated (xml)            time when transaction was last updated.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
dateAuthorized (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
date_authorized (xml)         time when transaction was authorized.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
status                        transaction status.
                              A list of available statuses can be found in the appendices — **Table:Transaction statuses**
message                       transaction message.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
authorizedAmount (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
authorized_amount (xml)       the transaction amount.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
capturedAmount (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
captured_amount (xml)         captured amount.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
refunded_amount (xml)         refunded amount.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
decimals                      decimal precision of transaction amount.
currency                      base currency for this transaction.
                              This three-character currency code complies with ISO 4217.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
ipAddress (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ip_address (xml)              the IP address of the customer making the purchase.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
ipCountry (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ip_country (xml)              country code associated to the customer's IP address.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
deviceId (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
device_id (xml)               unique identifier assigned to device (the customer's brower) by HiPay TPP.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
cdata1                        Custom data.
cdata2                        Custom data.
cdata3                        Custom data.
cdata4                        Custom data.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
avs_result (xml)              result of the Address Verification Service (AVS).Possible result codes can be found in the appendices
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
cvcResult (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
cvc_result (xml)              result of the CVC (Card Verification Code) check. Possible result codes can be found in the appendices
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
eci                           Electronic Commerce Indicator (ECI).
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
paymentProduct (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
payment_product (xml)         payment product used to complete the transaction.Informs about the payment_method section type.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
paymentMethod (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
payment_method (xml)          See tables below for further details.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
threeDSecure (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
three_d_secure (xml)          optional element. Result of the 3-D Secure Authentication

- enrollmentStatus (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
- enrollment_status (xml)     the enrollment status.
- enrollmentMessage (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
- enrollment_message (xml)    the enrollment status.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
fraudScreening (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
fraud_screening (xml)         Result of the fraud screening.
- scoring                     - total score assigned to the transaction (main risk indicator).
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
- result                      The overall result of risk assessment returned by the Payment Gateway.
                              Value must be a member of the following list.:
                              - pending: rules were not checked.
                              - accepted: transaction accepted.
                              - blocked: transaction rejected due to system rules.
                              - challenged:	transaction has been marked for review.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
- review                      The decision made when the overall risk result returns challenged.
                              An empty value means no review is required.
                              Value must be a member of the following list.
                              - pending: a decision to release or cancel the transaction is pending.
                              - allowed: the transaction has been released for processing.
                              - denied: the transaction has been cancelled.
----------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
Order                         information about the customer and his order.
- Id                          - unique identifier of the order as provided by Merchant.
- dateCreated (json)
- date_created (xml)          - time when order was created.
- attempts                    - indicates how many payment attempts have been made for this order.
- amount                      - the total order amount (e.g., 150.00). It should be calculated as a sum of the items purchased, plus the shipping fee (if present), plus the tax fee (if present).
- shipping                    - the order shipping fee.
- tax                         - the order tax fee
- decimals                    - decimal precision of the order amount base currency for this order
- currency                    - This three-character currency code complies with ISO 4217.
- customerId (json)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
- customer_id (xml)           - unique identifier of the customer as provided by Merchant.
- language                    - language code of the customer.
- email                       - email address of the customer.
============================  =====================================================================================================================================================================

Response fields specific to the payment product
-----------------------------------------------
Credit Card payments
  The following table lists and describes the response fields returned for transactions by credit/debit card.

=========================  =====================================================================================================================================================================
Field Name                 Description
=========================  =====================================================================================================================================================================
token                      Card token
-------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
brand                      Card brand. (e.g., VISA, MASTERCARD, AMERICANEXPRESS, MAESTRO).
pan                        Card number (up to 19 characters). Note that, due to the PCI DSS security standards, our system has to mask credit card numbers in any output (e.g., 549619******4769).
-------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
cardHolder (json)
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
card_holder (xml)          Cardholder name.
-------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
cardExpiryMonth (json)
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
card_expiry_month (xml)    Card expiry month (2 digits).
-------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
cardExpiryYear (json)
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
card_expiry_year (xml)     Card expiry year (4 digits).
-------------------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
issuer                     Card issuing bank name.
                           Do not rely on this value to remain static over time. Bank names may change over time due to acquisitions and mergers.
country                    Bank country code where card was issued.
                           This two-letter country code complies with ISO 3166-1 (alpha 2).
=========================  =====================================================================================================================================================================

QIWI payments
  The following table lists and describes the response fields returned for transactions by VISA QIWI Wallet.

=========================  =====================================================================================================================================================================
Field Name                 Description
=========================  =====================================================================================================================================================================
user                       The Qiwi user's ID, to whom the invoice is issued.
                           It is the user's phone number, in international format. Example: 79263745223
=========================  =====================================================================================================================================================================

Transaction Workflow
--------------------
Overview
  The HiPay TPP payment gateway can process transactions through many different acquirers using different payment methods and involving some anti-fraud checks. All these aspects change the transaction processing flow significantly for you.

Description
  When you send a transaction request to the gateway, you receive a response describing the transaction state.

Depending on the transaction state there are five options to action:

.. table:: Table: Transaction states

  ==================  =====================================================================================================================================================================
  Translation state   Description
  ==================  =====================================================================================================================================================================
  completed           if the transaction state is completed you are done.
                      This is the most common case for credit card transaction processing. Almost all credit card acquirers works in that way. Then you have to look into the status fied of the response to know the exact transaction status.
  forwarding          if the transaction state is forwarding you have to redirect your customer to an URL provided in the forward_url field of the response. In that case the transaction processing is not yet done. You will have to wait until the customer returned to your website after doing all redirects.
  pending             Transaction request was submitted to the acquirer but response is not yet available.
  declined            Transaction was processed and was declined by gateway.
  error               Transaction was not processed due to some reasons.
  ==================  =====================================================================================================================================================================

----------------------
Maintenance Operations
----------------------

Description
  To perform maintenance on an existing transaction, make an HTTP POST request to the following resource.
  POST /rest/v1/maintenance/transaction/{transaction_reference}

The payment gateway supports the following types of maintenance transactions.

.. table:: Table: Types of maintenance transactions

  ==================  =====================================================================================================================================================================
  Operation Type      Description
  ==================  =====================================================================================================================================================================
  capture             A request instructing the payment gateway to capture a previously-authorized transaction, i.e. transfer the funds from the customer's bank account to the merchant's bank account. This transaction is always preceded by an authorization.
  refund              A request instructing the payment gateway to refund a previously captured transaction. A captured transaction can be partly or fully refunded.
  cancel              A request instructing the payment gateway to cancel a previously-authorized transaction. Only authorized transactions can be canceled, captured transactions must be refunded.
  ==================  =====================================================================================================================================================================

URL Parameters
--------------

  =========================  =======  =======  ====  ===============================
  Parameter        	         Format   Length   Req   Description
  =========================  =======  =======  ====  ===============================
  {transaction_reference}    N                 M     The unique identifier of the transaction.
  =========================  =======  =======  ====  ===============================

Request Parameters
------------------

  =========================  =======  =======  ====  ===============================
  Parameter        	         Format   Length   Req   Description
  =========================  =======  =======  ====  ===============================
  operation
  {transaction_reference}    N                 M     The unique identifier of the transaction.
  =========================  =======  =======  ====  ===============================




.. rubric:: Footnotes

.. [1] The format of the element. Refer to "Table:Available formats of data elements” for the list of available formats.
.. [2] Specifies whether an element is required or not.
.. [ref1] Table:Issuers’ bank Id list