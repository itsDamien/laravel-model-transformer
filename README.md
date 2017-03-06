<p align="center"><img src="https://www.dropbox.com/s/qt0uvvotr6illx0/laravel-model-transformer.png?raw=1" width="600"></p>

[![Latest Stable Version](https://poser.pugx.org/itsdamien/laravel-model-transformer/v/stable)](https://packagist.org/packages/itsdamien/laravel-model-transformer)
[![Total Downloads](https://poser.pugx.org/itsdamien/laravel-model-transformer/downloads)](https://packagist.org/packages/itsdamien/laravel-model-transformer)
[![License](https://poser.pugx.org/itsdamien/laravel-model-transformer/license)](https://packagist.org/packages/itsdamien/laravel-model-transformer)
[![Build Status](https://travis-ci.org/itsDamien/laravel-model-transformer.svg?branch=master)](https://travis-ci.org/itsDamien/laravel-model-transformer)
[![Coverage Status](https://coveralls.io/repos/github/itsDamien/laravel-model-transformer/badge.svg?branch=master)](https://coveralls.io/github/itsDamien/laravel-model-transformer?branch=master)
[![Code Climate](https://codeclimate.com/repos/58b754070eb092025f0000c7/badges/0e315aaed1faf51787ed/gpa.svg)](https://codeclimate.com/repos/58b754070eb092025f0000c7/feed)
[![StyleCI](https://styleci.io/repos/83455319/shield?branch=master&style=flat)](https://styleci.io/repos/83455319)

This package helps API developers to easily transform Eloquent models into collection that are convertible to JSON.

## Installation

Installation using composer:

```
composer require itsdamien/laravel-model-transformer
```

## Usage

Create a model transformer class by extending the `AbstractTransformer` class:

```php
class UserTransformer extends \ItsDamien\Transformer\AbstractTransformer
{
    public function model($model)
    {
        return [
            'first_name' => $model->first_name,
            'last_name'  => $model->last_name,
            'full_name'  => $model->first_name.' '.$model->last_name,
            'photos'     => PhotoTransformer::transform($model->photos),
        ];
    }
}
```

Now you can call the transformer from any controller:

```php
return response([
    "user" => UserTransformer::transform(User::find(1))
]);

// Output:
// {
//     "user":{
//         "first_name":"John",
//         "last_name":"Doe",
//         "full_name":"John Doe",
//         "photos":[]
//     }
// }
```

You can also pass a collection and the result will be an collection of transformed models:

```php
return response([
    "users" => UserTransformer::transform(User::all())
]);

// Output:
// {
//     "users":[
//         {
//             "first_name":"John",
//             "last_name":"Doe",
//             "full_name":"John Doe",
//             "photos":[]
//         },
//         {
//             "first_name":"Dolores",
//             "last_name":"Abernathy",
//             "full_name":"Dolores Abernathy",
//             "photos":[]
//         },
//     ]
// }
```

## Passing options to the transformer

You may need to pass some options from the controller to the transformer, you can do that by providing an array of options to the `transform()` method as a second parameter:

```php
UserTransformer::transform($user, ['foo' => 'bar']);
```

Now from inside the `UserTransformer` you can check the options parameter:

```php
class UserTransformer extends \ItsDamien\Transformer\AbstractTransformer
{
    public function model($model)
    {
        return [
            'first_name' => $model->first_name,
            'last_name'  => $model->last_name,
            'full_name'  => $model->first_name.' '.$model->last_name,
            'foo'        => $this->options['foo'],
        ];
    }
}
```

## Complex transformer

The default method of your transformer is named `model`, but you can add other methods by adding the method name to the `transform()` method as a third parameter:

```php
class UserTransformer extends \ItsDamien\Transformer\AbstractTransformer
{
    public function model($model)
    {
        return collect([
            'first_name' => $model->first_name,
            'last_name'  => $model->last_name,
            'full_name'  => $model->first_name.' '.$model->last_name,
        ]);
    }
    
    public function withId($model)
    {
        return collect([
            'id' => $model->id,
            'first_name' => $model->first_name,
            'last_name'  => $model->last_name,
            'full_name'  => $model->first_name.' '.$model->last_name,
        ]);
    }
    
    public function mergeModel($model)
    {
        return $this->model($model)->merge(collect([
            'id' => $model->id,
        ]));
    }
}
```

Now call the transformer:

```php
return UserTransformer::transform(User::find(1));

// Output:
// {
//     "first_name":"John",
//     "last_name":"Doe",
//     "full_name":"John Doe"
// }
```

```php
return UserTransformer::transform(User::find(1), [], 'withId');

// Output:
// {
//     "id":1,
//     "first_name":"John",
//     "last_name":"Doe",
//     "full_name":"John Doe"
// }
```

```php
return UserTransformer::transform(User::find(1), [], 'mergeModel');

// Output:
// {
//     "id":1,
//     "first_name":"John",
//     "last_name":"Doe",
//     "full_name":"John Doe"
// }
```
