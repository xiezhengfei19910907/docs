# bank transfer 支付方式记录



## 1. 支付方式的配置

#### 1.1 我们网站目前的支付方式是在cms里配置的.

操作界面具体位置是 **产品 => 支付规则设置**, 选择具体的站点,国家或货币, 即可配置支付方式. 具体的数据表是 **payment_config**.



#### 1.2 新增支付方式目前不支持界面操作, 需要写SQL插入数据. 

具体数据表是 **payment**.



#### 1.3 新增支付方式描述信息目前不支持界面操作, 需要写SQL插入数据.

具体数据表是 **payment_language**.



#### 1.4 支付成功后的描述信息, 目前不支持界面操作, 需要写SQL插入数据.

具体数据表是 **multilanguage**.对应的code是 page_payment_bank_transfer_payment_desc.





## 2. 支付流程

#### 2.1 目前是在结算页, 顾客选择运费地址后, 根据cms的具体配置, 在结算页下方**Payment Methods** 处更新支付方式.

网站相关代码如下:

\esmeralda\payment\CachedPaymentService::getPaymentConfig

```php
public function getPaymentConfig($lang, $country, $currency, $enabled = 1) {
        $p1key = PaymentUtil::createPCKey($lang, $country, $currency);
        $p2key = PaymentUtil::createPCKey($lang, '', $currency);
        $p3key = PaymentUtil::createPCKey($lang, '', '');

        $pcs = $this->_getPaymentConfigs(array($p1key, $p2key, $p3key), $enabled);

        if (isset($pcs[$p1key]) && !empty($pcs[$p1key])) {
            return $pcs[$p1key];
        }
        if (isset($pcs[$p2key]) && !empty($pcs[$p2key])) {
            return $pcs[$p2key];
        }
        if (isset($pcs[$p3key]) && !empty($pcs[$p3key])) {
            return $pcs[$p3key];
        }
        return array();
    }
```

备注: 其实支付方式的配置是支持语言的, 但是我们的cms配置里并没有语言这一项.





#### 2.2 顾客选择支付方式(假设选择的是**bank transfer**), 点击结算按钮生成订单后, 我们会根据顾客订单中的国家和货币去请求gc的接口. 

网站相关代码如下:

src/php/webapp/checkout.php:261

```php
//bank transfer
if ($checkoutInfo['payment_id'] == 167) {
    $payment = $container['payment']->getPayment($checkoutInfo['payment_id']);
    $payObj = \lestore\util\Payment::gateway($payment->payment_code);
    $payResult = $payObj->process($orderInfo, $lang_code);
    if ($payResult['code'] == 'REQUEST_FAILED') {
        $orderInfo = $container['user.order']->get($checkoutInfo['user_id'], $sn, $lang_code);
        $checkoutInfo['payment_id'] = $orderInfo->payment_id;
    }

    $logHandle = $payObj->get_log_handle();
    $logHandle->add(__FILE__, print_r($payResult, true), 'OK', $orderInfo->order_sn);
}
```



\lestore\util\paymentgateway\Bank_transfer::process

