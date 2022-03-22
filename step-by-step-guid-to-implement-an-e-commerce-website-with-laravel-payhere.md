# How to build an E-commerce website with Laravel PayHere

[← back](https://milroyfraser.github.io)

<br>

ApiChef Laravel PayHere package provides more features than what is explain in the guide, such as easy subscriptions API, payment details retrieval. You can find the full package [documentation here](https://milroy.me/laravel-pay-here)

<br/>

- [Watch the screencast of this demo](https://www.youtube.com/playlist?list=PLtX53M2iMi4_riAi4ZBy8i23RzpBnTzgz)
- [GitHub](https://github.com/apichef/laravel-pay-here)

<br/>

I have done few things befe I start this guide.

- craete a laravel project
```bash
$ composer create-project laravel/laravel sri-commerce
```
- Install larave tailwindcss preset
```bash
$ composer require laravel-frontend-presets/tailwindcss --dev
```
- Generate UI scaffolding login & registration
```bash
$ php artisan ui tailwindcss --auth
```
- Add Nunito font to tailwind config
```js
module.exports = {
  // ...
  theme: {
      // ...
      fontFamily: {
          serif: ['Nunito', 'sans-serif'],
      }
  },
  // ...
}
```
- Link Nunito font in the heder section of `resources/views/layouts/app.blade.php`
```
<!-- Font -->
<link href="https://fonts.googleapis.com/css2?family=Nunito:ital,wght@0,200;0,300;0,400;0,600;0,700;0,800;0,900;1,200;1,300;1,400;1,600;1,700;1,800;1,900&display=swap" rel="stylesheet">
```

<br/>

## OKii. We are ready to get started.

<br/>

#### Step 1. Installation

```bash
$ composer require apichef/laravel-pay-here
```

#### Step 2. Publish migrations and config
```bash
$ php artisan vendor:publish --provider="ApiChef\PayHere\PayHereServiceProvider"
```

#### Step 3. Run migrations
```bash
$ php artisan migrate
```

#### Step 4. Configur credentials

You need to set the following envirnoment variables:
```
PAY_HERE_MERCHANT_ID=
PAY_HERE_MERCHANT_SECRET=

PAY_HERE_BUSINESS_APP_ID=
PAY_HERE_BUSINESS_APP_SECRET=
```

Merchant ID is issued per account by PayHere, You can copy it from your PayHere settings page.

<br/>

To obtain a merchant secret, you need to add a domain to allowed domains/apps under the Domains & Credentials tab

<br/>

You must create a business application to consume other PayHere services. Go to the Business Apps tab and create app.

<br/>

**NOTE: Tick the following permission**
- Payment Retrieval API
- Subscription Management API (only if you use subscriptions)

<br/>

#### Step 5. Implement checkout and redirect route

```php
Route::get('/checkout/success', 'CheckoutController@success')
    ->name('checkout.success');

Route::get('/checkout/cancelled', 'CheckoutController@cancelled')
    ->name('checkout.cancelled');

Route::get('/checkout/{product}', 'CheckoutController@show')
    ->name('checkout');
```

<br/>

#### Step 6. Implement checkout controller

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
            ->with('payment', $payment);
    }

    public function success(Request $request)
    {
        $payment = Payment::findByOrderId($request->get('order_id'));
        
        /** @var Product $product */
        $product = $payment->payable;
        
        // perform the side effects of successful payment
        
        // redirect to payment success page
    }

    public function cancelled(Request $request)
    {
        $payment = Payment::findByOrderId($request->get('order_id'));
        
        /** @var Product $product */
        $product = $payment->payable;

        // perform the side effects of cancelled payment

        // redirect to payment cancelled page
    }
}
```

<br/>

#### Step 7. Implement checkout form

[Example form tag](https://gist.github.com/milroyfraser/f0526968f6fd1a6ad09bfdbf5f3d232a)

<br/>

#### Step 8. Query purchased items.

You can query the purchased items from the Payment model using **paidBy** and **success** scopes.

```php
$products = Payment::query()
    ->with('payable')
    ->paidBy(Auth::user())
    ->success()
    ->get()
    ->map->payable;
```

<br/>

You can find the source code of this example in this [repository](https://github.com/techleadlk/sri-commerce).

<br>

[← back](https://milroyfraser.github.io)
