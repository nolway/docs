# Plugins

## Introduction

A plugin allows you to add new features to your site, to
many plugins are available on our [Market](https://azuriom.com/market)
but you can create one if you can't find one that matches your
needs.

## Creating a plugin

Before creating a plugin, it is recommended that you read the
[Laravel documentation](https://laravel.com/docs/).

### Structuring a plugin

```
plugins/  <-- Folder containing the different installed plugins
|  example/  <-- Id of your plugin
|  |  plugin.json  <-- The main file of your theme containing the various information
|  |  assets/  <-- The folder containing the assets of your plugin (css, js, images, svg, etc)
|  |  database/
|  |  | migrations/ <-- The folder containing your plugin's migrations
|  |  resources/
|  |  |  lang/  <-- The folder containing the translations of your plugin
|  |  |  views/  <-- The folder containing the views of your plugin
|  |  routes/ <-- The folder containing the different routes of your plugin
|  |  src/ <-- The folder containing the sources of your plugin
```

### The plugin.json file

The `plugin.json` file is required to load a plugin, and
contains the different information of a plugin:
```json
{
    "id": "example",
    "name": "Example",
    "version": "1.0.0",
    "description": "A great plugin.",
    "url": "https://azuriom.com",
    "authors": [
        "Azuriom"
    ],
    "providers": [
        "\\Azuriom\\Plugin\\Example\\Providers\\ExampleServiceProvider",
        "\\Azuriom\\ Plugin\\Example\\Providers\\RouteServiceProvider"
    ]
}
```

#### Plugin ID

Each plugin must have an id, which must be unique and contain only numbers,
lowercase letters and dashes. It is recommended to use the name as a basis for
creating the id, for example if the name is `Hello World`, the id could be
`hello-world`. Also, the plugin's directory must have the same name as its id. 

> {info} To create a plugin you can use the following command that will
automatically generate the plugin's folder and many files by
default:
```
php artisan plugin:create <plugin name>
```

### Routes

Routes allow you to associate a URL with a particular action.

They are stored in the `routes' directory at the root of the plugin.

For more information on how routes work you can read the
[Laravel documentation](https://laravel.com/docs/7.x/routing).

Example:
```php
Route::get('/support', 'SupportController@index')->name('index');
```

> {warn} Please be careful not to use roads with closures,
as these are not compatible with some CMS optimizations.

#### Admin routes
 
For a route to be in the admin panel, just place it in the file `routes/admin.php` of the plugin.

### Views

The views are the visible part of a plugin, they are the content files HTML
of the plugin to display a page.

Azuriom using [Laravel](https://laravel.com/), views can be made using the Blade.
If you don't master Blade it is highly recommended reading
[its documentation](https://laravel.com/docs/7.x/blade), especially since it is quite short.

> {warn} It is highly recommended NOT to use PHP syntax.
when you work with Blade, because Blade does not bring you the traditional
no advantages and only disadvantages.

To display a view you can use `view('<plugin id>::<name of the view>')`,
of example `view('support::tickets.index')` to display the `tickets.index` view
of the support plugin.

To define the layout of the page, it is necessary that the view extends the view containing
the layout, you can either use the default layout (or the theme layout if there is one)
with `@extends('layouts.app')`, or create your own layout and extend it.

Then put all the main content into the `content` section,
and the title of the page in the `title` section.

```html
@extends('layouts.app')

@section('title', 'Page name')

@section('content')
    <div class="container content">
        <h1>A title</h1>

        <p>A text</p>
    </div>
@endsection
```

#### Admin view

For a page to use the admin panel layout, just use the layout
`admin.layouts.admin`, it is also recommended creating an admin folder
in the `resources` folder and put the admin views in it.

### Controllers

Controllers are a central part of a plugin, they are located in the folder
`src/Controllers` at the root of the plugin, and they take care of
to transform a request into the answer that will be sent back to the user.

For more information on how the controllers work you can read the
[Laravel documentation](https://laravel.com/docs/7.x/controllers).

example:
```php
<?php

namespace Azuriom\Plugin\Support\Controllers;

use Azuriom\Http\Controllers\Controller;
use Azuriom\Plugin\Support\Models\Ticket;

class TicketController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        // We're picking up all the tickets
        $tickets = Ticket::all();

        // We're return a view, pass him the tickets...
        return view('support::tickets.index', [
            'tickets' => $tickets,
        ]);
    }
}
```

### Models

Templates represent an entry in a database table, and allow you to interact with
the database.

You can also define in a model the different relationships of the model,
For example, a `ticket` can have a `user` and a `category`, and have `comments`.

You can find more information about models (also called Eloquent on Laravel) in the
[Laravel documentation](https://laravel.com/docs/7.x/eloquent).

```php
<?php

namespace Azuriom\Plugin\Support\Models;

