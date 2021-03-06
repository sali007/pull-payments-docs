# Responses

Response may be in XML or JSON format, depending on "Accept" header in request.

## Successful Invoice Operations {#response_bill}

Object with operation's result.

<aside class="notice">
When you send two consecutive requests with the same <i>{prv_id}</i>, <i>{bill_id}</i> and <i>amount</i> parameters, you will get the same result code.
</aside>
	 
~~~xml
<response>
   <result_code>0</result_code>
   <bill>
    <bill_id>bill1234</bill_id>
    <amount>99.95</amount>
    <originAmount>99.95</originAmount>
    <ccy>RUB</ccy>
    <originCcy>RUB</originCcy>
    <ccy>USD</ccy>
    <status>paid<status>
    <error>0</error>
    <user>tel:+79161231212</user>
    <comment>Invoice from ShopName</comment>
   </bill>
</response>
~~~

~~~json
{
 "response": {
  "result_code": 0,
  "bill": {
    "bill_id": "BILL-1",
    "amount": "10.00",
    "originAmount": "10.00", 
    "ccy": "RUB",
    "originCcy": "RUB",
    "status": "waiting",
    "error": 0,
    "user": "tel:+79031234567",
    "comment": "test"
  }
}}
~~~

Parameter|Type|Description
--------|---|--------
result_code|Integer|Only "0", means successful operation
bill_id|String|Unique invoice identifier generated by the merchant
amount|String|The invoice amount. The rounding up method depends on the invoice currency.
originAmount|String|The invoice amount in the original invoice's currency (see `originCcy` parameter). The rounding up method depends on the invoice currency.
ccy	|String|Currency identifier (Alpha-3 ISO 4217 code)
originCcy|String|Currency identifier of the invoice (Alpha-3 ISO 4217 code)
status	|String|Current [invoice status](#status)
error	|Integer|[Error code](#errors)
user|String|The Visa QIWI Wallet user’s ID, to whom the invoice is issued. It is the user’s phone number with "tel:" prefix.
comment|String|Comment to the invoice

## Successful Refund Operations {#response_refund}

Object with operation's result.
	 
~~~xml
<response>
<result_code>0</result_code>
<refund>
<refund_id>122swbill</refund_id>
<amount>10.0</amount>
<status>processing<status>
<error>0</error>
<user>tel:+79161231212</user>
</refund>
</response>
~~~

~~~json
{
 "response": {
  "result_code": 0,
  "refund": {
    "refund_id": 122swbill,
    "amount": "10.0",
    "status": "processing",
    "error": 0,
    "user": "tel:+79161231212"
  }
}}
~~~

Parameter|Type|Description
--------|---|--------
result_code|Integer|Only "0", means successful operation
refund_id|String|The refund identifier, unique number in a series of refunds processed for a particular invoice
amount|String|The actual amount of the refund. The rounding up method depends on the original invoice currency.
status	|String|Current [refund status](#status_refund)
error	|Integer|[Error code](#errors). **Important!** When the amount of refund exceeds the initial invoice amount or the amount left after the previous refunds, error code 242 is returned.
user|String|The Visa QIWI Wallet user’s ID, to whom the original invoice is issued. It is the user’s phone number with "tel:" prefix.

## Response in Case of Operation's Error {#response_error}

There will be no invoice/refund data in the response, when corresponding operation is unsuccessful.

~~~xml
<response>
<result_code>150</result_code>
<description>Authorization failed</description>
</response>
~~~

~~~json
{
 "response": {
  "result_code": 150,
  "description": "Authorization failed"
  }
}
~~~

Parameter|Type|Description
--------|---|--------
result_code|Integer|[Result code of the operation](#errors)
description|String|Error description
