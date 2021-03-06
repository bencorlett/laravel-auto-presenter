# Laravel Auto Presenter 3

[![Build Status](https://travis-ci.org/ShawnMcCool/laravel-auto-presenter.svg?branch=master)](https://travis-ci.org/ShawnMcCool/laravel-auto-presenter)

This package automatically decorates objects bound to views during the view render process.

## Version 3 Beta Concerns

The purpose of the 3.0 major version was to drop a lot of the functionality of earlier versions. Then, incrementally we'll solve each problem as they come up. All-in-all we believe that the solutions for most presentation concerns are very rudimentary. If new concerns are raised, we'd like a chance to architect a solution that supports a solid separation of concerns.

- Currently, we're trying to decide if there's a reason at all to implement ArrayAccess.

Please chime in on the issues if you have a strong opinions.

## Upgrading From Version 2 To 3

First of all, we recommend not doing this until it's out of beta.

* PHP 5.5 is now a requirement for version 3

## Upgrading From Version 1 To 2

* Version 2 is now Laravel 4.2 only.
* The HasPresenter was added as the method for determining the correct presenter class. Read more on this in the instructions below.

<a name="features"/>
## Features

- Automatically decorate objects bound to views
- Automatically decorate objects within paginator instances
- Automatically decorate objects within collection objects

<a name="install-laravel-package-installer"/>
## Installing With The Laravel 4 Package Installer

1. Install the [Laravel 4 Package Installer](https://github.com/rtablada/package-installer)
2. Run `php artisan package:install mccool/laravel-auto-presenter`

<a name="requirements"/>
## Requirements

Beta Version 3

- PHP 5.5+
- Laravel 4.2.x
- [optional] PHPUnit to run the tests

Version 2

- PHP 5.4+
- Laravel 4.2.x
- [optional] PHPUnit to run the tests

Version 1

- PHP 5.3+
- Laravel 4.1.x
- [optional] PHPUnit to run the tests

<a name="install-composer"/>
## Installing With Composer

Install this package with [Composer](http://getcomposer.org/).

Add the following "require" to your `composer.json` file and run the `composer install` command to install it.

**For Laravel 4.1**

```json
{
    "require": {
        "mccool/laravel-auto-presenter": "~1.2"
    }
}
```

**For Laravel 4.2 (Stable)**

```json
{
    "require": {
        "mccool/laravel-auto-presenter": "~2.2"
    }
}
```

**For Laravel 4.2 (Beta)**

```json
{
    "require": {
        "mccool/laravel-auto-presenter": "~3.0@beta"
    }
}
```

Then, in config/app.php add this line to your 'providers' array.

```php
'McCool\LaravelAutoPresenter\LaravelAutoPresenterServiceProvider',
```

## Usage

To show how it's used, we'll pretend that we have an Eloquent Post model. It doesn't have to be Eloquent, it could be any kind of class. But, this is a normal situation. The Post model represents a blog post.

I'm using really basic code examples here, so just focus on how the auto-presenter is used and ignore the rest.

```php
<?php namespace Example\Blog;

use Example\Accounts\User;

class Post extends \Eloquent
{
    protected $table = 'posts';
    protected $fillable = array('author_id', 'title', 'content', 'published_at');

    public function author()
    {
        return $this->belongsTo(User::class, 'author_id');
    }
}
```

Also, we'll need a controller..

```php
class PostsController extends \Controller
{
    public function getIndex()
    {
        $posts = Post::all();
        return View::make('posts.index', compact('posts'));
    }
}
```

and a view...

```twig
@foreach($posts as $post)
    <li>{{ $post->title }} - {{ $post->published_at }}</li>
@endforeach
```

In this example the published_at attribute is likely to be in the format: "Y-m-d H:i:s" or "2013-08-10 10:20:13". In the real world this is not what we want in our view. So, let's make a presenter that lets us change how the data from the Post class is rendered within the view.

```php
<?php namespace Example\Blog;

use McCool\LaravelAutoPresenter\BasePresenter;

class PostPresenter extends BasePresenter
{
    public function __construct(Post $post)
    {
        $this->wrappedObject = $post;
    }

    public function published_at()
    {
        return Carbon::createFromFormat('Y-m-d H:i:s', $this->wrappedObject->published_at, 'Europe/Berlin')
            ->toFormattedDateString();
    }
}
```

Here, the automatic presenter decorator is injecting the Post model that is to be decorated. We need the post class to implement the interface.

```php
<?php namespace Example\Blog;

use McCool\LaravelAutoPresenter\HasPresenter;
use Example\Accounts\User;

class Post extends \Eloquent implements HasPresenter
{
    protected $table = 'posts';
    protected $fillable = array('author_id', 'title', 'content', 'published_at');

    public function author()
    {
        return $this->belongsTo(User::class, 'author_id');
    }

    public function getPresenterClass()
    {
        return \Example\Blog\PostPresenter::class;
    }
}
```

Now, with no additional changes our view will show the date in the desired format.

## Troubleshooting

If an object isn't being decorated correctly in the view then there's a good chance that it's simply not in existence when the view begins to render. For example, lazily-loaded relationships won't be decorated. You can fix this by eager-loading them instead. Auth::user() will never be decorated. I prefer to bind $currentUser to my views, anyway.

## License

MIT
