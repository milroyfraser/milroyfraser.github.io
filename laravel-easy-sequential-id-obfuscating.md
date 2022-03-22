# Laravel easy sequential ID obfuscation


[← back](https://milroyfraser.github.io)

<br>

When you use sequential IDs / auto-increment IDs in URLs, you are giving away some sensitive information about your application.

Which allows visitors to:
- Estimated content counts e.g. products count, sellers count.
- Parameter injection by incrementing ID in URL e.g. /dislike-post/{10}
- Automate scanning through your application by a simple script. 

Read more [Auto-Incrementing IDs: Giving your Data Away](https://phil.tech/2015/auto-incrementing-to-destruction/)

<br/>

#### Why this package

Yes, there are many solutions available. This package uses [jenssegers/optimus](https://github.com/jenssegers/optimus) for encoding and decoding and it is **super fast**. And the package provides everything you need:

- Seamless route model binding
- Validation rule class
- Helpers for encoding and decoding

<br/>

#### Install via composer

``` bash
$ composer require apichef/laravel-obfuscate
```

#### Usage

The only thing you have to do is using the the `Obfuscatable` trait in you model class.

```php
namespace App;

use Illuminate\Database\Eloquent\Model;
use ApiChef\Obfuscate\Obfuscatable;

class Post extends Model
{
    use Obfuscatable;

    // ...
}
```

#### Route model bilding

```php
Route::get('/posts/{post}', function (Post $post) {
    return $post;
})->name('post.show');
```

#### Generate the URL to a named route.

```php
$post = Post::find(1);

echo(route('post.show', $post));

// https://my-app.test/api/posts/458047115
```

#### Validation

Sometimes you need to validate whether a record is available in the database for  the given ID. For that package provides `HashExists` rule class. 
 
```php
namespace App\Http\Requests;

use ApiChef\Obfuscate\Rules\HashExists;
use Illuminate\Foundation\Http\FormRequest;

class PostStoreRequest extends FormRequest
{
    // ...
    public function rules()
    {
        return [
            'post_id' => [
                'required',
                new HashExists('posts', 'id')
            ],
        ];
    }
}
```

#### Facade

Sometime you might need to encode an auto-increment ID and vice versa. For that you can use `Obfuscate` facade.

```php
use ApiChef\Obfuscate\Support\Facades\Obfuscate;

$result = Obfuscate::encode(1);
// 458047115

$result = Obfuscate::encode([1, 2]);
// [458047115, 2033899500]

$result = Obfuscate::decode('458047115');
// 1

$result = Obfuscate::decode([458047115, 2033899500]);
// [1, 2]
```

<br>

[← back](https://milroyfraser.github.io)
