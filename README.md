# merchantApi
Api for Ownbit Merchant Wallet

Ownbit Merchant Wallet helps merchant accept Bitcoin & other cryptocurrencies for online payments. Integrating Ownbit Merchant Api is easy and straightforward. There are three parties: Merchant Website(your website for selling online goods), Ownbit Merchant Wallet, and Ownbit Platform. This is how the Api works:

1. You create a Ownbit Merchant Wallet (you own the wallet mnemonics, thus control the coming crypto-payments fully), search Ownbit in iOS AppStore / Google Play or download at https://ownbit.io.
2. You integrate Ownbit Merchant Api into your existing website, so that your customer can select pay for goods by Bitcoin & other cryptocurrencies.
3. The customer paid, your Ownbit Merchant Wallet received the payment directly, and you can spend the received payment immediately, since you control the Ownbit Merchant Wallet fully.
4. The Ownbit Platform call a callback_url (you configured in the Ownbit Merchant Wallet) to notify your website that a payment is received (or a payment is canceled or failed).

### Legal notes

Ownbit Merchant Api is only for selling online goods (including digitial goods). It can not be used in finance services, like Borrowing, financial management, raising funds from the public, recharging on exchanges, etc.

Merchants are also not allowed to engage in any illegal financial activities, such as money laundering and absorbing funds. Otherwise, the relevant account will be permanently disabled.

### 法律说明

Ownbit 商家Api仅用于商家销售在线商品（包括数字商品）。 它不能用于借贷、理财，从公众处筹集资金，交易所充值等金融服务。

商家也不得从事任何非法金融活动，例如洗钱和吸收资金。 否则，相关帐户将被永久禁用。

### TERMS & DEFINITION

- **apiKey**: The Api Key for your Ownbit Merchant Wallet. It can be found in your Ownbit Merchant Wallet -> Wallet Management -> Merchant Options. ApiKey is used to compute the orderHash. Always keep apiKey secret.
- **walletId**: Your Ownbit Merchant Wallet ID, you inputted when creating the Ownbit Wallet.
- **orderId**: Any string that identify an order, length: 1-64, must be unique among the system.
- **orderPrice**: Format: amount CURRENCY_SYMBOL, can be fiat or crypto, example: 9.9 USD, means 9.9 US Dollar, 0.23 BTC, means 0.23 Bitcoin. ATTENTION: There's one space between amount and symbol.
- **coinType**: Coin symbols separated by |, example: BTC|LTC|BSV|DASH, one coin only example: BTC.
- **minPaidRate**: (Optional)The minimum paid rate for an order. Can be a float value range: 0 - 1 (Default). Example: 0.95, means the minimum paid is 95% of the target amount.  
- **orderHash**: Used to authenticate the request. It is dynamically computed for each order. The computing algorithm is as follows:

When parameter **minPaidRate** is given: 

> orderHash = SHA256(walletId+":"+orderId+":"+orderPrice+":"+minPaidRate+":"+apiKey)  

When parameter **minPaidRate** is **NOT** given: 

> orderHash = SHA256(walletId+":"+orderId+":"+orderPrice+":"+apiKey) 

Examples:

> Example 1, apiKey = 11f9eaff08754dac910d744449b7a377, walletId = r89fdk3mrf1d, orderId = order12345, orderPrice = 9.9 USD and minPaidRate is not given.

> Example 2, apiKey = 11f9eaff08754dac910d744449b7a377, walletId = r89fdk3mrf1d, orderId = order12345, orderPrice = 9.9 USD and minPaidRate = 0.95.

then: 

> orderHash(Example 1) = SHA256("r89fdk3mrf1d:order12345:9.9 USD:11f9eaff08754dac910d744449b7a377"); Note that space inside orderPrice is included in computing.

> orderHash(Example 2) = SHA256("r89fdk3mrf1d:order12345:9.9 USD:0.95:11f9eaff08754dac910d744449b7a377");

Merchant Api supports the following crypto: 
- **BTC|ETH|BCH|LTC|BSV|DASH|ZEC|DOGE|DCR|DGB|USDT|DAI|USDC|TUSD|PAX**   
> Note: USDT|DAI|USDC|TUSD|PAX are ERC20 tokens.

Merchant Api supports the following fiat currency: 
> Ownbit merchant API supports all fiat currencies, examples are: USD, CNY, EUR, JPY, AUD, etc.

### Api