```php
public function process($order, $lang_code) {
        global $container;

        $order = is_object($order) ? (array)$order : $order;

        //check param
        if (empty($order) || empty($order['order_sn'])) {
            return array('res' => 'ERROR', 'code' => 'NON_ORDER');
        }

        //set order sn
        $this->set_order_sn($order['order_sn']);

        //output params
        $params = $this->outParams($order, $lang_code);

        //genarate xml
        $requestXml = $this->genareateRequest($params);

        //send request
        $responseData = $this->sendRequest($requestXml, $params);

        $pay_result = false;
        if (!empty($responseData)) {
            $xml = simplexml_load_string($responseData);
            $xml = Legacy::xml2array($xml);
            if (!empty($xml['REQUEST']['RESPONSE']['RESULT']) && ($xml['REQUEST']['RESPONSE']['RESULT'] == 'OK') &&
                ($xml['REQUEST']['RESPONSE']['ROW']['STATUSID'] == '800')) {
                $pay_result = true;
                $this->recordOrderPayment($xml, $params);
            }
        }

        //request failed, update order payment_id and payment_name
        if (empty($pay_result)) {
            $payment = $container['payment']->getPayment(158);  // wire_transfer_jjshouse
            $updateResult = $this->updateOrderPaymentMethod($payment);
            if (empty($updateResult['update'])) {   // update failed
                $container['alert']->sev2(OrderCode::UPDATE_ERROR, 'update bank transfer order payment method failed', func_get_args());
            }
        }

        //record transaction log
        $txn_id = '';
        if (isset($xml['REQUEST']['RESPONSE']['ROW']['EXTERNALREFERENCE'])) {
            $txn_id = $xml['REQUEST']['RESPONSE']['ROW']['EXTERNALREFERENCE'];
        } else if (isset($xml['REQUEST']['RESPONSE']['ROW']['ADDITIONALREFERENCE'])) {
            $txn_id = $xml['REQUEST']['RESPONSE']['ROW']['ADDITIONALREFERENCE'];
        } else {
            $txn_id = !empty($xml['REQUEST']['RESPONSE']['ERROR']['CODE']) ? $xml['REQUEST']['RESPONSE']['ERROR']['CODE'] : null;
        }

        $_status = $pay_result ? 'REQUEST_SUCCESS' : 'REQUEST_FAILED';
        $txn_data = array(
            'txn_id'     => $txn_id,
            'order_sn'   => $this->order_sn,
            'txn_post'   => $responseData,
            'txn_status' => $xml['REQUEST']['RESPONSE']['RESULT'],
            'site_code'  => $_status,
            'txn_result' => $pay_result ? 'OK' : 'NOK',
        );

        return array(
            'code' => $_status,
            'res'  => $txn_data,
        );
    }
```





##### 2.2.1 请求成功后gc返回的原始数据如下:

```xml
<XML>
<REQUEST>
    <ACTION>INSERT_ORDERWITHPAYMENT</ACTION>
    <META>
        <MERCHANTID>7441</MERCHANTID>
        <IPADDRESS>50.19.255.229</IPADDRESS>
        <VERSION>2.0</VERSION>
        <REQUESTIPADDRESS>127.0.0.2</REQUESTIPADDRESS>
    </META>
    <PARAMS>
        <ORDER>
            <ORDERID>8682573074</ORDERID>
            <MERCHANTREFERENCE>_8682573074_160527040748</MERCHANTREFERENCE>
            <AMOUNT>2400</AMOUNT>
            <CURRENCYCODE>EUR</CURRENCYCODE>
            <LANGUAGECODE>en</LANGUAGECODE>
            <COUNTRYCODE>FI</COUNTRYCODE>
            <SURNAME>56756765</SURNAME>
            <CITY>23432</CITY>
            <FIRSTNAME>3243243244</FIRSTNAME>
            <STREET>324234324234</STREET>
            <ADDITIONALADDRESSINFO></ADDITIONALADDRESSINFO>
            <ZIP>2343243243</ZIP>
            <STATE></STATE>
        </ORDER>
        <PAYMENT>
            <PAYMENTPRODUCTID>11</PAYMENTPRODUCTID>
            <AMOUNT>2400</AMOUNT>
            <CURRENCYCODE>EUR</CURRENCYCODE>
            <COUNTRYCODE>FI</COUNTRYCODE>
            <LANGUAGECODE>en</LANGUAGECODE>
            <HOSTEDINDICATOR>0</HOSTEDINDICATOR>
        </PAYMENT>
    </PARAMS>
    <RESPONSE>
        <RESULT>OK</RESULT>
        <META>
            <RESPONSEDATETIME>20160527060750</RESPONSEDATETIME>
            <REQUESTID>478727</REQUESTID>
        </META>
        <ROW>
            <SWIFTCODE>NDEAFIHH</SWIFTCODE>
            <STATUSDATE>20160527060750</STATUSDATE>
            <ATTEMPTID>1</ATTEMPTID>
            <PAYMENTREFERENCE>744100007599</PAYMENTREFERENCE>
            <IBAN>FI8521531800030818</IBAN>
            <COUNTRYDESCRIPTION>Finland</COUNTRYDESCRIPTION>
            <MERCHANTID>7441</MERCHANTID>
            <ADDITIONALREFERENCE>_8682573074_16052704</ADDITIONALREFERENCE>
            <STATUSID>800</STATUSID>
            <ORDERID>8682573074</ORDERID>
            <ACCOUNTHOLDER>Global Collect BV</ACCOUNTHOLDER>
            <EXTERNALREFERENCE>_8682573074_160527040748</EXTERNALREFERENCE>
            <EFFORTID>1</EFFORTID>
            <BANKNAME>Nordea Bank</BANKNAME>
            <CITY>Helsinki</CITY>
            <BANKACCOUNTNUMBER>215318-30818</BANKACCOUNTNUMBER>
        </ROW>
    </RESPONSE>
</REQUEST>
</XML>
```

