## Laravel Artisan Cache Commands Explained

Often times, when you are in the middle of developing a Laravel application, you may find that the changes you made in your code are not reflecting well on the application when testing.

Usually, the case is most likely caused by caching applied by the Laravel framework.

Here are some of the common commands you can run in your terminal to alleviate the issue.

>❗️ Make sure you are running them in the context of your application. Meaning, your terminal is currently in the same directory as your Laravel application.

## 1. Configuration Cache

Caching configuration helps with combining all of the configuration options for your application into a single file which will be loaded quickly by the framework.

### Clearing Configuration Cache

However, if you notice changes to the configuration values in `.env` file is not reflecting on your application, you may want to consider clearing the configuration cache with the following command:

```
$ php artisan config:clear
Configuration cache cleared!
```

If you want to quickly reset your configuration cache after clearing them, you may instead run the following command:

```
$ php artisan config:cache
Configuration cache cleared!
Configuration cached successfully!
```

Caching your configuration will also help clear the current configuration cache. So it helps save your time without having to run both commands.

## 2. Route Caching

Caching your routes will drastically decrease the amount of time it takes to register all of your application's routes. When you add a new route, you will have to clear your route cache for the new route to take effect.

### Clearing Route Cache

The following command will clear all route cache in your application:

```
$ php artisan route:clear
Route cache cleared!
```

To cache your routes again, simply run the following command:

```
$ php artisan route:cache
Route cache cleared!
Routes cached successfully!
```

Again, running the above command alone is enough to clear your previous route cache and rebuild a new one.

## 3. Views Caching

Views are cached into compiled views to increase performance when a request is made. By default, Laravel will determine if the uncompiled view has been modified more recently than the compiled view, before deciding if it should recompile the view.

### Clearing View Cache

However, if for some reason your views are not reflecting recent changes, you may run the following command to clear all compiled views cache:

```
$ php artisan view:clear
Compiled views cleared!
```

In addition, Laravel also provides an Artisan command to precompile all of the views utilized by your application. Similarly, the command also clears the view cache before recompiling a new set of views:

```
$ php artisan view:cache
Compiled views cleared!
Blade templates cached successfully!
```

## 4. Events Cache

If you are using Events in your Laravel application, it is recommended to cache your Events, as you likely do not want the framework to scan all of your listeners on every request.

### Clearing Events Cache

When you want to clear your cached Events, you may run the following Artisan command:

```
$ php artisan event:clear
Cached events cleared!
```

Likewise, caching your Events also clear any existing cache in the framework before a new cache is rebuilt:

```
$ php artisan event:cache
Cached events cleared!
Events cached successfully!
```

## 5. Application Cache

Using Laravel's Cache is a great way to speed up frequently accessed data in your application. While developing your application involving cache, it is important to know how to flush all cache correctly to test if your cache is working properly.

### Clearing Application Cache

To clear your application cache, you may run the following Artisan command:

```
$ php artisan cache:clear
Application cache cleared!
```

This will clear all the cache data in storage which are typically stored in `/storage/framework/cache/data/`. The effect is similar to calling the `Cache::flush();` Facade method via code.

>❗️ This command will NOT clear any **config**, **route**, or **view** cache, which are stored in `/bootstrap/cache/` directory.

## 6. Clearing All Cache

Laravel provides a handy Artisan command that helps clear *ALL* the above caches that we have covered above. It is a convenient way to reset all cache in your application, without having to run multiple commands introduced before.

To clear all Laravel's cache, just run the following command:

```
$ php artisan optimize:clear
Compiled views cleared!
Application cache cleared!
Route cache cleared!
Configuration cache cleared!
Compiled services and packages files removed!
Caches cleared successfully!
```

As you can read from the terminal feedback, all cache types that existed in your Laravel application will be cleared entirely, except Events cache.

>❗️ The `Compiled services and packages files removed!` can be run individually via `$ php artisan clear-compiled` command. It is used to remove the compiled class file in the framework.

## 🎁 Bonus

If the above Laravel's Artisan commands don't seem to resolve the issue you are facing, you may need to look at other related environments in your project that may be causing it.

When building a Laravel project, it is common to employ the Composer Dependency Manager for PHP, as well as NPM for any JavaScript library that might be needed in your project. We just have to take note that both package managers are using some form of caching for performance improvements.

### Clearing Composer Cache

Sometimes, a new package you just installed via Composer doesn't appear to be working at all. Or a new project you just cloned from a repository doesn't seem to be running correctly.

Such issues are usually caused by classmap error from a newly installed library class, or the cached version of a particular library does not match with the ones required by the project codebase you just cloned. In such a situation, you need to update the PHP autoloader by running the following command:

```
$ composer dump-autoload
```

As well as any of the following variations(they all achieve the same purpose of deleting all content from Composer's cache directories):

```
$ composer clear-cache
$ composer clearcache
$ composer cc
```

### Clearing NPM Cache

To clear NPM cache:

```
$ npm cache clean --force
```

## 🏁 Summary

That's all you need to know about Laravel's Artisan cache related commands, as well as few additional commands for Composer and NPM, which you most likely will be using together in a Laravel project.

Here's a final command before we end the article:

```
$ php artisan list
```

The above command will list down all available Artisan commands when executed. I recommend you to take a look at it, as you might find something new that will be useful to you!

❗️ Thank you for reading! If you find this useful, feel free to share it with your followers. 🦊

## 📚 References

- https://laravel.com/docs/6.x/configuration#configuration-caching
- https://laravel.com/docs/8.x/controllers#route-caching
- https://laravel.com/docs/8.x/views#optimizing-views
- https://laravel.com/docs/8.x/events#event-discovery
- https://laravel.com/docs/8.x/cache#removing-items-from-the-cache
- https://getcomposer.org/doc/03-cli.md#dump-autoload-dumpautoload-
- https://getcomposer.org/doc/03-cli.md#clear-cache-clearcache-cc
- https://docs.npmjs.com/cli/v6/commands/npm-cache