- **https://merchantservice.bittool.com:14443/bitbill/merchant/getCryptoByOrderId**
- **https://merchantserviceh2.bittool.com:14443/bitbill/merchant/getCryptoByOrderId**

The two nodes are operating the same way. You can choose to use one of them acorrding to the network speed you reach them. 

> Get crypto info by an order ID, method: POST, POST data is in JSON format:

```
{
  "orderHash": "5d9153dbea146fb09bc87585cbef718114cc078321bb100fbda9c0a672b44568", 
  "walletId": "r89fdk3mrf1d",
  "orderId": "order12345", 
  "orderPrice": "9.9 USD", 
  "coinType": "BTC|ETH|USDT|LTC",
  "minPaidRate": 0.95
}
```

> You should turn on the sepcific coin in your Ownbit Merchant Wallet before making the call.

Another example with fixed crypto rate (minPaidRate not given):

```
{
  "orderHash": "c55018f9017078d833124c603be839194a91a9adbf61bf0de5d800ddc8e07be0", 
  "walletId": "r89fdk3mrf1d",
  "orderId": "order12345", 
  "orderPrice": "0.12 BTC" --> ask the customer to pay 0.12 BTC regardless of the exchange rate
}
```

When the first time of this interface is called for a specific order ID, a new address for each requested coin type is allocated. Success Response:

```
{
	"data": {
		"crypto": [{
			"address": "353GjPxEY1kd14QD8XzMKRCvuv4XwhkpDK",
			"coinType": "BTC",
			"indexNo": 9,  --> The BIP32 index for generating the address, example: m/49'/0'/0'/0/9, 9 is the index
			"requestedAmount": "0.003778",  --> The amount for the specific coin the customer should pay
			"exactMatch": false
		}, {
			"address": "qzpc2q53zres6eq32mc9a3ghque3urs8xvh9mq2f9x",
			"coinType": "BCH",
			"indexNo": 6,
			"requestedAmount": "0.121193",
			"exactMatch": false
		}, {
			"address": "LUuqRpxZ43fqyxQxWxQZrxPuGLugb2h9FM",
			"coinType": "LTC",
			"indexNo": 2,
			"requestedAmount": "0.510948",
			"exactMatch": false
		}, {
			"address": "0x3d0243Bba4eF808D9d98f8A6E7567a1e772Ef861",
			"coinType": "ETH",
			"indexNo": 11,
			"requestedAmount": "0.1756324",
			"exactMatch": false
		}, {
			"address": "0x3d0243Bba4eF808D9d98f8A6E7567a1e772Ef861",
			"coinType": "USDT",
			"contractAddress": "0xdac17f958d2ee523a2206206994597c13d831ec7",
			"indexNo": 11,
			"requestedAmount": "37.950888",
			"exactMatch": false
		}],
		"payment": {
			"txHash": "ea6b0490a2e62d841677fc62cc1dd48eb987e8bc121c25ec0d4af9db116e6e9b",
			"coinType": "BTC",
			"amount": "0.003778", --> received amount 
			"paymentStatus": 1,
			"confirmations": 0,
			"rbf": false  --> whether the payment can be rbf (replaced-by-fee), only valid when confirmations is 0.
		},
		"orderId": "order-example-0009",
		"orderPrice": "270 CNY",
		"minPaidRate": 0.95,  --> Optional
		"secondsLeft": 80000 --> Seconds left for receiving a new payment
	},
	"message": "success",
	"status": 0
}
```

> The "payment" block only exists after a valid payment is received.  
> The order must be paid before **secondsLeft** becomes 0, New payments for orders that have been timed out (secondsLeft 0) will be ignored.
> The initial secondsLeft for an order is 82800 (23 hours). The allocated address for orders older than 24 hours will be revoked and reused.

**amount** rules:
- **exactMatch**: this parameter is only for **ETH/ERC20 tokens(USDT/DAI/USDC...)**. When this value is true, means you are using single address mode. In this case, the received amount must be **exactly the same** as requested. Less or greater than requested will be treated as invalid payments. Example, the requested amount is 1.234523 ETH, and the user paid 1.234524 ETH or 1.234522 ETH, will all be treated as invalid payments. When exactMatch is false, means you are using multi-address mode. In that case, the amount rule is the same as the following rules for UTXO coins.
- **For UTXO coins(BTC/BCH/LTC...)**: exactMatch for UTXO coins is always **false**. The received amount should be **equal or greater than** requested **plus** **minPaidRate**. Example, the request is 0.123456 BTC and minPaidRate equals 1, the payment is 0.123455 BTC, it will be treated as invalid, no payment info will be returned, and no notification will be sent. If payment is 0.123456 BTC or 0.123457 BTC, then it will be treated as valid. When minPaidRate equals 0.98, the target amount will be: 0.123456 * 0.98 = 0.12098688 BTC, a payment of 0.123455 BTC will also be treated as valid.