里面包含了顾客要汇款的具体银行帐号信息, 其中**PAYMENTREFERENCE** 字段很重要且是唯一的. gc根据它来标志每一笔交易.





##### 2.2.2 请求失败后, 会更改订单的支付方式为wire transfer. 

失败后显示香港的银行帐号. gc返回的数据记录在pay_log表中. 失败的可能原因如下:

- 我们的ip不在gc的ip白名单中, gc返回数据如下:

  ```xml
        <XML>
            <REQUEST>
                <ACTION>INSERT_ORDERWITHPAYMENT</ACTION>
                <META>
                    <MERCHANTID>7441</MERCHANTID>
                    <IPADDRESS>50.19.255.229</IPADDRESS>
                    <VERSION>2.0</VERSION>
                    <REQUESTIPADDRESS>127.0.0.2</REQUESTIPADDRESS>
                </META>
                <PARAMS>
                    <ORDER>
                        <ORDERID>5788139003</ORDERID>
                        <MERCHANTREFERENCE>_5788139003_160607195825</MERCHANTREFERENCE>
                        <AMOUNT>30097</AMOUNT>
                        <CURRENCYCODE>USD</CURRENCYCODE>
                        <LANGUAGECODE>en</LANGUAGECODE>
                        <COUNTRYCODE>DE</COUNTRYCODE>
                        <SURNAME>234234</SURNAME>
                        <CITY>23423423</CITY>
                        <FIRSTNAME>214324</FIRSTNAME>
                        <STREET>23432432</STREET>
                        <ADDITIONALADDRESSINFO></ADDITIONALADDRESSINFO>
                        <ZIP>234234324</ZIP>
                        <STATE>Berlin</STATE>
                    </ORDER>
                    <PAYMENT>
                        <PAYMENTPRODUCTID>11</PAYMENTPRODUCTID>
                        <AMOUNT>30097</AMOUNT>
                        <CURRENCYCODE>USD</CURRENCYCODE>
                        <COUNTRYCODE>DE</COUNTRYCODE>
                        <LANGUAGECODE>en</LANGUAGECODE>
                        <HOSTEDINDICATOR>0</HOSTEDINDICATOR>
                    </PAYMENT>
                </PARAMS>
                <RESPONSE>
                    <RESULT>NOK</RESULT>
                    <META>
                        <RESPONSEDATETIME>20160608045826</RESPONSEDATETIME>
                        <REQUESTID>75700</REQUESTID>
                    </META>
                    <ERROR>
                        <CODE>102020</CODE>
                        <MESSAGE>ACTION 130 IS NOT ALLOWED FOR MERCHANT 7441, IPADDRESS 54.234.218.49</MESSAGE>
                    </ERROR>
                </RESPONSE>
            </REQUEST>
        </XML>
  ```

    **解决方式: 联系财务通知gc, 添加ip到gc的ip白名单.**



