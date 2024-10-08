# JPM
JP Morgan Checkout
## Class information

This document provides instructions on how to use the \`checkout\` PHP class for processing transactions using the JPMorgan Chase API. The guide covers configuration, adding items, setting customer and shipping details, generating tokens, and processing checkout requests.

## Requirements

> \- PHP 7.2+  
> \- Composer  
> \- Dependencies: \`firebase/php-jwt\`

---

## JSON configuration files

Before using the \`checkout\` class, you need to create a configuration file (e.g., \`config.json\`) with the following structure:

```

{
   "privateKey": "path/to/private/key",
   "certPath": "path/to/cert",
   "client_id": "your_client_id",
   "resource_id": "your_resource_id",
   "merchant_id": "your_merchant_id"
}
```

In addition you must have the public and private key to use this class.

---

## Using the class

### Initializing the Checkout Class

Initialize the class by passing the path to your configuration file:

```

require_once 'uh/vendors/jpmorgan/checkout.php';
$configPath = 'path/to/config.json';
$checkout = new checkout($configPath);
```

---

### Adding Configuration Items

While you don't need to add any additional items to the Configuration it is helpful to add values to use a single source.

Return a single value of what the key contains

```

$checkout->addToConfig('new_key', 'new_value');
```

---

### Getting Confirmation Items

To retreive a confirmation item you can use

```
$checkout->getConfig('key');

// to get all keys as an array

$checkout->getConfig();
```

### Generating a JWT key

This is not nessesary because the class makes care of this for you. Â If you are needing the JWT token you can generate it like this.

Returns a JWT String

```

$jwtToken = $checkout->generateToken();
```

### Generating an accessToken

This is not nessesary but if you are needing an access token to use the API you can generate one as follows.

```

$accessToken = $checkout->getAccessToken();
```

Returns

```
return [
    "access_token" => "",
    "token_type" => "error",
    "expires_in" => "",
    "bearer_token" => $this->config['jwtToken']
];
```

---

## How to do a checkout

To do a checkout you need to do the following

1.  Create your cart as you normally would.
2.  Go to a page that makes the checkout intent.
    1.  Your intent must have at least one item.
    2.  Optional Customer information.
    3.  Optional Shipping information.
3.  Generate the url that you will use to redirect them to the hosted checkout page.

Example:

```
require_once 'uh/vendors/jpmorgan/checkout.php';
$jpmorgan = new checkout('/publish/lib/auth/jp_morgan_payment_configs/test-certs/config1.json');


$jpmorgan->addItem([
            "id"=>"ITEM$lp",
            "name"=>"Item $lp",
            "description"=>"this is an item desc of $lp",
            "quantity"=>"1",
            "unitPrice"=>"$x"
            "imageUrl" => "<optional> can be url or base64 encoded image"
    ]);

/* OPTIONAL */
$jpmorgan->setCustomer([
        "email"=>"rbirkline@uh.edu",
        "phone"=>"26493",
        "name"=>"Robert Birkline",
        "address1"=>"my address",
        "address2"=>"my address 2",
        "city"=>"Houston",
        "state"=>"Texas",
        "country"=>"US",
        "postalCode"=>"77204"
]);

/* OPTIONAL */
$jpmorgan->setShippingAddress([
        "name"=>"Shipping",
        "address1"=>"Ship 1",
        "address2"=>"Ship 2",
        "city"=>"Houston 2",
        "state"=>"some state",
        "country"=>"US",
        "postalCode"=>"11111"
]);

$request = $jpmorgan->generateCheckoutRequest();
```

$request returns

```
[
    "statusCode"        =>    $info['http_code'],
    "responseStatus"    => "ok",
    "responseMessage"    => "normal redirect to payment page",
    "requestID"            => $requestId,
    "orderNumber"        => $orderNumber,
    "merchantID"        => $this->config['merchant_id'],
];
```

You will need to save the **orderNumber.** This is used to retrieve the details of the transaction later.

Suggestion something like this that would store your accessToken and order number so that you can use it later

```
setcookie('aToken', $jpmorgan->getConfig('accessToken'), (time()+60*15), '/', '.uh.edu', true, true );
setcookie('tOrderNumber', $request['orderNumber'], (time()+60*15), '/', '.uh.edu', true, true );
```

Then you can redirect the user to the hosted checkout page

```
if ($request['redirectedUrl']){
    header("Location: ".$request['redirectedUrl']);
}else{
    print "<pre>";
    print_r($request);
    print "</pre>";
}
```

## To retrieve information about the request

This page MUST be publically accessible for JP Morgan to redirect the user to.

An example of how you would do this is as follows

```
require_once 'uh/vendors/jpmorgan/checkout.php';
$p = new checkout('/publish/lib/auth/jp_morgan_payment_configs/test-certs/config1.json');
$p->addToConfig('accessToken',$_COOKIE['aToken']);
$p->addToConfig('orderNumber',$_COOKIE['tOrderNumber']);

print "<pre>";
print_r($p->getNotification('-2 min',$_COOKIE['tOrderNumber']));
print "</pre>";
