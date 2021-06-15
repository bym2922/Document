# About Midas Payment



## I. Introduction

Midas Payment is a global payment services platform that has provided professional billing solutions for more than 4,000 merchants across the world since 2012.It provides merchants with an all-in-one approach to business development in terms of payment, marketing configuration, transaction processing, risk control, reconciliation, and settlement.Midas Payment is compatible with both official channels such as Apple and Google,and game platforms like Steam, Switch, and Samsung.It offers a full range of payment services that support official website access, client SDKs, and server APIs.It also offers unified payment interfaces by combining channel capabilities to make it easy for merchants to complete integration.

## II. Overall architecture

Midas Payment has an architecture that consists of the following parts: Official website access: Merchants can connect their businesses to Midas Payment and manage them on the official website of Midas through Midas Payment console, and publish apps in sandbox and live version environments for integration testing and verification. Client SDK: Midas Payment supports various systems such as Android, iOS, Unity, and Unreal by providing SDKs for each environment and integration types for each merchant. Server API: Server is the core of Midas Payment system. It includes systems such as inventory management, transaction engine, risk control, and channel service. 
![img](markdown/AboutMidasPayment/16067441751976.png)


## III. Calling relations

Midas Payment is compatible with official and host channels and global mainstream third-party channels. Every channel has different protocols, eliminating the differences between Midas Payment's SDKs and server APIs. Merchants just need to call the SDK and server API provided by Midas Payment. 
 ![img](markdown/AboutMidasPayment/16067441821090.png)
 
## IV. Platform support

Midas Payment supports access to Android, iOS, Unity, Unreal, Cocos, and console games and offers C# and C++ APIs.

| **Platform** | **API type** | **Note** |
| ------------ | ------------ | -------- |
| Android      | java         |          |
| iOS          | oc           |          |
| Unity3D      | C#           |          |
| Unreal       | C++          |          |
| Cocos        | C++          |          |
| Steam        | C++          |          |
| PS4          | C++          |          |
| PS5          | C++          |          |
| Xbox One     | C++          |          |

## V. Channel support

| **Channel name** | **Support** | **Note** |
| ---------------- | ----------- | -------- |
| Google Play      | Yes         |          |
| IAP              | Yes         |          |
| Steam            | Yes         |          |
| Switch           | Yes         |          |
| Xbox             | Yes         |          |
| PS4              | Yes         |          |
| PS5              | Yes         |          |
| Samsung          | Yes         |          |

## VI. Basic terms

- In-game currency mode: Accounts are hosted on Midas Payment server. Merchants can manage accounts by calling the server API provided by Midas Payment, instead of managing them directly.
- Prop mode: Merchants manage accounts directly for in-game currencies or props. Midas Payment only takes care of the process of payment transaction and informs merchant's servers to release goods after successful payment.
- Subscription mode: Products in an app that charge users at regular intervals (through the payment method provided by the users).
- offerid: ID used to identify an app. Assigned by Midas Payment for every connected app.

## VII. Data security and privacy

1. Midas Payment ensures security of user data from design and deployment aspects.We can choose shared cloud or independent and private deployment to prevent data from being leaked. 2. Midas Payment does not collect private user information without permission. We outline all the permissions and details of user information we require in relevant collaborative documents.Our products also support different privacy terms required by channels, such as GDPR.



## VIII. Merchant access steps

1. Apply for offerid on the Midas Payment website and complete relevant configurations such as information of goods, channel selection, and server partition.
2. Complete relevant configurations on third-party payment platforms, such as Apple, Google, and Switch. Refer to channel configuration.
3. Complete SDK integration (different versions for different platforms).
4. Access server protocol.
5. Publish in sandbox environment and do integration testing.
6. Publish in live version environment and do integration testing.
7. Go online.