**ETH/ERC20 Address mode**:
For ETH and ERC20 tokens, your merchant wallet can have two type of different modes: **Single address mode** and **Multiple address mode**. Differences between them are:  
- Multiple address mode uses different ETH address for each order request, while Single address mode uses the same ETH address for order requests.
- Multiple address mode can support up to 100k orders per day (totally 100k addresses can be generated), while Single address mode supports less. Single address mode uses different amount for each order equest, and by matching the amount exactly to distinguish different payments.
- Single address mode needs the user to pay **EXACTLY** the amount it requests. Pay slightly more or less is not valid.
- For Multiple address mode, balances in other addresses will not be counted in the main address by default. That's to say the balance you see in your Ownbit wallet is less than the total balances you have. The Ownbit merchant wallet is now providing a tool for you to collect those balances into your ETH main address. Find the tool in your wallet's ETH coin page (a link named: Manage Balances on top of the page).
- Merchant wallets are default to use Multiple address mode. If you want to use the Single address mode, create a new merchant wallet, and append "_single" in your wallet ID. Example wallet ID for single mode: **myshop_single**.   

**paymentStatus** can have the following value:
- **0**: Initial status, no payment (or invalid payments);
- **1**: A valid payment is received, status: unconfirmed;
- **2**: A valid payment is received, status: confirmed;
- **9**: The payment transaction is canceled or failed;

> Note: The merchant should handle paymentStatus 9 in a proper manner. paymentStatus 9 can happen even after a transaction is confirmed (in case of blockchain rollback).

If something is wrong for processing, or you passed the wrong parameters for calling the interface, the Api will return the corresponding error code:

```
{
  "message": <error message>,
  "status": <error code>
}
```

A list of status codes are:
- **0**: success
- **-1**: Insufficient fee
- **-31**: Lack of madatory parameters
- **-41**: Wallet not exists
- **-61**: Invalid Order Price
- **-62**: Unsupported Fiat Currency
- **-63**: Unsupported Coin Type
- **-64**: Coins not open in the corresponding Merchant Wallet
- **-65**: Order ID and Order Price not match (Fetch orderId again with non-empty orderPrice, but with a different value than the first fetch)
- **-66**: Can't generate more addresses
- **-67**: Merchant Account not exists
- **-68**: orderHash incorrect
- **-202**: Invalid exchange rate for coins (Server side error)

### Fee

The Ownbit Platform charges **1% - 0.1%** of transaction volume as the processing fee according to how much volume the merchant has processed in total. And the fee must be deposited into your Ownbit Merchant Wallet before hand. If the current fee is insufficient, the Api will return the following error:

```
{
  "message": "Insufficient fee",
  "status": -1
}
```

Ownbit charges merchants small fees (1% - 0.1%). Fee rate for different volume range :
- **1%**: Total processed volume less than 1,000 USD
- **0.9%**: Total processed volume between 1,000 USD - 10,000 USD
- **0.8%**: Total processed volume between 10,000 USD - 100,000 USD
- **0.7%**: Total processed volume between 100,000 USD - 500,000 USD
- **0.6%**: Total processed volume between 500,000 USD - 1,000,000 USD
- **0.5%**: Total processed volume between 1,000,000 USD - 5,000,000 USD
- **0.4%**: Total processed volume between 5,000,000 USD - 10,000,000 USD
- **0.3%**: Total processed volume between 10,000,000 USD - 50,000,000 USD
- **0.2%**: Total processed volume between 50,000,000 USD - 100,000,000 USD
- **0.1%**: Total processed volume > 100,000,000 USD


### Callback

The Ownbit Platform will call the merchant's callback_url to notify the merchant that a payment is received or a payment state is changed. callback_url must be a POST interface. POST data is passed as the following:

**Note: Deposit enough fee in your Ownbit Merchant Wallet. The Ownbit Platform will stop calling the notification as long as the fee balance becomes less than 0.**

**Note: When you deposite enough fee in your Ownbit Merchant Wallet, the missed notification will be resent to callback_url automatically.**

