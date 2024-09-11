# Laravel PayHere

[← back](https://milroyfraser.github.io)

<br>

> [PayHere](https://www.payhere.lk/) is the leading payment service provider for **Sri Lanka**.

<br/>

- [Read the step by step guide to develop a E-commerce wesite using this package](https://milroy.me/step-by-step-guid-to-implement-an-e-commerce-website-with-laravel-payhere)
- [Watch the video tutorial (Sinhala)](https://www.youtube.com/playlist?list=PLtX53M2iMi4_riAi4ZBy8i23RzpBnTzgz)
- [GitHub](https://github.com/apichef/laravel-pay-here)

<br/>

## Installation

```bash
$ composer require apichef/laravel-pay-here
```

#### Publish migrations and config
```bash
$ php artisan vendor:publish --provider="ApiChef\PayHere\PayHereServiceProvider"
```

#### Run migrations
```bash
$ php artisan migrate
```

<br/>
<br/>

## Configurations

```php
return [
    /*
    |--------------------------------------------------------------------------
    | Base Url
    |--------------------------------------------------------------------------
    |
    | This is where you can configure PayHere base url, based on the
    | application environment. On production, the value should set to
    | https://www.payhere.lk/
    |
    */

    'base_url' => env('PAY_HERE_BASE_URL', 'https://sandbox.payhere.lk/'),

    /*
    |--------------------------------------------------------------------------
    | Merchant Credentials
    |--------------------------------------------------------------------------
    |
    | Merchant ID is issued per account by PayHere, You can copy it from your
    | PayHere settings page in the Domains & Credentials tab.
    |
    | To obtain a merchant secret, you need to add a domain to allowed
    | domains/apps.
    |
    */

    'merchant_credentials' => [
        'id' => env('PAY_HERE_MERCHANT_ID'),
        'secret' => env('PAY_HERE_MERCHANT_SECRET'),
    ],

    /*
    |--------------------------------------------------------------------------
    | Business App Credentials
    |--------------------------------------------------------------------------
    |
    | You must create a business application to call other PayHere services. Go 
    | to the Business Apps tab and create an app.
    |
    | NOTE: Tick the following permission
    | - Payment Retrieval API
    | - Subscription Management API (only if you use subscriptions)
    |
    */

    'business_app_credentials' => [
        'id' => env('PAY_HERE_BUSINESS_APP_ID'),
        'secret' => env('PAY_HERE_BUSINESS_APP_SECRET'),
    ],

    /*
    |--------------------------------------------------------------------------
    | PayHere Database Connection
    |--------------------------------------------------------------------------
    |
    | This is the database connection you want PayHere to use while storing &
    | reading your payment data. By default PayHere assumes you use your
    | default connection. However, you can change that to anything you want.
    |
    */

    'database_connection' => env('DB_CONNECTION'),

    /*
    |--------------------------------------------------------------------------
    | PayHere Middleware
    |--------------------------------------------------------------------------
    |
    | This is the middleware group that PayHere payment notification webhook
    | and redirect on success/canceled routes uses.
    |
    */

    'middleware' => env('PAY_HERE_MIDDLEWARE', []),
]
```

<br/>
<br/>

## Create a payable instance

When you are implementing the checkout form you need to pass a payable (payment or a subscription) instance. To do this package provides **make** static method in the **Payment** and **Subscription** classes.

<br/>

<small>payment example</small>
```php
namespace App\Http\Controllers;

use ApiChef\PayHere\Payment;
use App\Product;
use Illuminate\Http\Request;

class CheckoutController extends Controller
{
    public function show(Product $product, Request $request)
    {
        $payment = Payment::make($product, $request->user(), $product->price);

        return view('checkout')
            ->with('payable', $payment);
    }
}
```
<br/>

<small>subscription example</small>
```php
namespace App\Http\Controllers;

use ApiChef\PayHere\Subscription;
use App\Package;
use Illuminate\Http\Request;

class CheckoutController extends Controller
{
    public function show(Package $package, Request $request)
    {
        $subscription = Subscription::make(
            $package, 
            $request->user(), 
            '1 Month', 
            'Forever', 
            $package->price
        );

        return view('checkout')
            ->with('payable', $subscription);
    }
}
```

<br/>
<br/>

## Checkout form

Package provides **x-pay-here-checkout-form** blade component to make it easy to implemnt checkout form.

<br/>

It accepts 4 parametrs

- **payable** a payable instance
- **success-url** the url to be redirect user on successful paymnet
- **cancelled-url** the url to be redirect user on paymnet failed
- **form-class** css classes to be applied to the form tag (optional)

[Example blade code](https://gist.github.com/milroyfraser/f0526968f6fd1a6ad09bfdbf5f3d232a)

<br/>
<br/>

## Querying payable (Payment or Subscription) models by order_id

PayHere redirects the user back to the application with **order_id** GET parameter on successful or cancllled payment.

<br/>

In this controlle you nee to fetch payable instance by **order_id**. To do this package provides **findByOrderId** static methon in both Payment and Subscription models.

<br/>

```php
namespace App\Http\Controllers;

use ApiChef\PayHere\Payment;
use App\Product;
use Illuminate\Http\Request;

class CheckoutController extends Controller
{
    public function success(Request $request)
    {
        $payment = Payment::findByOrderId($request->get('order_id'));
        
        // ...
    }
}
```

<br/>
<br/>

## Quering Payment Model

<br/>

**Payment** model has **payable** relationship to get the item paid for.
```php
$item = Payment::find(10)->payable;
```

<br/>

**payer** relationship allows you to get the buyer of the item.
```php
$buyer = Payment::find(10)->payer;
```

<br/>

To fetch all the payments of a buyer, you can call **paidBy** cope.
```php
$myPayments = Payment::paidBy(Auth::user())->get();
```

<br/>

If you need to fetch all the payments made for an item, you can query **paidFor** scope. 
```php
$allPaymentsForTheBook = Payment::paidFor(Book::find(10))->get();
```

<br/>

If you need to fetch all successful payments, you can query **success** scope. 
```php
$allSuccessfulPayments = Payment::success()->get();
```

<br/>

PayHere redirects user to your website on success or cancled checkout with **order_id** GET parameter. You can use **findByOrderId** method to find the payment by **order_id**. 
```php
$payments = Payment::findByOrderId($request->get('order_id'));
```

<br/>

### You can combine the above scopes and query:
```php
$mySuccessfulPayments = Payment::paidBy(Auth::user())->success()->get();
```

<br/>
<br/>

## Quering Subscription Model

<br/>

**Subscription** model has **subscribable** relationship to get the item paid for.
```php
$plan = Subscription::find(10)->subscribable;
```

<br/>

**payer** relationship allows you to get the subscriber of the item.
```php
$subscriber = Subscription::find(10)->payer;
```

<br/>

To fetch all the subscriptions of a user, you can call **paidBy** cope.
```php
$mySubscriptions = Subscription::paidBy(Auth::user())->get();
```

<br/>

If you need to fetch all the subscriptions for an item, you can query **subscribedTo** scope. 
```php
$allSubscriptionsForGoldPlan = Subscription::subscribedTo(Plan::find(1))->get();
```

<br/>

If you need to fetch all active subscriptions, you can query **active** scope. 
```php
$allActiveSubscriptions = Subscription::active()->get();
```

<br/>

PayHere redirects user to your website on recurring payment sumition success or cancled with **order_id** GET parameter. You can use **findByOrderId** method to find the subscription by **order_id**.
```php
$subscription = Subscription::findByOrderId($request->get('order_id'));
```

<br/>

### You can combine the above scopes and query:
```php
$myActiveSubscriptions = Subscription::paidBy(Auth::user())->active()->get();
```

<br/>
<br/>

## Querying Payment Details

<br/>

[Example form implementation](https://gist.github.com/milroyfraser/f0526968f6fd1a6ad09bfdbf5f3d232a)

<br>

[← back](https://milroyfraser.github.io)
