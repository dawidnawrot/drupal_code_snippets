## Logging data procedural way

```php
\Drupal::logger('my_module')->notice('message');
\Drupal::logger('my_module')->log('message');
\Drupal::logger('my_module')->warning('message');
\Drupal::logger('my_module')->warning('error');
// Log array data:
\Drupal::logger('debugging')->warning('<pre><code>' . print_r($_GET, TRUE) . '</code></pre>');
```
