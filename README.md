Pingpp PHP SDK
=================
## 简介
lib 文件夹下是 PHP SDK 文件，  
example 文件夹里面是简单的接入示例，该示例仅供参考。

## 版本要求
PHP 版本 5.3 及以上  
你可以执行目录下的环境检测脚本，来进行一些基本检测
``` bash
php PingppEnvInspect.php
```

## 安装
### 使用 Composer
执行
```
composer require pingplusplus/pingpp-php
```

使用 Composer 的 autoload 引入
```php
require_once('vendor/autoload.php');
```

### 手动引入
``` php
require_once('/path/to/pingpp-php/init.php');
```

## 接入方法
### 初始化
```php
\Pingpp\Pingpp::setApiKey('YOUR-KEY');
```

### 设置请求签名密钥
密钥需要你自己生成，公钥请填写到 [Ping++ Dashboard](https://dashboard.pingxx.com)
```php
\Pingpp\Pingpp::setPrivateKeyPath('/path/to/your_rsa_private_key.pem');
```

### 支付
```php
$ch = \Pingpp\Charge::create(
    array(
        'order_no'  => '123456789',
        'app'       => array('id' => 'APP_ID'),
        'channel'   => 'alipay',
        'amount'    => 100,
        'client_ip' => '127.0.0.1',
        'currency'  => 'cny',
        'subject'   => 'Your Subject',
        'body'      => 'Your Body',
        'extra'     => $extra
    )
);
```

> 由于 PHP 5.3 不支持 `JsonSerializable` JSON 序列化接口，当你需要将其放入其他数组或者对象时，
> 建议先将其转成数组，例：`$arr = array('charge' => json_decode($ch, true))`。

### charge 查询
```php
\Pingpp\Charge::retrieve('CHARGE_ID');
```

```php
\Pingpp\Charge::all(array('limit' => 5, 'app' => array('id' => 'APP_ID')));
```

### 退款
``` php
$ch = \Pingpp\Charge::retrieve('CHARGE_ID');
$re = $ch->refunds->create(array('description' => 'Refund Description');
```

### 退款查询
```php
$ch = \Pingpp\Charge::retrieve('CHARGE_ID');
$ch->refunds->retrieve('REFUND_ID');
```


```php
$ch = \Pingpp\Charge::retrieve('CHARGE_ID');
$ch->refunds->all(array('limit' => 5));
```

### 微信红包
```php
\Pingpp\RedEnvelope::create(
    array(
        'order_no'  => '123456789',
        'app'       => array('id' => 'APP_ID'),
        'channel'   => 'wx_pub',
        'amount'    => 100,
        'currency'  => 'cny',
        'subject'   => 'Your Subject',
        'body'      => 'Your Body',
        'extra'     => array(
            'nick_name' => 'Nick Name',
            'send_name' => 'Send Name'
        ),
        'recipient'   => 'Openid',
        'description' => 'Your Description'
    )
);
```

### 查询指定微信红包
```php
\Pingpp\RedEnvelope::retrieve('RED_ID');
```

### 查询微信红包列表
```php
\Pingpp\RedEnvelope::all(array('limit' => 5));
```

### 微信公众号获取签名
如果使用微信 JS-SDK 来调起支付，需要在创建 `charge` 后，获取签名（`signature`），传给 HTML5 SDK。
```php
$jsapi_ticket_arr = \Pingpp\WxpubOAuth::getJsapiTicket($wx_app_id, $wx_app_secret);
$ticket = $jsapi_ticket_arr['ticket'];
```
**正常情况下，`jsapi_ticket` 的有效期为 7200 秒。由于获取 `jsapi_ticket` 的 api 调用次数非常有限，频繁刷新 `jsapi_ticket` 会导致 api 调用受限，影响自身业务，开发者必须在自己的服务器全局缓存 `jsapi_ticket`。**

_下面方法中 `$url` 是当前网页的 URL，不包含 `#` 及其后面部分_
```php
$signature = \Pingpp\WxpubOauth::getSignature($charge, $ticket, $url);
```
然后在 HTML5 SDK 里调用
```js
pingpp.createPayment(charge, callback, signature, false);
```

### event 查询

```php
\Pingpp\Event::retrieve('EVT_ID');
```

### event 列表查询
```php
\Pingpp\Event::all(array('type' => 'charge.succeeded'));
```
**详细信息请参考 [API 文档](https://pingxx.com/document/api?php)。**

### 微信企业付款
```php
\Pingpp\Transfer::create(
    array(
        'amount' => 100,
        'order_no' => '123456d7890',
        'currency' => 'cny',
        'channel' => 'wx_pub',
        'app' => array('id' => 'APP_ID'),
        'type' => 'b2c',
        'recipient' => 'o9zpMs9jIaLynQY9N6yxcZ',
        'description' => 'testing',
        'extra' => array('user_name' => 'User Name', 'force_check' => true)
    )
);
```

### 查询指定 transfer
```php
\Pingpp\Transfer::retrieve('TR_ID');
```

### 查询 transfer 列表
```php
\Pingpp\Transfer::all(array('limit' => 5));
```

### 查询卡片信息
```php
\Pingpp\CardInfo::query(array(
    'app' => 'APP_ID',
    'card_number' => 'ENCRYPTED_CARD_NUMBER'
));
```



### 身份证认证
``` php
\Pingpp\Identification::identify(array(
    'type' => 'id_card',
    'app' => $app_id,
    'data' => array(
        'id_name' => '张三', // 姓名
        'id_number' => '310181198910107641' // 身份证号
    )
));
```

### 银行卡认证
``` php
\Pingpp\Identification::identify(array(
    'type' => 'bank_card',
    'app' => $app_id,
    'data' => array(
        'id_name' => '张三', // 姓名
        'id_number' => '310181198910107641', // 身份证号,
        'card_number' => '6201111122223333', // 银行卡号
        'phone_number' => '18623234545' // 银行预留手机号，不支持 178 号段
    )
));
```

### 批量转账
``` php
\Pingpp\BatchTransfer::create(
    [
        'amount'      => 8000,
        'app'         => APP_ID,
        'batch_no'    => 'Your batch no',       //批量退款批次号，3-24位，允许字母和英文
        'channel'     => 'alipay',
        'description' => 'Your Description',    //批量退款详情，最多 255 个 Unicode 字符
        'recipients'  => [                      //需要退款的  charge id 列表，一次最多 100 个
            [
                'account' => 'account01@alipay.com',
                'amount'  => 5000,
                'name'    => '张三'
            ],
            [
                'account' => 'account02@alipay.com',
                'amount'  => 3000,
                'name'    => '李四'
            ]
        ],
        'type'      => 'b2c'
    ]
);
```

### 查询指定批量转账
``` php
\Pingpp\BatchTransfer::retrieve('181611151506412852');        //批量转账对象id ，由 Ping++ 生成
```

### 查询批量转账列表
``` php
\Pingpp\BatchTransfer::all(['page'=>1]);
```

### 批量退款
``` php
\Pingpp\BatchRefund::create(
    [
        'app'         => APP_ID,
        'batch_no'    => 'Your batch no',       //批量退款批次号，3-24位，允许字母和英文
        'description' => 'Your Description',    //批量退款详情，最多 255 个 Unicode 字符
        'charges'     => [                      //需要退款的  charge id 列表，一次最多 100 个
            'ch_qn5G8GH1SOCCvnv10S8mXTqP',
            'ch_SijjXL8Ki1u1arL1S49q5ifL'
        ]
    ]
);
```

### 查询指定批量退款
``` php
\Pingpp\BatchRefund::retrieve('151611141520583238');        //批量退款对象id ，由 Ping++ 生成
```

### 查询批量退款列表
``` php
\Pingpp\BatchRefund::all(['page'=>1]);
```

### 报关
``` php
\Pingpp\Custom::create(
    [
        'app'               => APP_ID,
        'charge'            => 'ch_L8qn10mLmr1GS8e5OODmHaL4',
        'channel'           => 'alipay',
        'trade_no'          => '12332132131',         // 商户报关订单号,8~20位
        'customs_code'      => 'GUANGZHOU',
        'amount'            => 8000,
        'transport_amount'  => 10,
        'is_split'          => true,
        'sub_order_no'      => '123456',
        'extra'  => [
            'pay_account'   => '1234567890',
            'certif_type'   => '02',
            'customer_name' => 'A Name',
            'certif_id'     => 'ID Card No',
            'tax_amount'    => '10',
        ]
    ]
);
```

### 查询指定报关
``` php
\Pingpp\Custom::retrieve('14201609281040220109');           // 报关对象id ，由 Ping++ 生成
```