- 订单中的国家 + 货币, gc bank transfer 不支持. gc返回数据如下:

  ```xml
        <XML>
            <REQUEST>
                <ACTION>INSERT_ORDERWITHPAYMENT</ACTION>
                <META>
                    <MERCHANTID>7441</MERCHANTID>
                    <IPADDRESS>50.19.255.229</IPADDRESS>
                    <VERSION>2.0</VERSION>
                    <REQUESTIPADDRESS>127.0.0.2</REQUESTIPADDRESS>
                </META>
                <PARAMS>
                    <ORDER>
                        <ORDERID>9823356303</ORDERID>
                        <MERCHANTREFERENCE>_9823356303_160616060051</MERCHANTREFERENCE>
                        <AMOUNT>2598</AMOUNT>
                        <CURRENCYCODE>USD</CURRENCYCODE>
                        <LANGUAGECODE>en</LANGUAGECODE>
                        <COUNTRYCODE>DE</COUNTRYCODE>
                        <SURNAME>fdsfds</SURNAME>
                        <CITY>23432423</CITY>
                        <FIRSTNAME>fdsfdsf</FIRSTNAME>
                        <STREET>dsfdsfdsfdsf</STREET>
                        <ADDITIONALADDRESSINFO></ADDITIONALADDRESSINFO>
                        <ZIP>2343243243</ZIP>
                        <STATE>Berlin</STATE>
                    </ORDER>
                    <PAYMENT>
                        <PAYMENTPRODUCTID>11</PAYMENTPRODUCTID>
                        <AMOUNT>2598</AMOUNT>
                        <CURRENCYCODE>USD</CURRENCYCODE>
                        <COUNTRYCODE>DE</COUNTRYCODE>
                        <LANGUAGECODE>en</LANGUAGECODE>
                        <HOSTEDINDICATOR>0</HOSTEDINDICATOR>
                    </PAYMENT>
                </PARAMS>
                <RESPONSE>
                    <RESULT>NOK</RESULT>
                    <META>
                        <RESPONSEDATETIME>20160616080059</RESPONSEDATETIME>
                        <REQUESTID>1132720</REQUESTID>
                    </META>
                    <ERROR>
                        <CODE>430900</CODE>
                        <MESSAGE>NO VALID PROVIDERS FOUND FOR COMBINATION MERCHANTID: 7441, PAYMENTPRODUCT: 11, COUNTRYCODE: DE,
                            CURRENCYCODE: USD
                        </MESSAGE>
                    </ERROR>
                    <ROW>
                        <ORDERID>9823356303</ORDERID>
                    </ROW>
                </RESPONSE>
            </REQUEST>
        </XML>
  ```

    **解决方式: 联系产品, 决定是否需要修改该国家 + 货币的支付方式.**


