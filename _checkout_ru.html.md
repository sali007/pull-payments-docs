# Форма оплаты {#checkout_ru}

###### Последнее обновление: 2017-07-11 | [Редактировать на GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/_checkout_ru.html.md)


Провайдер может предложить пользователю немедленно оплатить счет с помощью переадресации на платежную форму посредством HTTP GET-запроса по адресу:

<h3 class="request method">Запрос → REDIRECT</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://bill.qiwi.com/order/external/main.action</span></h3>
    </li>
</ul>

<aside class="notice">
В ответ сервер формирует на сайте Visa QIWI Wallet страницу с выставленным счетом и выбором способа оплаты счета.

Если провайдер использует выставление счета через [веб-форму](#webform_ru), данное действие выполняется автоматически.
</aside>

<ul class="nestedList params">
    <li><h3>Параметры</h3>
    </li>
</ul>

Параметр|Тип|Описание|Обяз.
---------|--------|-------|-----
shop| Строка|Идентификатор провайдера. Соответствует параметру `{prv_id}` из [запроса на выставление счета](#invoice-rest).|+
transaction|Строка|Идентификатор счета в информационной системе провайдера. Соответствует параметру `{bill_id}` из запроса на выставление счета.|+
iframe|Логический, true/false| Признак отображения страницы в iframe (более компактный вид, удобный для встраивания ее в сайт провайдера). По умолчанию `false`|-
successUrl|URL-закодированная строка|URL для переадресации в случае успешного создания транзакции в Visa QIWI Wallet. Ссылка должна вести на сайт провайдера. Если пользователь выбрал на платежной форме способ оплаты, отличный от оплаты с баланса Visa QIWI Wallet, то переадресация на сайт провайдера не выполняется.|-
failUrl|URL-закодированная строка|URL для переадресации в случае неуспеха при создании транзакции в Visa QIWI Wallet. Ссылка должна вести на сайт провайдера. Если пользователь выбрал на платежной форме способ оплаты, отличный от оплаты с баланса Visa QIWI Wallet, то переадресация на сайт провайдера не выполняется.|-
target|Строка "iframe"|Флаг, показывающий, что ссылки в параметрах <br>`successUrl` / `failUrl` открываются в iframe. Если отсутствует, то считается выключенным|-
pay_source|Строка| Способ оплаты по умолчанию, который необходимо отобразить пользователю при открытии платежной формы. Возможные значения:<br>`qw` – оплата с баланса Visa QIWI Wallet;<br> `mobile` – оплата с баланса мобильного телефона;<br> `card` – оплата банковской картой;<br> `wm` – оплата с привязанного кошелька WebMoney;<br> `ssk` – оплата наличными в терминале QIWI.<br>Если способ оплаты не доступен, пользователю отображается предупреждение, при этом на странице можно выбрать другие способы оплаты.|-

## Возврат на сайт провайдера {#status-links}

<aside class="notice">
Если в ссылке на платежную форму указан параметр `successUrl` или `failUrl`, то сайт Visa QIWI Wallet переадресует пользователя на соответствующий URL после завершения процесса оплаты.
</aside>

<aside class="warning">
Сам по себе факт перенаправления на адрес, указанный в параметре `successUrl`, не означает, что счет успешно оплачен.
Для принятия окончательного решения о предоставлении клиенту услуги или товара провайдеру необходимо дождаться [уведомления](#notification_ru) от сервера Visa QIWI Wallet с финальным статусом счета. Если провайдер не использует уведомления, необходимо запрашивать статус счета [отдельным запросом API](#invoice-status).
</aside>

<aside class="notice">
При переадресации на сайт провайдера добавляется дополнительный параметр `order`, в котором будет передан идентификатор счета (значение параметра `transaction` из первоначального GET-запроса платежной формы). Используя этот параметр, провайдер может отобразить необходимую информацию на своей стороне.
</aside>

## Пример использования {#checkout-examples}

* Провайдер после выставления счета переадресует пользователя на URL:

    * `https://bill.qiwi.com/order/external/main.action?shop=2042&transaction=1234567&successUrl=http%3A%2F%2Fmystore.com%2Fsuccess%3Fa%3D1%26b%3D2&failUrl=http%3A%2F%2Fmystore.com%2Ffail%3Fa%3D1%26b%3D2&iframe=true&target=iframe&pay_source=qw`


* Клиент видит на странице метод оплаты с баланса Visa QIWI Wallet (отображается в iframe) и оплачивает счет этим методом.

* После и успешного создания транзакции сайт Visa QIWI Wallet выполняет возврат клиента на страницу:

    * `http://mystore.com/success?a=1&b=2&order=1234567` (отображается в iframe).


* В случае неуспеха при создании транзакции сайт Visa QIWI Wallet выполняет возврат клиента на страницу:

    * `http://mystore.com/fail?a=1&b=2&order=1234567` (отображается в iframe).