**Note: Set callback_url in your Ownbit Merchant Wallet -> Wallet Management -> Merchant Options.**

```
{
  "orderId":"order12345", 
  "orderPrice":"1000 USD",
  "payment": {
      "txHash": "ea6b0490a2e62d841677fc62cc1dd48eb987e8bc121c25ec0d4af9db116e6e9b",
      "coinType": "BTC",
      "amount": "0.123765", --> received amount 
      "paymentStatus": 2,
      "confirmations": 1,
      "rbf": false
   },
   "callbackHash": "fc1e249e4595220f18c8298e1e5d261e352b4382a1d33c3ccc8342e94fd94be5"
}
```

**callbackHash** is used to verify that the call is really originated from the Ownbit Platform. The merchant side should recompute the callbackHash, and compare with the one received in the request, to make sure the call is from Ownbit. The computing algorithm is as follows:

> callbackHash = SHA256(walletId+":"+orderId+":"+orderPrice+":"+txHash+":"+coinType+":"+amount+":"+paymentStatus+":"+confirmations+":"+rbf+":"+apiKey)  
> In above example, if apiKey = 11f9eaff08754dac910d744449b7a377, walletId = r89fdk3mrf1d

then: 

> callbackHash = SHA256("r89fdk3mrf1d:order12345:1000 USD:ea6b0490a2e62d841677fc62cc1dd48eb987e8bc121c25ec0d4af9db116e6e9b:BTC:0.123765:2:1:0:11f9eaff08754dac910d744449b7a377"); Note that space inside orderPrice is included in computing.  
> If **rbf** is true, use **1** for computing, otherwise use **0** for computing.   
> Tip: Beware of the space inside orderPrice (It's '1000 USD' in the example).

The Ownbit Platform expects a plain string: "SUCCESS" or a JSON string contains "SUCCESS" as the response. If the response is not SUCCESS, the platform will continuously to call the url in specific time gaps (30 secs, 1 min, 2 mins, 5 mins, 30 mins, 2 hours, 6 hours, 1 day and 2 days), until it fails after 10 times. Example response:

```
SUCCESS
```

or 

```
{"status":"SUCCESS"}
```

**Situations the callback is triggered** 
- **A: A new payment is received;**
- **B: The unconfirmed payment becomes confirmed;**
- **C: A payment is canceled or failed;**

The merchant might get multiple notifications for a payment. Possible notification cases:

> CASE 1: A -> B (First get notification A, then get B, the pyament comes to unconfirmed first, and then confirmed)  
> CASE 2: B (the payment goes to confirmed directly, no unconfirmed state)  
> CASE 3: A -> C (the payment goes to unconfirmed, and then canceled)    
> CASE 4: B -> C (the payment goes to confirmed, and then canceled or failed, this sutiation can happen in case of block rollback.)    

### Trust Unconfirmed or Not? 

Sometime payments may take dozens of minute to confirm, should the merchant trust unconfirmed payments or not? If not trust unconfirmed payments, the customer may need to wait for long time to get the goods. It's not friendly in some cases especially for digital contents. But if we trust all unconfirmed payments, attackers can make use of this vulnerability, open a new order, get the goods, and then cancel or make the transaction invalid.

To get around of this problem, Ownbit suggests a general rule for merchants to follow:
- **For account based coins, like: ETH/ERC20 tokens(USDT/DAI/USDC...)**, always trust **confirmed** payments only, ship your digital contents to your cusomter only after payment transaction get confirmed.
- **For UTXO based coins, like: BTC/BCH/LTC...**, merchants can trust **unconfirmed** payments when **rbf** is false. The merchant can ship the digital contents immediately if rbf is false. 

> - Unconfirmed payments with **rbf** equals to true, can be canceled in technical very easily. If merchants trust such payments, they should have a mechanism to get their goods back if the payments get canceled.  
> - It's very difficult and very rare for unconfirmed payments get canceled if the rbf is false. But it's not to say impossible in theory. 
> - Merchants should get well prepared for handle notification of paymentStatus 9, to deal with payments cancelation.

### Integrate Ownbit Pay page (Optional)

If you won't integrate /getCryptoByOrderId Api in low level, you can use Ownbit Pay Page directly. For each order, you open an URL like below:

https://ownbit.io/pay/?orderId=order-online-example021&orderPrice=1.5%20CNY&minPaidRate=0.98&walletId=rufjlwgw839y&orderHash=c3874b2026331480aa03aad691e0c0da080e1201c7ac7dab064c60e79d2d79eb&coinType=BTC%7CETH%7CUSDT%7CBCH%7CLTC%7CDASH&orderSubject=Buy%20Pizza%20Online&orderDescription=A%20pizza%208-10%20inches%20with%206%20slices.&lang=en

The Ownbit Pay page looks like this:

![image](https://bitbill.oss-accelerate.aliyuncs.com/pics/ownbit-pay-example.jpeg)

**Parameters for Pay page:**
- **orderId**: same as /getCryptoByOrderId Api.
- **orderPrice**: same as /getCryptoByOrderId Api.
- **walletId**: same as /getCryptoByOrderId Api.
- **orderHash**: same as /getCryptoByOrderId Api.
- **coinType**: same as /getCryptoByOrderId Api (You can change the sequence of the coin types. For example, 'BTC|ETH|USDT' will make BTC as the default payment coin, while 'ETH|BTC|USDT' will make the ETH as the default payment coin.)
- **minPaidRate**: (Optional)same as /getCryptoByOrderId Api.
- **orderSubject**: The Subject for the order.
- **orderDescription**: The description text for the order.
- **redirectUrl**: (Optional)The url to redirect after payment success. Example: https://mywebsite.com/paysuccess?orderId=order-online-example021 (Note that the entire content should be URLEncoded.)
- **appCallback**: (Optional)Callback to the merchant's app after payment success. Value can be: **iOS** or **Android**.
- **lang**: the page language, allowed values are:
   - **en**: English, default
   - **cn**: Simplified Chinese
   - **tra**: Traditional Chinese
   - **jp**: Japanese
   - **kr**: Korean
   - **ru**: Russian

**Note that all parameters should be URLEncoded. Example, an URLEncoded value of "1.3 CNY" is "1.3%20CNY", while an URLEncoded value of "https://mywebsite.com/paysuccess?orderId=order-online-example021" is "https%3A%2F%2Fmywebsite.com%2Fpaysuccess%3ForderId%3Dorder-online-example021"**.

JS code example for URLEncode:

```
// Encode a URI
var param = "A pizza 8-10 inches with 6 slices";
var encodedParam = encodeURI(param);
// OUTPUT: A%20pizza%208-10%20inches%20with%206%20slices
```

**appCallback**
Value can be **iOS** or **Android** (Case insensitive), When value **iOS** is given, the Ownbit Pay page will try to post a message to the merchant's App when the payment becomes success (assume the merchant's App is using WKWebView).

```
// js code to call swiftSuccessCallback
window.webkit.messageHandlers.obMessage.postMessage(orderId); //js tells the iOS app that this orderId has just received a successful payment.
```

In your swift code, you can register the callback similar as follows:

```
//register js callback to swift
webView.configuration.userContentController.add(self, name: "obMessage") //for js callback

func userContentController(_ userContentController: WKUserContentController,
                               didReceive message: WKScriptMessage) {
        switch message.name {
        case "obMessage":
            let result = (message.body as? String) ?? ""
	    //result is the orderId
	    //do something in your app
	default: break
        }
}
```

When value **Android** is given, the Ownbit Pay page will try to call **android.jsSuccessHandler** of the merchant's App.

```
// js code to call android.jsSuccessHandler
android.jsSuccessHandler("ownbitpay", orderId); //js tells the Android app that this orderId has just received a successful payment.
```

In your Android code, you can register the callback similar as follows:

```
//create WebView
WebView mWebView = new WebView(context);
mWebView.addJavascriptInterface(this, "android"); //this is to register the javascript interface
WebSettings webSettings = mWebView.getSettings();
webSettings.setJavaScriptEnabled(true);
webSettings.setAllowUniversalAccessFromFileURLs(true);
...

//add JavascriptInterface
@JavascriptInterface
public void jsSuccessHandler(String func, String result){
	if (func.equals("ownbitpay")) {
		//do something, result contains the orderId string
	}
}
```

### Java code 

**Java code for integrating /getCryptoByOrderId Api and orderHash computing is in the java directly.**

### Miscellaneous

**Max orders within 1 day**: The maximum new orders a merchant can open within 1 day is **100000**. If you want more, please contact us through email: **hi@bitbill.com**.  

### Technical Support

If you have any problem integrating with Ownbit merchant Api, please feel free to contact us:  
**By Email**: hi@bitbill.com. 
**By Telegram**: https://t.me/ownbit
