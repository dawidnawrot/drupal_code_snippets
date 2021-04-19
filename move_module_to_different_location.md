Moving modules to a different location without disabling it is a quite tricky process in Drupal 8. There's a lot of discussion about it but I coudn't find a proper solution so I've decided to digg a bit to find out what's behind it. In Drupal 7 it was easy, just put a module to new location and you're done. So, this is the quick step through without answering why you need to do this or that.

With APCU extension enabled.

1. Check which version of PHP you have and if `apcu` extension is installed and enabled. You should do it by visiting `/admin/reports/status/php`. Keep in mind there's no point to check it through the console as it might use different php.ini file. This needs to be done through browser.
2. Move your module code to different location.
3. Clear drupal cache.
4. If you do have a `apcu` extension enabled you need to call `apcu_clear_cache()` function, but it has to be triggered from browser. There's no way it'll work by calling it from console not drush, so keep in mind that `drush ev "apcu_clear_cache();"` will not work.
5. That's it, now your module should be available to use and it should not complaining by missing PHP classes or plugins.

Without APCU extension enabled.

1. Move your module to different location
2. Clear drupal cache.
3. Done

Let's digg a bit, so we have some idea what's happning here. 

In most answers you will find out that cleaning drupal cache or providing extra `class_loader_auto_detect` setting to FALSE will work for you, and that is true. However, if you put your `class_loader_auto_detect` setting to FALSE you will loose class caching mechanism. 

