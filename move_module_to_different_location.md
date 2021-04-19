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

In most answers you will find out that cleaning drupal cache or providing extra `class_loader_auto_detect` setting to FALSE will work for you. Sometimes it's work, sometimes not. If you put your `class_loader_auto_detect` setting to FALSE you will loose class caching mechanism. If it is an option and it's ready to use why would you? There's no point to do it. Instead let's use it. So, let's assume that you do have `APCU` extension enabled. While drupal 8 is booting DrupalKernel.php boot method will use APCU by default if it's not disabled in you settings.php (`class_loader_auto_detect` settings) which can be found here:

https://github.com/drupal/drupal/blob/9.2.x/core/lib/Drupal/Core/DrupalKernel.php#L476

So, if the `class_loader_auto_detect` setting is not set just make it TRUE and use it. When that happens we know that APCU extension is already used. But what exactly does it? Well, it does walk through all of the php classes and cache its names, locations and namespaces. That's the reason your module is broken after the relocation. APCU cache however can be cleared:

https://www.php.net/manual/en/function.apcu-clear-cache.php

But first, let's prove that the APCU keep your classes locations. Just move your module into new location and for testing purposes just call apcu_cache_info() function. It'll show you your PHP classes registered in old location. That that information is still used by class_exists() function for locating your classes. Because in the end calling class_exists() function will break your site. If the class plugin cannot be found (which happens 

