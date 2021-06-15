# Midas Payment Product Capabilities and Application Scenarios



## I. Payment process

Midas Payment provides two payment methods with different payment processes to cater to the needs of various payment scenarios.

1. In-game currency mode (recommended): Virtual currency accounts of merchants are hosted on the Midas Payment server. After successful payment, the Midas Payment backend calls account system APIs, tops up virtual token for the user, and completes token purchase. Merchants can use the accounts with query, consume, and rollback APIs provided by Midas Payment.This method can reduce development costs. Midas Payment also provides a series of marketing, risk control, and reconciliation features based on the account, which helps merchants go online quickly after the access, as shown below:

![img](markdown/MidasPaymentProduct/16068036689286.png)

1. Prop mode: Merchants' accounts are not hosted, like props and gifts.Merchants complete payment through APIs provided by Midas Payment. Midas Payment informs merchants to deliver goods through server APIs.This is a complete process as well. But compared to the in-game currency mode, merchants can't use Midas Payment's value added services such as marketing and risk control capabilities highly dependent on accounts, as shown below:

![img](markdown/MidasPaymentProduct/16068036787273.png)



1. Subscription mode:

1. Non-automatic renewal:

Non-automatic renewal allows users to enjoy your app content, services, or advanced features in a set period of time. For every subscription, users only pay for a set period of time like a month, a week, or a year. When the time expires, users will not be charged automatically and need to subscribe again to access the services.

Details of the process is as follows:

 ![image-20201123193057820](markdown/MidasPaymentProduct/16068037033550.png)

2. Automatic renewal:

Automatic renewal allows users to enjoy the content, services, and advanced features without interruption.Automatic renewal will charge users and renew the subscription automatically when the time expires unless users cancel the subscription.

The subscription mode is available to various business, such as adding new game levels, serialized content, SaaS, cloud support, businesses that provide continued substantial updates, and businesses that provides access permissions to content libraries or content collections.

Details of the process is as follows:

![image-20201123193111516](markdown/MidasPaymentProduct/16068037207744.png)

3. Discounts for automatic renewal

The promotional events are designed to attract more subscribed users and let newly subscribed users experience the value of subscription features before paying the full price to encourage subscriptions.Giving customers discounts and letting them purchase automatically renewed subscription at a discounted price or enjoy it for free in a set period of time can attract more customers and improve retention.During the promotional event period, subscriptions will be renewed automatically at the standard price unless users cancel subscriptions or disable automatic renewal. Different channels support different subscription discounts. Please contact Midas team if you require this feature.



## II. Promotional events

1. Examples of promotional event:

First purchase event: Purchase more than 100 Diamonds in the game for the first time and get an item worth 111 diamonds. Conditional giveaways: Get an item worth 10 diamonds every time you purchase more than 100 Diamonds during the event. Get an item worth 20 diamonds every time you purchase more than 200 Diamonds during the event. Get an item worth 30 diamonds every time you purchase more than 300 Diamonds during the event. Limited giveaways: Purchase more than 100 Diamonds for the first time during the event and get an item worth 100 diamonds. Purchase more than 200 Diamonds for the first time during the event and get an item worth 200 diamonds. Purchase more than 300 Diamonds for the first time during the event and get an item worth 300 diamonds.

1. Possible event types: First purchase, accumulative giveaways, discount, free gifts, monthly pass subscription.
2. Event access process: 1>Configure an event. 2>Call SDKs to pull and parse event information. 3>Show event information to merchants. 4>Import ID of goods when purchasing. Payment successful. 5>Midas Payment backend releases regular and event goods.



## III. Risk control engine

1. Basic risk control:

When a user purchases a product, Midas Payment will evaluate if a user is a regular user with its risk control model. If someone is a malicious user, Midas Payment will take measures using several strategies.Midas Payment's risk control measure includes several strategies, such as intercepting orders, intercepting delivery, allowing delivery but banning account, and banning delivery but not account.For suspected malicious users, interception measures could be too extreme. There are also flexible risk control measures that allow users to continue with payment after certain verification steps. This is a medium-level risk control.

1. Joint risk control measures:

When Midas Payment discovers a suspicious user, it will return an error code to the merchant during payment process. After receiving the error code, the merchant asks the user to complete certain restrictive tasks. When the tasks are completed, the backend reports to Midas Payment to unblock the user, and then the user would be able to continue with the purchase.



## Quick Start

Download demo

1. Quick integration of SDK.
2. Call initialization interface.
3. Call payment interface.
4. Callback processing for successful payment.



## FAQs

1. Can I use the same account for Android and iOS systems? A: Yes. Midas Payment allows you to use one or multiple accounts on different platforms.
2. How does Midas Payment display multiple currencies? A: The language, currency, and amount of an app can differ in various countries. It is recommended to call Midas Payment SDKs to acquire multi-currency information APIs and display localized sales information on the product page.