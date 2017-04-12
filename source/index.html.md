---
title: Documentation

language_tabs:
  - php

search: true
---

# Introduction

Welcome to the WP Blade documentation. WP Blade is a simple wrapper around Laravel's excellent Blade.

Blade is ment for theme developers rather than plugin developers, as it ties in with the template engine of WordPress.

# Installing

> Simply add this somewhere in your theme functions, or as a `mu-plugin`

```php
<?php
$blade = TorMorten\View\Blade::create();
```

> Make sure your theme is ready for WP Blade by adding theme support for `blade-templates`

```php
<?php 
add_action('after_theme_setup', function() {
  add_theme_support('blade-templates');
});
```

WP Blade is, for now, only availiable as a composer package. If you have not used composer before, you should really check it out.

Install it by adding `"tormjens/wp-blade" : "dev-5.4-dev"` to your `composer.json`.

# Controllers

> First create your controller and include it somewhere before you load it. This will attach the controller to every single view you specify in the `$views` array.

```php 
<?php 

class BooksController extends TorMorten\View\Controller {

  protected $views = ['books', 'library'];

  public function process() {
    $books = get_posts('post_type=books');
    return compact('books');
  }

}
```

> Once you have included your controller somewhere you may initialize it, either via the `$blade` variable from the previous example.

```php
<?php
$blade->addController('BooksController');
```

> Or via the `instance` method on the class.

```php 
<?php
TorMorten\View\Blade::instance()->addController('BooksController');
```

One of the main features of WP Blade is the ability to move the logic for your view out to controllers. Controllers are tied to one or multiple views, and only return data.

> That's it. You can now access the data your returned in your `process()` method as variables in your Blade template.

```php
@if($books)
  @foreach($books as $book)
    <p>I like the book {{$book->post_title}}</p>
  @endforeach
@endif
```

# Directives

WP Blade comes with a set of custom directives to ease your development workflow and make your templates look cleaner.

> Define a variable

```php
@var('foo', 'bar')
{{$foo}}
```

> The loop

```php 
@wpposts
  <h1>{{get_the_title()}}</h1>
@wpempty
  No posts here.
@wpend
```

> WP Query

```php
@wpquery('post_type=books')
  <p>I like the book {{get_the_title()}}</p>
@wpempty
  No posts here.
@wpend
```

> Advanced Custom Fields

```php
# Repeater / flexible content
@acf('authors')
  <p>The author name is {{get_sub_field('name')}}</p>
@acfempty
  No authors added
@acfend

# Check if a field value exists
@acfhas('authors')
  Yes, I have authors.
@acfend

# Output a field (if it exists)
@acffield('sub_title')

# Output a sub field (if it exists)
@acfsub('name')
```

# Changing the defaults

WP Blade operates with a couple of default paths. You can easily change these.

The defaults are:

Variables | Default | Description |
--------- | ------- | ----------- |
`BLADE_VIEWS` | {template_directory}/views/ | The directory that will be searched for partials when you use the `@include('view')` directive.
`BLADE_CACHE` | {template_directory}/views/ | The directory that contains all compiled Blade templates.

> The defaults can be set before you intialize Blade. Or even in `wp-config.php`

```php
<?php
define('BLADE_VIEWS', '/path/to/my/views');
define('BLADE_CACHE', '/path/to/my/cache');
```

> You can also change these using a series of filters.

```php 
<?php
add_filter('wp_blade_views_directory', function($path) {
  return '/path/to/my/views';
});

add_filter('wp_blade_cache_directory', function($path) {
  return '/path/to/my/cache';
});
```

# Filters and actions

A series of filters and actions are also availiable if you need to hook in to any point in the code.

## Actions

Action | Description |
--------- | ----------- |
`wp_blade_booting` | Runs at the very start of the initialization process.
`wp_blade_booted` | Runs when the initialization is completed.
`wp_blade_add_directive` | Allows adding more custom directives.

# Extending

WP Blade can easily be extended with directives that you might need.

> Somewhere in your code, call the `wp_blade_add_directive` with your new directive.

```php 
<?php 
add_action('wp_blade_add_directive', function($compiler) {
  $compiler->directive('foo', function() {
    return "<?php $foo = 'bar'; echo $foo; ?>";
  });
  $compiler->directive('meta', function($expression) {
    return "<?php $meta = get_post_meta{$expression}; echo $meta; ?>";
  });
});
```

> You can now use these directives in your Blade template

```php 
# the foo directive
@foo
# the meta directive
@meta(get_the_ID(), '_meta_key', true)
```
