# Fakefactory ![Build status](https://api.travis-ci.org/skovachev/fakefactory.png)

This is a Laravel 4 package that makes the process of generating fake data models a breeze. It will enable you to quickly generate data for your tests or just to seed your application's database for UI review.

## Installation

You'll need to add the package to you `composer.json` file.

```js
"require-dev": {
    "skovachev/fakefactory": "dev-master"
}
```

Then you need to run `composer install` to download the package contents and update the autoloader.

Once it's installed you will need to register its service provider with your application. Open up `app/config/app.php` and add the following update your `providers` key.

```php
'providers' => array(
    'Skovachev\Fakefactory\FakefactoryServiceProvider',
)
```

You'll also need to update your `aliases` key as well.

```php
'aliases' => array(
    'Fakefactory' => 'Skovachev\Fakefactory\Facade',
)
```

That's it.

Please note that this packages requires [Faker](https://github.com/fzaninotto/Faker).

## Fakefactory
The Fakefactory is the main class that you'll be using. It's responsible for create the fake models for your application. When it's instructed to create a model it will query the database for any information about that model's attributes and relations to other models. Based on that and any information provided by [Faker objects](#fakers) it will create a model instance.

### Usage
Here are several examples as how you can use the Fakefactory to create model instances. 

```php
$user = Fakefactory::make('User');
```

The above example creates a `User` model instance, but without saving it to the database. To create a model and save it one go use:

```php
$user = Fakefactory::create('User');
```

### Extracting model data from the database
The Fakefactory will try to extract any data it can about your model from the database. It will extract column names and types and based on that select appriate value generation methods. It will also be able to determine any related models based on foreign keys present on the model's table. 
You can edit the default value generation rules by editing the modules configuration. You'll need run the following code to publish the configration into your config folder:

```
php artisan config:publish skovachev/fakefactory
```

### ID generation
By default the Fakefactory will not generate ID attributes for your models. This however can be changed either in the package `config.php` file or by using the following build option:

```php
$user = Fakefactory::generateId()->create('User');
```

This build option can also be used as a one time override of the configuration. If you had it enabled by default you could then disable it for a single model instance:

```php
$user = Fakefactory::generateId(false)->create('User');
```

### Override attributes
In some cases you'd want to override some attributes with specific values instead of using the fake values generated by the factory. You can do that with the `overrideAttributes` build option.

```php
$user = Fakefactory::overrideAttributes(['username' => 'foobar'])->make('User');
```

Alternatively, you can provide overrides in the make / create method as well.

```php
$user = Fakefactory::make('User', ['username' => 'foobar']);
```

### Generate relation models
In some cases you may want to generate a model along with a related model and insert both of them. This can be done with the `with` build option.

```php
$user = Fakefactory::with('posts')->make('User');
```

In the above example the factory will generate a `Post` model and attach ot to the `User` model based on their relation type.
You can also override attributes of the related model by nesting the like in the following example:

```php
$user = Fakefactory::with('posts')->make('User', [
	'username' => 'foobar',
	'posts' => [
		'title' => 'Fake title'
	]
]);
```

### Excluding attributes from generation
In some cases you may want to NOT generate values for some attributes. In that case you can use the `excludeAttributes` build option.

```php
$user = Fakefactory::excludeAttributes('username')->make('User');
```
The above code will generate a `User` model instance, but without providing a value for the `username` attribute.

## Fakers<a name='fakers'></a>
In some cases the Fakefactory will not able to extract all the information it needs to generate a good fake instance from the database. In other cases you'll want more control over how fake values are generated for a specific model attribute. Then you'll need to create a Faker object for your class.
Faker objects provide additional information on how to generate fake models that match your business domain more closely.

### Creating fakers
A simple Faker object for the `User` model may look like so:

```php

class UserFaker extends \Skovachev\Fakefactory\Faker
{
    protected $attributes = [
        'type' => ['randomElement', ['admin', 'client']]
    ];

    protected $relatedTo = ['posts'];
}
```

The above code will tell the Fakefactory class that when generating a `User` model it needs to set the `type` attribute to either *admin* or *client*. Additionally it tells it that there is relation present that does not include a foreign key on the model's table. In this case the foreign key would be on the *posts* table - for instance *user_id*.

### Assigning fakers to classes
Simply creating the class, however, will not be enough. You'll need to tell the Fakefactory to use this Faker class when it needs to generate `User` models. The following code registers the Faker object with the Fakefactory:

```php
Fakefactory::registerClassFaker('User', 'UserFaker');
```

### Generator rules
The Fakefactory package uses the awesome [Faker package](https://github.com/fzaninotto/Faker) to generate fake data.
It will extract column names and types from the database and use the appropriate generator rules for them. 
For instance if we had a column of type `Integer` the factory would use the following faker method:

```php
$value = $faker->randomNumber(1, 10000);
```

If we had a column named *phone* the `phoneNumber` method would be used.
All this can be changed in the package's `config.php` file.

### Custom rules
If you wanted to generate a custom rule for a specific attribute you'll need to have a `Faker` class with a custom method. Let's say we wanted all usernames to be in the form <random_number>_<random_string>. Our Faker class could look like so:

```php

class UserFaker extends \Skovachev\Fakefactory\Faker
{
    protected $attributes = [
        'username' => 'custom'
    ];

    public function getUsernameFakeValue($faker)
    {
        return $faker->randomNumber(1,1000) . '_' . $faker->lexify;
    }
}
```

The `$faker` argument is an instance of the `Faker\Generator` object providing all the generation of abilities of the [Faker package](https://github.com/fzaninotto/Faker).

## License

Fakefactory is released under the MIT Licence. See the bundled LICENSE file for details.

## Contributions

Please do not hesitate to send suggestions and feature requests. Let's make this package awesome!


