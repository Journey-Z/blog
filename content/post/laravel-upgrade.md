---
title: "Laravel 版本升级"
date: 2020-03-16T14:56:17+08:00
categories:
- Laravel
- subcategory
tags:
- Laravel
- php
keywords:
- Laravel
- php
thumbnailImagePosition: left
thumbnailImage: img/gopher10th-small.jpg

Laravel 版本由5.2升级到6.x...
<!--more-->


# Laravel 版本升级
因为项目很久之前就开始使用了 laravel 来搭建，当时搭建之时刚好最新版本为 laravel 5.2,就直接一直沿用至今，也没有进行过 laravel 框架上的版本迭代，最近回看整个项目架构，发现很多开发扩展包已经慢慢不再去兼容淘汰掉或者太陈旧的版本，感觉如果一直因为一直规避风险而放弃去使用这些能极大提高效率的玩意，另外万一官方宣布停止了老版本的框架维护和安全的 fix，怕以后会掉进更新大黑洞里，其实后果和风险会更大，另外新的版本框架做了更多的优化和提升，我觉得也应该大胆去拥抱它，不要固守陈旧
# 不破不立
有些事情，不走出自己的舒适区的话，是很难找到自身的缺点和瓶颈的，趁着这次机会，顺便更多的熟悉一下 laravel 的源码和梳理下原有项目的架构，所以要么就不搞，要么就搞个大事情，决定把 laravel 框架从 5.2 升级到 6.0

# 说在升级之前
在升级之前先看一下 laravel 的发布路线图

