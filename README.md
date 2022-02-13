# Generate slugs when saving Eloquent models | Support Arabic

This package provides a trait that will generate a unique slug when saving any Eloquent model

Froked from [spatie/laravel-sluggable](https://github.com/spatie/laravel-sluggable) to support Arabic language.

```php
$model = new EloquentModel();
$model->name = 'activerecord is awesome';
$model->save();

echo $model->slug; // outputs "activerecord-is-awesome"
```

The slugs are generated with Laravels `Str::slug` method, whereby spaces are converted to '-'.



## Installation

You can install the package via composer:
``` bash
composer require salah3id/laravel-slugs
```

## Usage

Your Eloquent models should use the `Salah3id\Sluggable\HasSlug` trait and the `Salah3id\Sluggable\SlugOptions` class.

The trait contains an abstract method `getSlugOptions()` that you must implement yourself.

Your models' migrations should have a field to save the generated slug to.

Here's an example of how to implement the trait:

```php
namespace App;

use Salah3id\Sluggable\HasSlug;
use Salah3id\Sluggable\SlugOptions;
use Illuminate\Database\Eloquent\Model;

class YourEloquentModel extends Model
{
    use HasSlug;

    /**
     * Get the options for generating the slug.
     */
    public function getSlugOptions() : SlugOptions
    {
        return SlugOptions::create()
            ->generateSlugsFrom('name')
            ->saveSlugsTo('slug');
    }
}
```

With its migration:

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateYourEloquentModelTable extends Migration
{
    public function up()
    {
        Schema::create('your_eloquent_models', function (Blueprint $table) {
            $table->increments('id');
            $table->string('slug'); // Field name same as your `saveSlugsTo`
            $table->string('name');
            $table->timestamps();
        });
    }
}

```

### Using slugs in routes

To use the generated slug in routes, remember to use Laravel's [implicit route model binding](https://laravel.com/docs/5.8/routing#implicit-binding):

```php
namespace App;

use Salah3id\Sluggable\HasSlug;
use Salah3id\Sluggable\SlugOptions;
use Illuminate\Database\Eloquent\Model;

class YourEloquentModel extends Model
{
    use HasSlug;

    /**
     * Get the options for generating the slug.
     */
    public function getSlugOptions() : SlugOptions
    {
        return SlugOptions::create()
            ->generateSlugsFrom('name')
            ->saveSlugsTo('slug');
    }

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }
}
```

### Using multiple fields to create the slug

Want to use multiple field as the basis for a slug? No problem!

```php
public function getSlugOptions() : SlugOptions
{
    return SlugOptions::create()
        ->generateSlugsFrom(['first_name', 'last_name'])
        ->saveSlugsTo('slug');
}
```

### Customizing slug generation

You can also pass a `callable` to `generateSlugsFrom`.

By default the package will generate unique slugs by appending '-' and a number, to a slug that already exists.

You can disable this behaviour by calling `allowDuplicateSlugs`.

```php
public function getSlugOptions() : SlugOptions
{
    return SlugOptions::create()
        ->generateSlugsFrom('name')
        ->saveSlugsTo('slug')
        ->allowDuplicateSlugs();
}
```

### Limiting the length of a slug

You can also put a maximum size limit on the created slug:

```php
public function getSlugOptions() : SlugOptions
{
    return SlugOptions::create()
        ->generateSlugsFrom('name')
        ->saveSlugsTo('slug')
        ->slugsShouldBeNoLongerThan(50);
}
```

The slug may be slightly longer than the value specified, due to the suffix which is added to make it unique.

You can also use a custom separator by calling `usingSeparator`

```php
public function getSlugOptions() : SlugOptions
{
    return SlugOptions::create()
        ->generateSlugsFrom('name')
        ->saveSlugsTo('slug')
        ->usingSeparator('_');
}
```

### Setting the slug language

To set the language used by `Str::slug` you may call `usingLanguage`

```php
public function getSlugOptions() : SlugOptions
{
    return SlugOptions::create()
        ->generateSlugsFrom('name')
        ->saveSlugsTo('slug')
        ->usingLanguage('nl');
}
```

### Using slugs in Arabic language

Because Laravels `Str::slug` method does not support Arabic letters, So You can override
the generated slug from `Str::slug` by calling `arabicable` to generate slugs in Arabic !

```php
namespace App;

use Salah3id\Sluggable\HasSlug;
use Salah3id\Sluggable\SlugOptions;
use Illuminate\Database\Eloquent\Model;

class YourEloquentModel extends Model
{
    use HasSlug;

