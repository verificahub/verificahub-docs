---
title: Public offer
sidebar_label: Public offer
sidebar_position: 1
---

# Public offer

> **Draft — pending legal review.** This document is a draft for review and is not yet in effect.

For the provision of Verificahub services.

## 1. General provisions

1.1. This Public Offer (the “Offer”) sets out the terms on which **Aigiz Iskuzhin, sole proprietor** (the “Provider”), offers Verificahub services.

1.2. A person accepting the Offer by registering, using the Service or paying for Services becomes the Customer and accepts its terms in full.

1.3. The Provider may amend the Offer. The current version is published on the Provider's Website with its effective date. Changes take effect 10 calendar days after publication unless the revised version specifies another period. Continued use after the changes take effect constitutes acceptance. A Customer who disagrees may stop using the Services and request a refund of their unused balance.

## 2. Definitions

2.1. **Provider:** Aigiz Iskuzhin, sole proprietor, providing Verificahub services.

2.2. **Customer:** an individual or legal entity that accepts the Offer.

2.3. **Service / Services:** access to the Verificahub user verification platform, including SMS, calls and verification through third-party services, through the API and dashboard at the applicable prices.

2.4. **Provider's Website:** https://verificahub.ru. The dashboard is available at https://app.verificahub.ru.

## 3. Services

3.1. The Provider gives the Customer access to the Service to verify users' phone numbers. The Customer pays for the Services as they are provided.

3.2. Available verification methods and their prices are specified in the current pricing and documentation on the Provider's Website. Changes to those documents do not require an amendment to this Offer. Methods available to the Customer are shown in the dashboard and in the `GET /v1/prices` API response.

3.3. The Provider may restrict or suspend a verification method, including for an individual Customer, if:

- The channel does not have enough capacity to handle the Customer's request volume with acceptable delivery quality.
- The volume or nature of requests threatens the operation of the Service or the quality of Services provided to other Customers.
- The telecommunications operator or another supplier providing the method requires it.

Other methods remain available. The Provider notifies the Customer through the dashboard. The method is also marked as unavailable in `GET /v1/prices`, and verification requests using it are rejected with `method_not_available`. If there is an immediate threat to the operation of the Service, the restriction takes effect immediately, followed by notification.

A restriction under this clause does not constitute a breach of the Offer and ends when the circumstances causing it are resolved.

## 4. Pricing and payments

4.1. Services are charged on a pay-as-you-go basis at the current prices on the Provider's Website. The charge event depends on the method:

- **Reverse flash-call:** successful verification.
- **Code or message delivery (SMS, Telegram):** sending the code or message.
- **Verification through a telecommunications operator (such as MTS ID):** each initiated verification attempt, including fallback SMS OTP delivery, whether or not verification succeeds.

4.2. An unavailable method is not provided or charged: the verification request is rejected before funds are deducted. The unused balance is refunded at the Customer's request within 10 business days.

## 5. Personal data

5.1. The Provider acts as the controller of the Customer's personal data and processes it in accordance with the [Privacy policy](./privacy_policy.md).

5.2. The Customer is the controller of end users' phone numbers submitted for verification. The Provider processes those numbers on the Customer's instructions. The Customer is responsible for having a lawful basis to submit the data and request verification.

## 6. Liability

6.1. The Service is provided “as is.” The Provider is not responsible for consequences arising from the Customer's use of the Service in breach of applicable law or this Offer.

6.2. The Provider does not guarantee uninterrupted availability of any individual verification method and is not responsible for losses caused by a restriction or suspension under clause 3.3.

## 7. Provider and contact details

Aigiz Iskuzhin, sole proprietor · privacy@verificahub.ru · +7 927 938-35-62
