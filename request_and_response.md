# Return completly custom html with drupal controller

Sometimes you want to return a html, but without all the theme wrappers like logo, nav, etc, but completly custom html code from `<html>` to `</html>` tag. How do we do this? Well it's pretty simple actually. Of course you need to have your route, like so:

`ck_chatbot.routing.yml`

Contents:
```yaml
ck_chatbot.chatbot_controller_hello:
  path: '/chatbot'
  defaults:
    _controller: '\Drupal\ck_chatbot\Controller\ChatbotController::content'
    _title: 'Chatbot'
  requirements:
    _access: 'TRUE'
```

...and the controller itself:

```php
<?php
namespace Drupal\ck_chatbot\Controller;

use Drupal\Core\Controller\ControllerBase;
use Symfony\Component\HttpFoundation\Response;
use Drupal\Core\Site\Settings;

/**
 * Class ChatbotController.
 */
class ChatbotController extends ControllerBase {

  /**
   * Chatbot.
   *
   * @return string
   *   Return html content of chatbot.
   */
  public function content() {

    $base_path = base_path();
    $module_path = drupal_get_path('module', 'ck_chatbot');
    $path = $base_path . $module_path . '/assets';
    $sitecode = Settings::get('ck_site_code');
    // Fallback to 'se' if training.
    // We should have testing env for chatbot.
    $sitecode = ($sitecode == 'training') ? 'se' : $sitecode; 

    $build = [
      '#theme' => 'ck_chatbot',
      '#module_path' => $path,
      '#sitecode' => $sitecode,
      '#cache' => [
        'max-age' => 0,
      ],
    ];
    $output = \Drupal::service('renderer')->renderRoot($build);
    $response = new Response();
    $response->setContent($output);
    return $response;
  }

}
```

So the important things starts from `$output = \Drupal::service('renderer')->renderRoot($build);` line. We just prepare an output out of custom ck_chatbot theme. This renderer service with its renderRoot method will provide an html for you. As for ck_chatbot theme, well it has all of the html code we want to return including `<html>` tag. Then we just instanciate new response and set content to the response and return it. That's it folks.
