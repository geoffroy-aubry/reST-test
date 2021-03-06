.. _Chap4-3DSecureIntegration:

.. _3-D Secure:

======================
3-D Secure Integration
======================

------------
Introduction
------------

Overview
  This chapter describes how you should implement :term:`3-D Secure` Authentication using the Remote REST API model.

Description
  This process involves redirecting the shopper to an authentication page.
  This page is provided and hosted by the shopper's Card Issuer.
  As this page is hosted by the shopper's card issuing bank, we have no control over its appearance or functionality.

----------------
About 3-D Secure
----------------

3-D Secure History
  In early 2001, VISA introduced a security protocol called :term:`3-D Secure` to improve online transaction performance and
  to accelerate the growth of electronic commerce through increased consumer confidence.

Objective of 3-D secure
  The objective of 3-D Secure was to provide Issuers with the ability to actually authenticate cardholders during
  an online purchase, 3-D Secure to reduce the likelihood of fraudulent usage of payment cards and to improve transaction performance to benefit merchants, consumers and acquirers.
  VISA's branded 3-D Secure Program is commonly known as Verified By VISA (VbV).
  Services based on the protocol have been also been adopted by MasterCard, under the name MasterCard SecureCode TM(MSC).

-----------------
Merchant Benefits
-----------------

3-D Secure Benefits
  The benefits of the :term:`3-D Secure` Process are the enhanced security available when performing an authenticated transaction
  as well as the shift of liability in the event of fraudulent transactions. Authentication should strengthen your existing anti-fraud strategy and
  help protect your business, but bear in mind that coverage of authentication programs is currently limited to Internet transactions.

Restriction
  This means that authentication programs do not cover fax, mail, or phone orders (MO/TO), nor do they cover all card types.
  The additional security benefits and liability shifts of authenticated transactions are currently only supported by Visa and MasterCard.

----------------
Transaction Flow
----------------

Workflow overview
-----------------

.. figure:: images/transactionFlow.png
    :align: center
    :alt: Workflow overview

    Figure: Workflow overview

Procedure
---------

Proceed as follow to carry out a transaction:

.. table::
  :class: table-with-wrap

  ======  ======================================================================================================================================================================================================================================================================================================================
  Step    Action
  ======  ======================================================================================================================================================================================================================================================================================================================
  **1**	  Merchant call HiPay API with "authentication_indicator=1" value. - Refer to parameters sent to API method used.
  ------  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **2**	  To complete the purchase; the cardholder press the Pay button after filling payment card details in payment page:

          - This activates the Merchant Plug-In (MPI) and initiates a transaction.
  ------  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **3**	  The MPI identifies the card number and sends it to the Directory Server to determine whether the card is in a participating card range.
  **4**   If the Issuer is participating for the card range, the Directory sends a Verify Enrollment Request message to the Issuer ACS to determine whether authentication is available for the account number.
  **5**   The ACS returns a Verify Enrollment Response to the Directory Server

          - **IF** Authentication is available for this card number **THEN** the response provides the URL of the ACS where the cardholder can be authenticated.
          - **IF** Authentication is not available **THEN** the Merchant server receives a Cardholder Not Enrolled or Authentication Not Available message and returns the transaction to the Merchant's commerce server to proceed with a standard transaction processing.
  ------  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **6**   The Directory Server forwards the ACS response to the MPI.
  **7**   The MPI sends an Authentication Request message to the cardholder's browser for routing to the ACS.
  **8**   The cardholder's browser passes the Authentication Request to the ACS.
  **9**   The ACS authenticates the cardholder.
  **10**  The ACS creates, digitally signs, and sends an Authentication Response to the Merchant via the cardholder's browser. The ACS also sends transaction record to the Authentication History Server for storage.
  **11**  The browser routes the Authentication Response back to the MPI.
  **12**  The MPI validates the digital signature in the response, verifying that it is from a valid participating Issuer.
  **13**  The Merchant formats and sends to its Acquirer an Authorization Request message, which includes information from the Issuer's Authentication Response - including the CAVV and the ECI. The Acquirer passes the Authorization Request to the Card Network and the transaction completes through standard processing.
  ======  ======================================================================================================================================================================================================================================================================================================================

----------------------
Authentication Results
----------------------

The following table lists the Enrollment message and status:

.. table:: Table: Enrollment Message and Status
  :class: table-with-wrap

  =======  =========================  =====================  ============  ===================================================================================================================================================================================================
  Status   Enrollment Message         3-D Secure Available?  :term:`ECI`   Description
  =======  =========================  =====================  ============  ===================================================================================================================================================================================================
  Y        Authentication Available   Yes                                  Card is enrolled in the :term:`3-D Secure` program and the payer is eligible for authentication processing.
  N        Cardholder Not Enrolled    No                     6             Card is not enrolled in 3-D Secure program.

                                                                           Card is eligible for authentication processing (it is within the card associations range of accepted cards) but the card-issuing bank does not participate in the 3-D Secure program.

                                                                           **Chargeback Liability Shift**: If the cardholder later disputes the purchase, the issuer may not submit a chargeback to the merchant.
  U        Unable to Authenticate     No                     7             The card associations were unable to verify if the cardholder is enrolled in the :term:`3-D Secure` program.

                                                                           Merchants can choose to accept the card nonetheless and proceed the purchase as non-authenticated when submitting the authorization.

                                                                           **Chargeback Liability Shift**: The Acquirer/Merchant retains liability if the cardholder later disputes making the purchase.
  E        *Any error message here*   No                     7             An error occurred during the enrollment verification process.

                                                                           **Chargeback Liability Shift**: The card can be accepted for authorization processing, yet the merchant may not claim a liability shift on this transaction in case of a dispute with the cardholder.
  =======  =========================  =====================  ============  ===================================================================================================================================================================================================

The following table lists the Enrollment message and status:

.. table:: Table: Authentication Message and Status
  :class: table-with-wrap

  =======  ======================================  ============  ============================================================================================================================================
  Status   Authentication Message                  :term:`ECI`   Description
  =======  ======================================  ============  ============================================================================================================================================
  Y        Authentication Successful               5             Cardholder was successfully authenticated. The Issuer has authenticated the cardholder by verifying the identity information or password.
  A        Authentication Attempted                6             Authentication could not be performed but a proof of authentication attempt was provided.
  U        Authentication Could Not Be Performed   7             The Issuer is not able to complete the authentication request due to a technical error or other problem.

                                                                 Possible reasons include:

                                                                 - Invalid type of card such as a Commercial Card or any anonymous Prepaid Card.
                                                                 - Unable to establish an :term:`SSL` session with cardholder browser.
  N        Authentication Failed                                 The cardholder did not complete authentication and the card should not be accepted for payment.

                                                                 The following are reasons to fail an authentication:

                                                                 - Cardholder fails to correctly enter the authentication information
                                                                 - Cardholder cancels the authentication process.

                                                                 An authentication failure may be a possible indication of a fraudulent user.
                                                                 :term:`Authorization` **request should not be submitted**
  E        *Any error message here*                              An error occurred during the authentication process.

                                                                 **Authorization request should not be submitted.**
  =======  ======================================  ============  ============================================================================================================================================
