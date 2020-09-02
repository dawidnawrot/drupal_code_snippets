It's easy to setup a custom hook, however there are couple of small things that are crucial to make it work.

First of all type of hook, for example alter hook which starts with `hook_` and ends with `_alter`.
For example:
hook_field_options_alter

This hook can be implemented like so:

```php
<?php
function color_options() {
  $options = [
    'orange' => t('Orange'),
    'red' => t('Red'),
    'blue' => t('Blue'),
  ];

  // Call modules that implement the hook, and let them add items.
  // Notice that in the call there's no hook word or alter, it's just a field_options.
  \Drupal::moduleHandler()->alter('field_options', $options);
  return $options;
}
```

So this way you can gather togather all the alters from external modules
and merge it here to return final list of options.

So, if we want to add another option with separate module you can easily implement the hook like so:

```php
/**
 * Implements hook_field_options_alter().
 */
function my_module_field_options_alter(&$options) {
  $options['green'] = t('Green');
}
```

Whenever we call color_options module we will end up with four options including green because of an alter hook used.
Here is another example with invokeAll method: https://drupal.stackexchange.com/a/276069/14665
