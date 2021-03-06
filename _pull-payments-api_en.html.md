# Pull REST API {#pull_rest_api}

###### Last update: 2017-07-11 | [Edit on GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_pull-payments-api_en.html.md)

## Invoicing Operation Flow

![Pull API Invoicing](images/pullrest_1_en.png)

1. User submits an order on the merchant’s website. 
2. Merchant sends [Create invoice](#invoice) request to Visa QIWI Wallet server with authorization parameters.
3. Merchant is recommended to redirect to [QIWI Checkout](#checkout) page on Visa QIWI Wallet site when the request is completed. Otherwise, invoice can be paid in any Visa QIWI Wallet interfaces, such as web (qiwi.com), mobile applications and self-service terminals.
4. If merchant enables [notifications](#notification), then Visa QIWI Wallet sends to the merchant's server a notification on the invoice status once invoice is paid or cancelled by the user. Authorization on the merchant's side is required for notifications.
5. Merchant can [request current status](#invoice-status) of the created invoice, or [cancel invoice](#cancel) (provided that it has not been paid yet) at any moment.
6. Merchant delivers ordered services/goods when the invoice gets paid.

## Authorization {#auth_rest_api}

Pull REST API requests are authorized through HTTP Basic-authorization with [API ID and API password](#auth_param).


~~~shell
user@server:~$ curl "server_URL"
  --header "Authorization: Basic MjMyNDQxMjM6NDUzRmRnZDQ0Mw=="
~~~

<aside class="notice">
Header is <i>Authorization</i> string and its value is <i>Basic Base64(API_ID:API_PASSWORD)</i>
</aside>


## Issuing Invoice for the Order {#invoice}

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/test234578"
  -X PUT 
  -d 'user=tel%3A%2B79161111111&amount=1.00&ccy=RUB&comment=uud_TEST7&lifetime=2016-09-25T15:00:00'
  --header "Accept: text/json" --header "Authorization: Basic ***"  
~~~

~~~http
PUT /api/v2/prv/2042/bills/BILL-1 HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8

user=tel%3A%2B79031234567%26amount=10.0%26ccy=RUB%26comment=test%26lifetime=2012-11-25T09%3A00%3A00
~~~

~~~php
<?php
//Shop identifier from Merchant details page 
//https://ishop.qiwi.com/options/http.action
$SHOP_ID = "21379721";
//API ID from Merchant details page 
//https://ishop.qiwi.com/options/rest.action
$REST_ID = "62573819";
//API password from Merchant details page 
//https://ishop.qiwi.com/options/rest.action
$PWD = "**********"; 
//Invoice ID
$BILL_ID = "99111-ABCD-1-2-1";
$PHONE = "79191234567";

$data = array(
    "user" => "tel:+" . $PHONE,
    "amount" => "1000.00",
    "ccy" => "RUB",
    "comment" => "Good choice",
    "lifetime" => "2015-01-30T15:35:00",
    "pay_source" => "qw",
    "prv_name" => "Special packages"
);

$ch = curl_init('https://api.qiwi.com/api/v2/prv/'.$SHOP_ID.'/bills/'.$BILL_ID);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'PUT');
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($data));
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
curl_setopt($ch, CURLOPT_HTTPAUTH, CURLAUTH_BASIC);
curl_setopt($ch, CURLOPT_USERPWD, $REST_ID.":".$PWD);
curl_setopt($ch, CURLOPT_HTTPHEADER,array (
    "Accept: application/json"
));
$results = curl_exec ($ch) or die(curl_error($ch));
echo $results; 
echo curl_error($ch); 
curl_close ($ch);
//Optional user redirect
$url = 'https://bill.qiwi.com/order/external/main.action?shop='.$SHOP_ID.'&
transaction='.$BILL_ID.'&successUrl=http%3A%2F%2Fieast.ru%2Findex.php%3Froute%3D
payment%2Fqiwi%2Fsuccess&failUrl=http%3A%2F%2Fieast.ru%2Findex.php%3Froute%3D
payment%2Fqiwi%2Ffail&pay_source=card';
echo '<br><br><b><a href="'.$url.'">Redirect link to pay for invoice</a></b>';
?>
~~~

Request creates new invoice to the specified phone number which conincides with wallet ID in QIWI Wallet. Request type - HTTP PUT. 

<h3 class="request method">Request → PUT</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a></span></h3>
        <ul>
        <strong>Parameters are in the PUT-request URL pathname:</strong>
             <li><strong>prv_id</strong> - merchant’s Shop ID (numeric value, as displayed in Shop ID parameter of Protocols details section of ishop.qiwi.com web site)</li>
             <li><strong>bill_id</strong> - unique invoice identifier generated by the merchant (any non-empty string of up to 200 characters)</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json or Accept: application/json - JSON response</li>
             <li>Accept: text/xml or Accept: application/xml - XML response</li>
             <li>Content-type: application/x-www-form-urlencoded; charset=utf-8</li>
             <li>Authorization: Basic ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>Parameters are sent in the request body as <i>formdata</i>.</span>
    </li>
</ul>


Parameter|Description|Type|Required
---------|--------|---|------
user | The Visa QIWI Wallet user’s ID, to whom the invoice is issued. It is the user’s phone number with "tel:" prefix | String(20)|Y
amount | The invoice amount. The rounding up method depends on the invoice currency | Number(6.3)|Y
ccy | Invoice currency identifier (Alpha-3 ISO 4217 code). Depends on currencies allowed for the merchant. The following values are supported: RUB, EUR, USD, KZT | String(3)|Y
comment | Comment to the invoice | String(255)|Y
lifetime | Date and time up to which the invoice is available for payment. If the invoice is not paid by this date it will become void and will be assigned a final status.<br> **Important! Invoice will be automatically expired when 45 days is passed after the invoicing date**|dateTime|Y
pay_source |If the value is "mobile" the user’s MNO balance will be used as a funding source. If the value is "qw", any other funding source is used available in Visa QIWI Wallet interface. If parameter isn’t present, value "qw" is assumed |String |N 
prv_name|Merchant’s name| String(100)|N

[Response parameters](#response_bill)

[Error response](#response_error)

## Requesting Invoice Status {#invoice-status}

Merchant can request payment status of the invoice by sending the following GET-request.

~~~http
GET /api/v2/prv/2042/bills/BILL-1 HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8
~~~

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/sdf23452435"
  --header "Authorization: Basic ***" --header "Accept: text/json" 
~~~

<h3 class="request method">Request → GET</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a></span></h3>
        <ul>
        <strong>Parameters are in the GET-request URL pathname:</strong>
             <li><strong>prv_id</strong> - merchant’s Shop ID (numeric value, as displayed in Shop ID parameter of Protocols details section of ishop.qiwi.com web site)</li>
             <li><strong>bill_id</strong> - unique invoice identifier generated by the merchant (any non-empty string of up to 200 characters)</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json or Accept: application/json - JSON response</li>
             <li>Accept: text/xml or Accept: application/xml - XML response</li>
             <li>Authorization: Basic ***</li>
        </ul>
    </li>
</ul>

[Response parameters](#response_bill)

[Error response](#response_error)

## Cancelling Unpaid Invoice {#cancel}

Request cancels unpaid invoice.

~~~http
PATCH /api/v2/prv/2042/bills/BILL-1 HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8

status=rejected
~~~

~~~shell
user@server:~$ curl -X PATCH 
  --header "Authorization: Basic ***" 
  --header "Accept: text/json" 
  "https://api.qiwi.com/api/v2/prv/373712/bills/sdf23452435" 
  -d 'status=rejected'
~~~

<h3 class="request method">Request → PATCH</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a></span></h3>
        <ul>
        <strong>Parameters are in the PATCH-request URL pathname:</strong>
             <li><strong>prv_id</strong> - merchant’s Shop ID (numeric value, as displayed in Shop ID parameter of Protocols details section of ishop.qiwi.com web site)</li>
             <li><strong>bill_id</strong> - unique invoice identifier generated by the merchant (any non-empty string of up to 200 characters)</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json or Accept: application/json - JSON response</li>
             <li>Accept: text/xml or Accept: application/xml - XML response</li>
             <li>Content-type: application/x-www-form-urlencoded; charset=utf-8</li>
             <li>Authorization: Basic ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>Parameter is sent in the request body as <i>formdata</i>.</span>
    </li>
</ul>


Parameter|Value|Type|Required
---------|--------|---|------
status| `rejected` string (cancel status)|String|+

[Response parameters](#response_bill)

[Error response](#response_error)

## Refunds

Request processes a full or partial refund to user's Visa QIWI Wallet account, so a reversed transaction with the same currency is created for the initial one.

Merchant can create several refund operations for the same initial invoice provided that:

* Amount of all refund operations does not exceed initial invoice amount.
* Different refund IDs used for different refund operations of the same invoice (see below).

<aside class="warning">
When the transmitted amount exceeds the initial invoice amount or the amount left after the previous refunds, server returns error code 242.
</aside>

### Refund Operation Flow

![Refund Invoice REST API](images/pullrest_2_en.png)

1. To refund a part of the invoice amount or the full amount, merchant sends a request for refund to Visa QIWI Wallet server.
2. To make sure that the payment refund has been successfully processed, merchant can periodically request the invoice [refund status](#refund_status) until the final status is received.
3. This scenario can be repeated multiple times until the invoice is completely refunded (whole invoice amount has been returned to the user).

### Request Details

<h3 class="request method">Request → PUT</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a>/refund/<a>refund_id</a></span></h3>
        <ul>
        <strong>Parameters are in the PATCH-request URL pathname:</strong>
             <li><strong>prv_id</strong> - merchant’s Shop ID (numeric value, as displayed in Shop ID parameter of Protocols details section of ishop.qiwi.com web site)</li>
             <li><strong>bill_id</strong> - unique invoice identifier generated by the merchant (any non-empty string of up to 200 characters)</li>
             <li><strong>refund_id</strong> - refund identifier, a number specific to a series of refunds for the invoice <i>{bill_id}</i> (string of 1 to 9 symbols – any 0-9 digits and upper/lower Latin letters)</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json or Accept: application/json - JSON response</li>
             <li>Accept: text/xml or Accept: application/xml - XML response</li>
             <li>Content-type: application/x-www-form-urlencoded; charset=utf-8</li>
             <li>Authorization: Basic ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>Parameter is sent in the request body as <i>formdata</i>.</span>
    </li>
</ul>

~~~shell
user@server:~$ curl -v -w "%{http_code}" -X PUT 
  --header "Accept: text/json" 
  --header "Authorization: Basic ***" 
  "https://api.qiwi.com/api/v2/prv/373712/bills/test234578/refund/122swbill" 
  -d 'amount=10.0'
~~~

~~~http
PUT /api/v2/prv/2042/bills/BILL-1/refund/122swbill HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8

amount=10.0
~~~

Parameter|Description|Type|Required
---------|--------|---|------
amount | The refund amount should be less or equal to the amount of the initial transaction specified in<br> `{bill_id}`. The rounding up method depends on the invoice currency | Number(6.3)|Y

[Response parameters](#response_refund)

[Error response](#response_error)

## Check Refund Status

Merchant can verify current status of the refund by sending Refund Status GET-request.

<h3 class="request method">Request → GET</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a>/refund/<a>refund_id</a></span></h3>
        <ul>
        <strong>Parameters are in the GET-request URL pathname:</strong>
             <li><strong>prv_id</strong> - merchant’s Shop ID (numeric value, as displayed in Shop ID parameter of Protocols details section of ishop.qiwi.com web site)</li>
             <li><strong>bill_id</strong> - unique invoice identifier generated by the merchant (any non-empty string of up to 200 characters)</li>
             <li><strong>refund_id</strong> - refund identifier, a number specific to a series of refunds for the invoice <i>{bill_id}</i> (string of 1 to 9 symbols – any 0-9 digits and upper/lower Latin letters)</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json or Accept: application/json - JSON response</li>
             <li>Accept: text/xml or Accept: application/xml - XML response</li>
             <li>Authorization: Basic ***</li>
        </ul>
    </li>
</ul>

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/test234578/refund/122swbill"
  -v -w "%{http_code}" 
  --header "Accept: text/json" --header "Authorization: Basic ***" 
~~~

~~~http
GET /api/v2/prv/2042/bills/BILL-1/refund/122swbill HTTP/1.1
Accept: text/json
Authorization: Basic ***
Host: api.qiwi.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8
~~~

[Response parameters](#response_refund)

[Error response](#response_error)