| 版本 | 发布时间 | Bug 维护周期 | 安全性维护周期 |
| --- | --- | --- | --- |
| 5.0 | 2015.02.04 | 2015.08.04 | 2016.02.04 |
| [5.1(LTS)](https://learnku.com/docs/laravel/5.1)| 2015.06.09 | 2017.06.09 | 2018.06.09 |
| [5.2](https://learnku.com/docs/laravel/5.2) | 2015.12.21 | 2016.06.21 | 2016.12.21 |
| [5.3](https://learnku.com/docs/laravel/5.3) | 2016.08.23 | 2017.02.23 | 2017.08.23 |
| [5.4](https://learnku.com/docs/laravel/5.4) | 2017.01.24 | 2017.07.24 | 2018.01.24 |
| [5.5(LTS)](https://learnku.com/docs/laravel/5.5) | 2017.08.30 | 2019.08.30 | 2020.08.30 |
| [5.6](https://learnku.com/docs/laravel/5.6) | 2018.02.07 | 2018.08.07 | 2019.02.07 |
| [5.7](https://learnku.com/docs/laravel/5.7) | 2018.09.04 | 2019.02.04 | 2019.09.04 |
| [5.8](https://learnku.com/docs/laravel/5.8) | 2019.02.26 | 2019.08.26 | 2020.02.26 |
| [6.0(LTS)](https://learnku.com/docs/laravel/6.x) | 2019.09.03 | 2021.09.03 | 2022.09.03 |

从发布线路图中是可以看到是有一些小区别的，其中 5.1 和 5.5 还有 6.0 是属于 LTS 的版本

# 什么是 LTS

长期支持 （英语：Long-term support，缩写：LTS）是一种软件的产品生命周期政策，特别是开源软件，它增加了软件开发过程及软件版本周期的可靠度。长期支持延长了软件维护的周期；它也改变了软件更新（补丁）的类型及频率以降低风险、费用及软件部署的中断时间，同时提升了软件的可靠性。但这并不必然包含技术支持。

在长期支持周期的开始，软件设计师会将软件特性冻结：他们制作补丁来修复程序错误及计算机安全隐患，但不会加入新的，可能会造成软件回归的功能。软件维护者可能会单独发布补丁，或是将其置于维护版本、小数点版本或是服务包中发布。支持周期结束后，其称之为产品的生命周期结束。

“长期支持”这个术语通常是保留给特殊的软件版本，其他版本会有更短的生命周期。通常来说，长期支持版本至少会被维护两年。

Laravel LTS 版本如上表格所示，支持两年的 bug 修复和三年的安全更新支持，而一般的发行版只提供了六个月的 bug 修复支持和一年的安全修复支持，所以本次升级选择最终升级到最新的 laravel LTS 版本 6.0
# 开始升级
因为官网的升级步骤都一个版本一个升级，这里是直接升级从 5.2 到  6.0 LTS 作为一个跨度，将每个版本的逐级升级总结出来
* 打开 composer.json 文件，laravel 版本从 5.2.* 修改成 \^6.0, 另外还应该将 fideloper/proxy 依赖关系更新至 ~4.0
* 在 require-dev 部分下修改 symfony/css-selector 和 symfony/dom-crawler 到 ~4.0，修改 phpunit/phpunit 到 ~7.0，添加 "filp/whoops":"~2.0"，修改 fzaninotto/faker 到 ~1.4，修改 barryvdh/laravel-ide-helper 到 \^2.6.6，修改 barryvdh/laravel-debugbar 到 \^3.2.8，具体如下：


```
"require-dev": {
    "fzaninotto/faker": "~1.4",
    "mockery/mockery": "0.9.*",
    "phpunit/phpunit": "~7.0",
    "barryvdh/laravel-ide-helper": "^2.6.6",
    "barryvdh/laravel-debugbar": "^3.2.8",
    "symfony/css-selector": "~4.0",
    "symfony/dom-crawler": "~4.0",
    "filp/whoops": "~2.0"
  }
```

* 在 scripts 部分添加一个额外的 package 


```
"scripts": {
    ……
    "post-autoload-dump": [
      "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
      "@php artisan package:discover"
    ]
  }
```

* 执行 composer update -vvv --no-scripts
* 打开 App\Provider\EventServiceProvider.php 文件，将
以下内容

```
use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

public function boot(DispatcherContract $events)
{
    parent::boot($events);
    //
}
```
修改成

```
use Illuminate\Support\Facades\Event;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
public function boot()
{
    parent::boot();
    //
}
```
* 打开 App\Provider\RouteServiceProvider.php 文件，将以下内容


```
public function boot(Router $router)
{
  //
   parent::boot($router);
}
```
修改成

```
public function boot()
{
  parent::boot();
    //
}
```

* 打开 App\Http\Controllers\Controller.php 文件，删除 AuthorizesResources 的引入

到目前为止，laravel 升级到 6.x 的工作已经完成

# 框架升级过程中可能会踩到的坑

## 日志调整：
* 新的配置文件，所有的日志配置现在都存放在它自己的 config / logging.php 配置文件中，你应该在你自己的应用程序中放置一个默认配置文件的副本，并根据你的应用程序的需要进行调整修改，log 和 log_level 配置选项可以在 config / app.php 配置文件中删除，在 config 目录下添加 logging.php 文件


```
<?php

use Monolog\Handler\StreamHandler;

return [
    /*
    |--------------------------------------------------------------------------
    | Default Log Channel
    |--------------------------------------------------------------------------
    |
    | This option defines the default log channel that gets used when writing
    | messages to the logs. The name specified in this option should match
    | one of the channels defined in the "channels" configuration array.
    |
    */
    'default' => env('LOG_CHANNEL', 'stack'),
    /*
    |--------------------------------------------------------------------------
    | Log Channels
    |--------------------------------------------------------------------------
    |
    | Here you may configure the log channels for your application. Out of
    | the box, Laravel uses the Monolog PHP logging library. This gives
    | you a variety of powerful log handlers / formatters to utilize.
    |
    | Available Drivers: "single", "daily", "slack", "syslog",
    |                    "errorlog", "monolog",
    |                    "custom", "stack"
    |
    */
    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => ['daily'],
        ],
        'single' => [
            'driver' => 'single',
            'path' => env('LOG_PATH', storage_path('logs/laravel.log')),
            'level' => 'debug',
        ],
        'daily' => [
            'driver' => 'daily',
            'path' => env('LOG_PATH', storage_path('logs/laravel.log')),
            'level' => 'debug',
            'days' => 7,
        ],
        'slack' => [
            'driver' => 'slack',
            'url' => env('LOG_SLACK_WEBHOOK_URL'),
            'username' => 'Laravel Log',
            'emoji' => ':boom:',
            'level' => 'critical',
        ],
        'stderr' => [
            'driver' => 'monolog',
            'handler' => StreamHandler::class,
            'with' => [
                'stream' => 'php://stderr',
            ],
        ],
        'syslog' => [
            'driver' => 'syslog',
            'level' => 'debug',
        ],
        'errorlog' => [
            'driver' => 'errorlog',
            'level' => 'debug',
        ],
    ],
];
```
* Illuminate\Log\Writer 类已经被重命名为 Illuminate\Log\Logger 。 如果你明确地将此类作为你的应用中的类之一的依赖项，你应该将类引用更新为新名称。 或者， 二者选一地，你应该强烈考虑用标准化的 Psr\Log\LoggerInterface 接口来代替。

## 路由调整

* 在原来版本中的 routes.php 文件需要全部迁移到 Routes目录下，并根据原有路由的请求区分成 api.php，web.php 等路由，并将原有的路由内容拷贝到对应文件中
* Route::redirect 方法现在返回 HTTP 状态码为 302 的重定向，新增 permanentRedirect 方法实现 301 重定向。

## 缓存调整

* 为了在存储数据时允许更精细的到期时间，缓存数据的生存时间从分改为秒。 Illuminate\Cache\Repository 类和它的扩展类的 put ， putMany ， add ， remember 和 setDefaultCacheTime 方法，以及所有缓存存储器的 put 方法都完成了此更新，如果要向这些方法中的任何一个传递整数，请更新代码以确保保留在缓存中的数据传递的数值是秒，而不是分钟，例如：


```
// 原来老版本 - 存储数据30分钟
Cache::put('patpat', 'patpat', 30);

// 升级之后的版本 - 存储数据30秒
Cache::put('patpat', 'patpat', 30);

// 新版本如果要存储30分钟
Cache::put('patpat', 'patpat', 30 * 60);
```
## Eloquent 调整

* 如果你用一个自定义的中继模型定义了多对多的关系，而且这个中继模型拥有一个自增的主键，你应当确保这个自定义中继模型类中定义了一个 incrementing 属性其值为 true


```
/**
 * 标识 ID 是否自增
 *
 */
public $incrementing = true;
```

## 辅助函数调整

* 所有的 array_* and str_* 全局辅助函数 都被废弃。你需要直接使用 Illuminate\Support\Arr 和 Illuminate\Support\Str 提供的方法。


```
// 以前的老版本：
str_limit('Hello world', 2)

// 更新后的版本
Str::limit('Hello world', 2)
```
也可以在项目中添加新的 laravel/helpers 包来继续使用这些辅助函数


```
composer require laravel/helpers
```

## 授权调整

* 如有使用 authorizeResource 方法附加到控制器中的授权策略，现在应该要定义一个 viewAny 方法，当用户访问控制器中的 index 方法它就会被调用。换而言之，未经授权去访问控制器中的 index 方法将被拒绝。
* 如果你重写过 Laravel 框架中 RegisterController 的 register 或 registered 方法 ， 则应确保在对应方法中调用了 parent::register 和 parent::registered 方法，因为 Illuminate\Auth\Events\Registered 事件触发和新用户登录逻辑现在被移到 registered 方法中了，如果你重写了这些方法而没有调用对应的父级方法，则用户注册处理会失败。

## Carbon 包调整

* Carbon 1.x 不再支持，所有使用到 Carbon 都得更新为 Carbon 2.0 

## 本地化调整

* 翻译器的 Lang::trans 和 Lang::transChoice 方法已重命名为 Lang::get 和 Lang::choice，另外，如果你正在手动实现 Illuminate\Contracts\Translation\Translator 契约，你应该将实现的 trans 和 transChoice 方法更新为 get 和 choice

## 队列调整

* 在之前的 Laravel 版本 中，php artisan queue:work 命令会无限期重试队列任务，从 Laravel 6.0 开始，该命令默认只会重试队列任务一次，如果你想要强制任务无限期重试， 可以通过 --tries=0 指定:


```
php artisan queue:work --tries=0
```
另外，请确保你的数据库中包含 failed_jobs 数据表。你可以通过 Artisan 命令 queue:failed-table 来生成这个迁移:


```
php artisan queue:failed-table
```
## 请求 Facades 调整

* Input 门面是 Request 门面的复制品，已经被移除。如果你在使用 Input::get 方法，将其替换成 Request::input 方法。所有其他调用 Input 门面的地方都用需替换为 Request 门面.


```
// 原来老的版本
Input::get('patpat')

// 更新后的版本
Request::input('patpat')
或
request()->input('patpat')
```

# 其他第三方包升级
因为项目里还使用到 "torann/geoip" 获取 ip 的扩展包以及 "caffeinated/modules" 项目模块化扩展包，还有 laravel 框架下收集项目日志的 patpat/laravel-log-collection 扩展包，因此这三个包也要进行对应的版本升级

* 打开 composer.json 文件，以上两个扩展包版本分别修改成

```
"torann/geoip": "1.0.10",
"caffeinated/modules": "~6.0.0",
"patpat/laravel-log-collection": "dev-develop"
```

执行 composer update -vvv --no-scripts

* 打开 config\geoip.php 文件，将原有内容


```
<?php

return array(

	/*
	|--------------------------------------------------------------------------
	| Service
	|--------------------------------------------------------------------------
	|
	| Current only supports 'maxmind'.
	|
	*/

	'service' => 'maxmind',

	/*
	|--------------------------------------------------------------------------
	| Services settings
	|--------------------------------------------------------------------------
	|
	| Service specific settings.
	|
	*/

	'maxmind' => array(
		'type'          => env('GEOIP_DRIVER', 'database'), // database or web_service
		'user_id'       => env('GEOIP_USER_ID'),
		'license_key'   => env('GEOIP_LICENSE_KEY'),
		'database_path' => storage_path('app/geoip.mmdb'),
		'update_url'    => 'https://geolite.maxmind.com/download/geoip/database/GeoLite2-City.mmdb.gz',
		'locales'       => ['en'],
	),

	/*
	|--------------------------------------------------------------------------
	| Default Location
	|--------------------------------------------------------------------------
	|
	| Return when a location is not found.
	|
	*/

	'default_location' => array (
		"ip"           => "127.0.0.0",
		"isoCode"      => "US",
		"country"      => "United States",
		"city"         => "New Haven",
		"state"        => "CT",
		"postal_code"  => "06510",
		"lat"          => 41.31,
		"lon"          => -72.92,
		"timezone"     => "America/New_York",
		"continent"    => "NA",
	),

);

```
修改成：


```
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | Logging Configuration
    |--------------------------------------------------------------------------
    |
    | Here you may configure the log settings for when a location is not found
    | for the IP provided.
    |
    */

    'log_failures' => true,

    /*
    |--------------------------------------------------------------------------
    | Include Currency in Results
    |--------------------------------------------------------------------------
    |
    | When enabled the system will do it's best in deciding the user's currency
    | by matching their ISO code to a preset list of currencies.
    |
    */

    'include_currency' => true,

    /*
    |--------------------------------------------------------------------------
    | Default Service
    |--------------------------------------------------------------------------
    |
    | Here you may specify the default storage driver that should be used
    | by the framework.
    |
    | Supported: "maxmind_database", "maxmind_api", "ipapi"
    |
    */

    'service' => 'ipapi',

    /*
    |--------------------------------------------------------------------------
    | Storage Specific Configuration
    |--------------------------------------------------------------------------
    |
    | Here you may configure as many storage drivers as you wish.
    |
    */

    'services' => [

        'maxmind_database' => [
            'class' => \Torann\GeoIP\Services\MaxMindDatabase::class,
            'database_path' => storage_path('app/geoip.mmdb'),
            'update_url' => 'https://geolite.maxmind.com/download/geoip/database/GeoLite2-City.mmdb.gz',
            'locales' => ['en'],
        ],

        'maxmind_api' => [
            'class' => \Torann\GeoIP\Services\MaxMindWebService::class,
            'user_id' => env('MAXMIND_USER_ID'),
            'license_key' => env('MAXMIND_LICENSE_KEY'),
            'locales' => ['en'],
        ],

        'ipapi' => [
            'class' => \Torann\GeoIP\Services\IPApi::class,
            'secure' => true,
            'key' => env('IPAPI_KEY'),
            'continent_path' => storage_path('app/continents.json'),
            'lang' => 'en',
        ],

        'ipgeolocation' => [
            'class' => \Torann\GeoIP\Services\IPGeoLocation::class,
            'secure' => true,
            'key' => env('IPGEOLOCATION_KEY'),
            'continent_path' => storage_path('app/continents.json'),
            'lang' => 'en',
        ],

        'ipdata' => [
            'class'  => \Torann\GeoIP\Services\IPData::class,
            'key'    => env('IPDATA_API_KEY'),
            'secure' => true,
        ],

    ],

    /*
    |--------------------------------------------------------------------------
    | Default Cache Driver
    |--------------------------------------------------------------------------
    |
    | Here you may specify the type of caching that should be used
    | by the package.
    |
    | Options:
    |
    |  all  - All location are cached
    |  some - Cache only the requesting user
    |  none - Disable cached
    |
    */

    'cache' => 'all',

    /*
    |--------------------------------------------------------------------------
    | Cache Tags
    |--------------------------------------------------------------------------
    |
    | Cache tags are not supported when using the file or database cache
    | drivers in Laravel. This is done so that only locations can be cleared.
    |
    */

    'cache_tags' => ['torann-geoip-location'],

    /*
    |--------------------------------------------------------------------------
    | Cache Expiration
    |--------------------------------------------------------------------------
    |
    | Define how long cached location are valid.
    |
    */

    'cache_expires' => 30,

    /*
    |--------------------------------------------------------------------------
    | Default Location
    |--------------------------------------------------------------------------
    |
    | Return when a location is not found.
    |
    */

    'default_location' => [
        'ip' => '127.0.0.0',
        'iso_code' => 'US',
        'country' => 'United States',
        'city' => 'New Haven',
        'state' => 'CT',
        'state_name' => 'Connecticut',
        'postal_code' => '06510',
        'lat' => 41.31,
        'lon' => -72.92,
        'timezone' => 'America/New_York',
        'continent' => 'NA',
        'default' => true,
        'currency' => 'USD',
    ],

];

```

* 在 App\Modules 下，所有的原有 module 都需要做以下的调整，创建 Routes 文件夹，在 Routes 目录下创建 api.php 和 web.php 文件，将原来 App\Modules\Module\Http 目录下的 routes.php 文件的内容按照 web 请求和 api 的请求分别拷贝到对应的 web.php 和 api.php 文件里，此处的 Module 指的是原来 Modules 目录下原有的 Module，例如：App\Modules\Test
* 以原有项目存在的 Test 模块为例，打开 App\Modules\Test\Providers\RouteServiceProvider.php 文件，将原有文件内容


```
<?php
namespace App\Modules\Test\Providers;

use Caffeinated\Modules\Providers\RouteServiceProvider as ServiceProvider;
use Illuminate\Routing\Router;

class RouteServiceProvider extends ServiceProvider
{
	/**
	 * This namespace is applied to the controller routes in your module's routes file.
	 *
	 * In addition, it is set as the URL generator's root namespace.
	 *
	 * @var string
	 */
	protected $namespace = 'App\Modules\Test\Http\Controllers';

	/**
	 * Define your module's route model bindings, pattern filters, etc.
	 *
	 * @param  \Illuminate\Routing\Router $router
	 * @return void
	 */
	public function boot(Router $router)
	{
		parent::boot($router);

		//
	}

	/**
	 * Define the routes for the module.
	 *
	 * @param  \Illuminate\Routing\Router $router
	 * @return void
	 */
	public function map(Router $router)
	{
		$router->group(['namespace' => $this->namespace], function($router)
		{
			require (config('modules.path').'/Test/Http/routes.php');
		});
	}
}
```

修改成


```
<?php
namespace App\Modules\Test\Providers;

use Illuminate\Foundation\Support\Providers\RouteServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Route;

class RouteServiceProvider extends ServiceProvider
{
    /**
     * This namespace is applied to your controller routes.
     *
     * In addition, it is set as the URL generator's root namespace.
     *
     * @var string
     */
    protected $namespace = 'App\Modules\Test\Http\Controllers';

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        //

        parent::boot();
    }

    /**
     * Define the routes for the module.
     *
     * @return void
     */
    public function map()
    {
        $this->mapWebRoutes();

        $this->mapApiRoutes();

        //
    }

    /**
     * Define the "web" routes for the module.
     *
     * These routes all receive session state, CSRF protection, etc.
     *
     * @return void
     */
    protected function mapWebRoutes()
    {
        Route::group([
            'middleware' => 'web',
            'namespace'  => $this->namespace,
        ], function ($router) {
            require module_path('test', 'Routes/web.php');
        });
    }

    /**
     * Define the "api" routes for the module.
     *
     * These routes are typically stateless.
     *
     * @return void
     */
    protected function mapApiRoutes()
    {
        Route::group([
            'middleware' => 'api',
            'namespace'  => $this->namespace,
            'prefix'     => 'api',
        ], function ($router) {
            require module_path('test', 'Routes/api.php');
        });
    }
}
```
* 在 App\Modules\Test\Providers 目录下删除原有的 TestServiceProvider.php 文件并添加 ModuleServiceProvider.php 文件


```
namespace App\Modules\Test\Providers;

use Caffeinated\Modules\Support\ServiceProvider;

class ModuleServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap the module services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/../Resources/Lang', 'test');
        $this->loadViewsFrom(__DIR__.'/../Resources/Views', 'test');
        $this->loadMigrationsFrom(__DIR__.'/../Database/Migrations', 'test');
        $this->loadConfigsFrom(__DIR__.'/../config');
    }

    /**
     * Register the module services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->register(RouteServiceProvider::class);
    }
}
```
* 按照以上的步骤，就完成了项目内框架版本的从 5.2 到 6.x 的升级以及项目有依赖到框架版本的第三包（torann/geoip 和 caffeinated/modules 和 patpat/laravel-log-collection）升级, PS：所有用到了 caffeinated/modules 包的原有模块都要按照这个步骤来实现，要不然新版的扩展会报错以及找不到路由等等，其他更多的第三方扩展包升级这里不做赘述了
