Moving modules to a different location without disabling it is a quite tricky process in Drupal 8. There's a lot of discussion about it but I coudn't find a proper solution so I've decided to digg a bit. In Drupal 7 it was easy, just put a module to new location, clear the cache and you're done, but D8 might be different. So, this is the quick step through without answering why you need to do this or that.

Check which if you have and if `apcu` extension is installed and enabled. You should do it by visiting `/admin/reports/status/php`. Keep in mind there's no point to check it through the console as it might use different php.ini file. This needs to be done through browser.

<img src="http://i.imgur.com/MenhNok.png" alt="" />

With APCU extension enabled.

1. Move your module code to desired location.
2. Clear drupal cache.
3. Call `apcu_clear_cache()` function, but it has to be triggered from browser. There's no way it'll work by calling it from console or drush, so keep in mind that `drush ev "apcu_clear_cache();"` will not work.
5. That's it, now your module should be available to use and it should not complaining by missing PHP classes or plugins.

Without APCU extension enabled or installed.

1. Move your module to different location
2. Clear drupal cache.
3. Done

Let's digg a bit and find the details, so we have some idea what's happning here. 

In most answers you will find out that cleaning drupal cache or providing extra `class_loader_auto_detect` setting to FALSE will work for you. Sometimes it works, sometimes not. If you put your `class_loader_auto_detect` setting to FALSE it'll work but you will loose class caching mechanism. Is it really a good idea? Well, no. If it is an option and it's ready to use why would you? There's no point doing it. Instead let's use it. So, let's assume that you do have `APCU` extension enabled. While drupal 8 is booting DrupalKernel.php boot method will use APCU by default unless it's disabled in you settings.php (`class_loader_auto_detect` settings set to FALSE) which can be found here:

https://github.com/drupal/drupal/blob/9.2.x/core/lib/Drupal/Core/DrupalKernel.php#L476

So, if the `class_loader_auto_detect` setting is not set just make it TRUE and use it - that's what's in code. When that happens we know that APCU extension is already used. But what exactly does it? Well, it does walk through all of the php classes and cache its names, locations and namespaces. That's the reason your module is broken after the relocation. Let's assume you have a custom block defined at `/app/drupal/web/modules/custom/your_module/src/Plugin/Block/CustomBlock.php`. This block plugin class, namespace and its location is cached by APCU. Let's prove it. Just call `apcu_cache_info()` function which is part of function which `APCU` extension brings. Now, move your module into new location and for testing purposes (for example to: `/app/drupal/web/modules/contrib/your_module/src/Plugin/Block/CustomBlock.php`). Now, call again apcu_cache_info() function. You can see that the location of block plugin class is still registered at `/app/drupal/web/modules/custom/your_module/src/Plugin/Block/CustomBlock.php`. So this is the old location. Now, try to clear drupal cache and call it again. It's still the same. It's not going to be changed as it's not part of drupal cache. What drupal cache does in terms of relocating modules is an entry in you cache_container table. It contains information about location of your .module file and .yml file, but not about specific php classes.

So, what exactly happens when you try to display or load a block which has been relocated? Well, it'll throw a PluginException error, which happens here:

https://github.com/drupal/drupal/blob/9.2.x/core/lib/Drupal/Component/Plugin/Factory/DefaultFactory.php#L96

```
if (!class_exists($class)) {
  throw new PluginException(sprintf('Plugin (%s) instance class "%s" does not exist.', $plugin_id, $class));
}
```

It fails on class_exists() function because it gets the name and namespace of class which is still registered in `/app/drupal/web/modules/custom/your_module/src/Plugin/Block/CustomBlock.php` not `/app/drupal/web/modules/contrib/your_module/src/Plugin/Block/CustomBlock.php` location. That's what php gets. So, what can we do about it. Well, it seems like pretty simple task, we need to clear the APCU cache `apcu_clear_cache()`. But there is a catch, if you do it from console it'll not work, in short it's not hitting the same APC segment of your webserver (more details here: https://stackoverflow.com/a/43501450/567058). It needs to be triggered from browser. But keep in mind that it needs to be called at very early stage of drupal bootstrap or on any drupal page that actually loads without any issues. So, don't try to call it on broken block page in preprocess function or similar, because it'll never gets to preprocess functions. It'll fail way before and it'll never have a chance to clear apcu cache. The best solution is to use extra contrib module which brings apcu cache clear option or call it on any non broken page, might be custom controller or just a preprocess_html, up to you.

Final thoughts
As you can see drupal 8 modules relocation might be really tricky. There's several things that needs to be checked. There might be modules that can be relocated by clearing drupal cache and that's enough, just because the module constis of .module and .info files, without any php custom classes and there's nothing to be cached by apcu and all the information (cache_container table) are storred in DB, so it'll just work. There are cases where modules have custom classes and still can be relocated just becasue there's no apcu extension in your php instace or you have `class_loader_auto_detect` setting set to FALSE and uncommented. So, each case might be different and I think this is the reason why people are so confused about relocating modules and for some clearing the drupal cache will work and for other will not.