- 订单金额超过了gc bank transfer的范围. gc返回数据如下:

  ```xml
  <XML>
      <REQUEST>
          <ACTION>INSERT_ORDERWITHPAYMENT</ACTION>
          <META>
              <MERCHANTID>7441</MERCHANTID>
              <IPADDRESS>50.19.255.229</IPADDRESS>
              <VERSION>2.0</VERSION>
              <REQUESTIPADDRESS>127.0.0.2</REQUESTIPADDRESS>
          </META>
          <PARAMS>
              <ORDER>
                  <ORDERID>3273203462</ORDERID>
                  <MERCHANTREFERENCE>_3273203462_160728023256</MERCHANTREFERENCE>
                  <AMOUNT>8420808</AMOUNT>
                  <CURRENCYCODE>GBP</CURRENCYCODE>
                  <LANGUAGECODE>en</LANGUAGECODE>
                  <COUNTRYCODE>GI</COUNTRYCODE>
                  <SURNAME>dh</SURNAME>
                  <CITY>xghhh</CITY>
                  <FIRSTNAME>dfgg</FIRSTNAME>
                  <STREET>xghhgffhhh</STREET>
                  <ADDITIONALADDRESSINFO/>
                  <ZIP>cdfdggg</ZIP>
                  <STATE>dfggg</STATE>
              </ORDER>
              <PAYMENT>
                  <PAYMENTPRODUCTID>11</PAYMENTPRODUCTID>
                  <AMOUNT>8420808</AMOUNT>
                  <CURRENCYCODE>GBP</CURRENCYCODE>
                  <COUNTRYCODE>GI</COUNTRYCODE>
                  <LANGUAGECODE>en</LANGUAGECODE>
                  <HOSTEDINDICATOR>0</HOSTEDINDICATOR>
              </PAYMENT>
          </PARAMS>
          <RESPONSE>
              <RESULT>NOK</RESULT>
              <META>
                  <RESPONSEDATETIME>20160728113257</RESPONSEDATETIME>
                  <REQUESTID>11539387</REQUESTID>
              </META>
              <ERROR>
                  <CODE>410120</CODE>
                  <MESSAGE>AMOUNT_NOT_BETWEEN_MINAMOUNT_AND_MAXAMOUNT</MESSAGE>
              </ERROR>
              <ROW>
                  <ORDERID>3273203462</ORDERID>
              </ROW>
          </RESPONSE>
      </REQUEST>
  </XML>
  ```

  ​



## **3. 同步订单的支付状态**

目前在cms中有cronjob同步gc订单的支付状态.

具体代码: \CronjobGC_Controller::getGcOrderPayStatusAction.

主要流程如下:

1.  判断订单状态(pay_status)如果未支付, 请求gc接口, 获取订单的支付信息.
2.  如果订单的支付状态在 [1000, 1050], 则认为顾客已经汇款了.
3.  检查 顾客实际汇款给我们的金额 减去 我们退款给他们的金额 是否小于订单的金额, 如果不小于, 则认为订单支付成功.
4.  修改订单的支付状态.




从gc获取到的订单的信息如下:

```xml
<XML>
<REQUEST>
    <ACTION>GET_ORDERSTATUS</ACTION>
    <META>
        <MERCHANTID>7441</MERCHANTID>
        <IPADDRESS>50.19.255.229</IPADDRESS>
        <VERSION>2.0</VERSION>
        <REQUESTIPADDRESS>127.0.0.2</REQUESTIPADDRESS>
    </META>
    <PARAMS>
        <ORDER>
            <ORDERID>1489502872</ORDERID>
        </ORDER>
    </PARAMS>
    <RESPONSE>
        <RESULT>OK</RESULT>
        <META>
            <RESPONSEDATETIME>20160523042557</RESPONSEDATETIME>
            <REQUESTID>5376598</REQUESTID>
        </META>
        <STATUS>
            <STATUSDATE>20160521122141</STATUSDATE>
            <PAYMENTMETHODID>7</PAYMENTMETHODID>
            <MERCHANTREFERENCE>_1489502872_160511015050</MERCHANTREFERENCE>
            <ATTEMPTID>1</ATTEMPTID>
            <PAYMENTREFERENCE>744100252239</PAYMENTREFERENCE>
            <AMOUNT>15298</AMOUNT>
            <MERCHANTID>7441</MERCHANTID>
            <TOTALAMOUNTPAID>14833</TOTALAMOUNTPAID>
            <STATUSID>1050</STATUSID>
            <ORDERID>1489502872</ORDERID>
            <TOTALAMOUNTREFUNDED>0</TOTALAMOUNTREFUNDED>
            <EFFORTID>1</EFFORTID>
            <CURRENCYCODE>EUR</CURRENCYCODE>
            <PAYMENTPRODUCTID>11</PAYMENTPRODUCTID>
        </STATUS>
    </RESPONSE>
</REQUEST>
</XML>
```