    /**
     * Get the options for generating the slug.
     */
    public function getSlugOptions() : SlugOptions
    {
        return SlugOptions::create()
            ->generateSlugsFrom('name')
            ->saveSlugsTo('slug')
            ->arabicable();
    }
}
```

### Overriding slugs

You can also override the generated slug just by setting it to another value than the generated slug.

```php
$model = EloquentModel::create(['name' => 'my name']); //slug is now "my-name";
$model->slug = 'my-custom-url';
$model->save(); //slug is now "my-custom-url";
```

### Prevent slugs from being generated on creation

If you don't want to create the slug when the model is initially created you can set use the `doNotGenerateSlugsOnCreate()` function.

```php
public function getSlugOptions() : SlugOptions
{
    return SlugOptions::create()
        ->generateSlugsFrom('name')
        ->saveSlugsTo('slug')
        ->doNotGenerateSlugsOnCreate();
}
```

### Prevent slug updates

Similarly, if you want to prevent the slug from being updated on model updates, call `doNotGenerateSlugsOnUpdate()`.

```php
public function getSlugOptions() : SlugOptions
{
    return SlugOptions::create()
        ->generateSlugsFrom('name')
        ->saveSlugsTo('slug')
        ->doNotGenerateSlugsOnUpdate();
}
```

This can be helpful for creating permalinks that don't change until you explicitly want it to.

```php
$model = EloquentModel::create(['name' => 'my name']); //slug is now "my-name";
$model->save();

$model->name = 'changed name';
$model->save(); //slug stays "my-name"
```

### Regenerating slugs

If you want to explicitly update the slug on the model you can call `generateSlug()` on your model at any time to make the slug according to your other options. Don't forget to `save()` the model to persist the update to your database.

### Preventing overwrites

You can prevent slugs from being overwritten.

```php
public function getSlugOptions() : SlugOptions
{
    return SlugOptions::create()
        ->generateSlugsFrom('name')
        ->saveSlugsTo('slug')
        ->preventOverwrite();
}
```

### Using scopes


If you have a global scope that should be taken into account, you can define this as well with `extraScope`. For example if you have a pages table containing pages of multiple websites and every website has it's own unique slugs.

```php
public function getSlugOptions() : SlugOptions
{
    return SlugOptions::create()
        ->generateSlugsFrom('name')
        ->saveSlugsTo('slug')
        ->extraScope(fn ($builder) => $builder->where('scope_id', $this->scope_id));
}
```

### Integration with laravel-translatable

You can use this package along with [laravel-translatable](https://github.com/spatie/laravel-translatable) to generate a slug for each locale. Instead of using the `HasSlug` trait, you must use the `HasTranslatableSlug` trait, and add the name of the slug field to the `$translatable` array. For slugs that are generated from a single field _or_ multiple fields, you don't have to change anything else.

```php
namespace App;

use Salah3id\Sluggable\HasTranslatableSlug;
use Salah3id\Sluggable\SlugOptions;
use Salah3id\Translatable\HasTranslations;
use Illuminate\Database\Eloquent\Model;

class YourEloquentModel extends Model
{
    use HasTranslations, HasTranslatableSlug;

    public $translatable = ['name', 'slug'];

    /**
     * Get the options for generating the slug.
     */
    public function getSlugOptions() : SlugOptions
    {
        return SlugOptions::create()
            ->generateSlugsFrom('name')
            ->saveSlugsTo('slug');
    }
}
```

For slugs that are generated from a callable, you need to instantiate the `SlugOptions` with the `createWithLocales` method. The callable now takes two arguments instead of one. Both the `$model` and the `$locale` are available to generate a slug from.

```php
namespace App;

use Salah3id\Sluggable\HasTranslatableSlug;
use Salah3id\Sluggable\SlugOptions;
use Salah3id\Translatable\HasTranslations;
use Illuminate\Database\Eloquent\Model;

class YourEloquentModel extends Model
{
    use HasTranslations, HasTranslatableSlug;

    public $translatable = ['name', 'slug'];

    /**
     * Get the options for generating the slug.
     */
    public function getSlugOptions() : SlugOptions
    {
        return SlugOptions::createWithLocales(['en', 'nl'])
            ->generateSlugsFrom(function($model, $locale) {
                return "{$locale} {$model->id}";
            })
            ->saveSlugsTo('slug');
    }
}

```

#### Implicit route model binding

You can also use Laravels [implicit route model binding](https://laravel.com/docs/8.x/routing#implicit-binding) inside your controller to automatically resolve the model. To use this feature, make sure that the slug column matches the `routeNameKey`.  
Currently, only some database types support JSON opterations. Further information about which databases support JSON can be found in the [Laravel docs](https://laravel.com/docs/8.x/queries#json-where-clauses).

```php
namespace App;

use Salah3id\Sluggable\HasSlug;
use Salah3id\Sluggable\SlugOptions;
use Illuminate\Database\Eloquent\Model;

class YourEloquentModel extends Model
{
    use HasTranslations, HasTranslatableSlug;

    public $translatable = ['name', 'slug'];

    /**
     * Get the options for generating the slug.
     */
    public function getSlugOptions() : SlugOptions
    {
        return SlugOptions::create()
            ->generateSlugsFrom('name')
            ->saveSlugsTo('slug');
    }

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }
}
```


## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information what has changed recently.

## Testing

``` bash
composer test
```

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Security

If you discover any security related issues, please email salah.enm@gmail.com instead of using the issue tracker.

## Credits

- [Salah Eid](https://github.com/salah3id)
- [Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
