# Defining dynamic library

Sometimes simple module.libraries.yml file is not enough, as you need to pass some variables and yml library file cannot handle any of dynamic value. In that kind of case you need to define a new lib by using hook_library_info_build(). Example below:

```php
/**
 * Implements hook_library_info_build().
 */
function ck_cookie_complience_library_info_build() {
  $cookie_consent_id = Settings::get('cookie_consent_id');
  $libraries['cookie_banner'] = [
    'js' => [
      '//consent.cookiebot.com/uc.js' => [
        'type' => 'external',
        'weight' => -10,
        'attributes' => [
          'data-cbid' => $cookie_consent_id,
          'type' => 'text/javascript',
          'async' => TRUE,
          'data-culture' => \Drupal::languageManager()->getCurrentLanguage()->getId(),
          'blockingmode' => 'auto',
        ],
      ],
    ],
  ];

  return $libraries;
}
```

Having that you can use it as below:

```php
/**
 * Implements hook_preprocess_HOOK() for html.html.twig.
 */
function ck_cookie_complience_preprocess_html(array &$variables) {
  // Load cookie banner library.
  if (\Drupal::currentUser()->isAnonymous()) {
    $variables['page']['#attached']['library'][] = 'ck_cookie_complience/cookie_banner';
  }
}
```

Pretty simple yet very helpful. We have passed id of a cookie from settings which can be different for different sites / envs.
