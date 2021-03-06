# PhlySimplePage

A ZF2 module for "static" pages.

## Overview

In most ZF2 applications, you'll have at least a few pages that are basically
static -- the controller contains no logic for the given endpoint, and it 
simply renders a template.

By default, this requires the following steps:

- Create a route
- Create a controller (if you don't have one already)
- Create an action in that controller
- Create a template

This module halves the workflow by eliminating the first three steps.

## Installation

### Source download

Grab a source download:

- https://github.com/t4web/Prototype/archive/master.zip

Unzip it in your `vendor` directory, and rename the resulting directory:

```sh
cd vendor
unzip /path/to/Prototype-master.zip
mv Prototype-master phly-simple-page
```

### Use Composer

Assuming you already have `composer.phar`, add `PhlySimplePage` to your
`composer.json` file and link on this repository:

```js
{
    "repositories": [
        {
            "type": "git",
            "url": "https://github.com/t4web/Prototype.git",
            "reference": "master"
        }
    ],
    "require": {
        "phly/phly-simple-page": "dev-master"
    }
}
```

And then install:

```sh
php composer.phar install
```

## Enable the module

Once you've installed the module, you need to enable it. You can do this by 
adding it to your `config/application.config.php` file:

```php
<?php
return array(
    'modules' => array(
        'Application',
        'PhlySimplePage',
    ),
);
```

## Usage

Create configuration in your application, mapping a route to the controller
`PhlySimplePage\Controller\Page`, and specifying a `template` key in the route
defaults.

```php
return array(
    'router' => array(
        'routes' => array(
            'proto' => array(
                'type' => 'Literal',
                'options' => array(
                    'route' => '/proto',
                    'defaults' => array(
                        'controller' => 'PhlySimplePage\Controller\Page',
                        // optionally set a specific layout and template for this page
                        'template'   => 'application/pages/about',
                        'layout'     => 'layout/some-layout',
                    ),
                ),
            ),
        ),
    ),
);
```

Then, make sure you create a template for the page. In the above example, I'd 
likely create the file in `module/Application/view/about.phtml`.

After this you may go to page `http://your.host/proto?t=about` 
or 
- `http://your.host/proto?t=about/first` - and will be sown template `module/Application/view/about/first.phtml`
- `http://your.host/proto?t=about/second` - and will be sown template `module/Application/view/about/second.phtml`
- `http://your.host/proto?t=about/second&l=layout/empty` - and will be sown template `module/Application/view/about/second.phtml`
and layout `module/Application/view/layout/empty.phtml`

## Caching

You can enable a write-through cache for all pages served by the
`PageController`. This is done via the following steps:

- Creating cache configuration
- Enabling the page cache factory

To create cache configuration, create a `phly-simple-page` configuration key in
your configuration, with a `cache` subkey, and configuration suitable for
`Zend\Cache\StorageFactory::factory`. As an example, the following would setup
filesystem caching:

```php
return array(
    'phly-simple-page' => array(
        'cache' => array(
            'adapter' => array(
                'name'   => 'filesystem',
                'options => array(
                    'namespace'       => 'pages',
                    'cache_dir'       => getcwd() . '/data/cache',
                    'dir_permission'  => 0777,
                    'file_permission' => '0666',
                ),
            ),
        ),
    ),
);
```

To enable the page cache factory, do the following:

```php
return array(
    'service_manager' => array(
        'factories' => array(
            'PhlySimplePage\PageCache' => 'PhlySimplePage\PageCacheService',
        ),
    ),
);
```

### Selectively disabling caching for given routes

If you do **not** want to cache a specific page/route, you can disable it by
adding the default key `do_not_cache` with a boolean `true` value to the route.
As an example:

```php
'about' => array(
    'type' => 'Literal',
    'options' => array(
        'route' => '/about',
        'defaults' => array(
            'controller'   => 'PhlySimplePage\Controller\Page',
            'template'     => 'application/pages/about',
            'do_not_cache' => true,
        ),
    ),
),
```

### Clearing the cache

To clear the cache for any given page, or for all pages, your cache adapter (a)
must support cache removal from the command line (APC, ZendServer, and several
other adapters do not), and (b) must support flushing if you wish to clear all
page caches at once.

The module defines two command line actions:

- `php public/index.php phlysimplepage cache clear all` -- clear all cached
  pages at once.
- `php public/index.php phlysimplepage cache clear --page=` clear a single
  cached page; use the template name you used in the routing configuration as
  the page value.

## TODO

- Ability to clear sets of pages