use Azuriom\Models\Traits\HasTablePrefix;
use Azuriom\Models\Traits\HasUser;
use Azuriom\Models\User;
use Illuminate\Database\Eloquent\Model;

class Ticket extends Model
{
    use HasTablePrefix;
    use HasUser;

    /**
     * The table prefix associated with the model.
     *
     * @var string
     */
    protected $prefix = 'support_';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'subject', 'category_id',
    ];

    /**
     * The user key associated with this model.
     *
     * @var string
     */
    protected $userKey = 'author_id';

    /**
     * Get the user who created this ticket.
     */
    public function author()
    {
        return $this->belongsTo(User::class, 'author_id');
    }

    /**
     * Get the category of this ticket.
     */
    public function category()
    {
        return $this->belongsTo(Category::class);
    }

    /**
     * Get the comments of this ticket.
     */
    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}
```

### Service Provider

The service providers are the heart of a plugin, they are called at the initialization stage.
of Laravel, and allow to save the different parts of a plugin (views, translations, middlewares, etc).

Service providers must be added to the `providers` part of the `plugins.json`:
```json
{
    "providers": [
        "\\Azuriom\\Plugin\\Support\\Providers\\SupportServiceProvider"
    ]
}
```

You can find more information about the services provided in the
[Laravel documentation](https://laravel.com/docs/7.x/providers).

```php
<?php

namespace Azuriom\Plugin\Support\Providers;

use Azuriom\Extensions\Plugin\BasePluginServiceProvider;

class SupportServiceProvider extends BasePluginServiceProvider
{
    /**
     * Register any plugin services.
     *
     * @return void
     */
    public function register()
    {
        $this->registerMiddlewares();

        //
    }

    /**
     * Bootstrap any plugin services.
     *
     * @return void
     */
    public function boot()
    {
        // $this->registerPolicies();

        $this->loadViews();

        $this->loadTranslations();

        $this->loadMigrations();

        $this->registerRouteDescriptions();

        $this->registerAdminNavigation();

        //
    }
}
```


### Migration

Migrations allow you to create, modify or delete tables in the database.
data, they can be found in the `database/migrations` folder.

You can find more information about migrations in the
[Laravel documentation](https://laravel.com/docs/7.x/migrations).

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateSupportTicketsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('support_tickets', function (Blueprint $table) {
            $table->increments('id');
            $table->string('subject');
            $table->unsignedInteger('author_id');
            $table->unsignedInteger('category_id');
            $table->timestamp('closed_at')->nullable();
            $table->timestamps();

            $table->foreign('author_id')->references('id')->on('users');
            $table->foreign('category_id')->references('id')->on('support_categories');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('support_tickets');
    }
}
```

### Translations

Translations allow you to translate a plugin (amazing), they are found at
in the `resources/lang` directory at the root of a plugin, in the 
language folder (`en`, `fr`, etc...).

You can find more information on translations in the
[Laravel documentation](https://laravel.com/docs/7.x/localization).

To retrieve a translation you can use the
`trans('<plugin id>::<filename>.<message>')`, for example
`trans('support::messages.tickets.home')` to display the `tickets.home` message,
in the `messages.php` file of the support plugin:
```php
<?php

return [
  'tickets' => [
    'home' => 'Your tickets',
  ],
];
```

### Navigation

#### Utilisateurs

It is recommended to register the main routes of your plugin so that they can be
easily added in the navigation bar. To do this, simply call the
`$thiS->registerRouteDescriptions()` method in the plugin provider and return
the different routes in the `routeDescriptions()` method with the format 
`[<route> => <description>]`:
```php
    /**
     * Bootstrap any plugin services.
     *
     * @return void
     */
    public function boot()
    {
        // ...

        $this->registerRouteDescriptions();
    }

    /**
     * Returns the routes that should be able to be added to the navbar.
     *
     * @return array
     */
    protected function routeDescriptions()
    {
        return [
            'support.tickets.index' => 'support::messages.title',
        ];
    }
```

### Admin

To make your plugin's admin pages appear in the navbar of the admin panel,
you can register them by calling the method `$this->registerAdminNavigation()`
and returning the different routes in the `adminNavigation()` method.
```php
    /**
     * Bootstrap any plugin services.
     *
     * @return void
     */
    public function boot()
    {
        // ...

        $this->registerAdminNavigation();
    }

    /**
     * Return the admin navigations routes to register in the dashboard.
     *
     * @return array
     */
    protected function adminNavigation()
    {
        return [
            'support' => [
                'name' => 'support::admin.title', // Translation of the tab name
                'icon' => 'fas fa-question', // FontAwesome icon
                'route' => 'support.admin.tickets.index', // Page's route
                'permission' => 'support.tickets', // (Optional) Permission required to view this page
            ],
        ];
    }
```
