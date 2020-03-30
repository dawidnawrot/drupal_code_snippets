# Views snippets for drupal 8.
## Override value of the field (hook_preprocess_views_view_field approach)

If you want to override your table values for a view it might be done with hook_views_pre_render hook and with hook_preprocess_views_view_field. Important thing to note that if your view has any fields "outside" of scope of an entitiy, for example Custom Global Text and you'd like to adjust it you should use hook_preprocess_views_view_field hook. That becasue these kind of fields are not accessible at hook_views_pre_render hook - the prerender hook relies only on entity fields. Example: if your view has 4 fields link entity_id, entity_title, entity_published and custom global text then the last field is not going to be accessible to alter in prerender hook. Another thing worth to mention is a case where you have two different custom blobal text fields and you'd like to preprocess just one. Both of these fields are going to use presented as "nothing" - i'd expect it'd be a "nothing" and "nothing_1", but no. Both are going to have the same, so how do we vary both same fields? Well, if you edit your view in Views UI and you click the field you want to edit you can provide an administraive label and instead of label you can just put a machine name like "my_nothing_field". And once you save it you can easily extend your condition by providing && check like so: `&& $variables['field']->options['admin_label'] == 'my_nothing_field'`. 

In the example below I preprocess `nothing` field in `prices` view. Becasue I've added a special administrative label I can check if that's the field I want to preprocess. At the end you should override `$variables['output']` array by some translatable string or a value. Here I'm providing a difference between two numbers and provide it into `nothing` field.
 
```php
/**
 * Implements hook_preprocess_views_view_field().
*/
function ck_fuel_ui_preprocess_views_view_field(&$variables) {
  if ($variables['view']->id() == 'prices') {
    if ($variables['field']->field == 'nothing' && $variables['field']->options['admin_label'] == 'price_change') {
      $product_id = $variables['row']->_entity->get('field_price_product_reference')->getValue();
      if ($product_id[0]['target_id']) {
        $product_id = $product_id[0]['target_id'];

        $query = \Drupal::entityTypeManager()->getStorage('price')->getQuery()
          ->condition('field_price_date', $variables['row']->_entity->get('field_price_date')->value, '<')
          ->condition('field_price_product_reference', $product_id, '=')
          ->sort('field_price_date' , 'DESC')
          ->range(0, 1);
        $previous_price = $query->execute();

        if (count($previous_price) > 0) {
          $previous_price = \Drupal::entityTypeManager()->getStorage('price')->load(key($previous_price));
          $variables['output'] = ((float) $variables['row']->_entity->get('price_gross')->value - (float) $previous_price->get('price_gross')->value);
        }
      }
    }
  }
}
```

## Override value of the field (hook_views_pre_render(ViewExecutable $view) approach)

In this case you can override some data that comes directly from an entity, other field values like `Custom Global Text` are not accessible at this point. In this example I change the value of a real number by subtracting its initial value by 4.
```php
function hook_views_pre_render(ViewExecutable $view) {
  if ($view->id() == "prices" && $view->current_display == 'block_price_per_product') {
    foreach ($view->result as $value) {
      $price_gross = $value->_entity->get('price_gross')->value;
      $price_net = $value->_entity->get('field_price_price_net')->value;
      $value->_entity->set('price_gross', $price_gross - 4);
    }
  }
}
```

## Pass extra settings into a view
If your view depends on certian settings that needs to be used let's say in one of preprocess functions, but you'd like to pass it way before eg. right after loading view you can put these extra settings into $view->style_plugin->options array.

For example:
```php
// Load a view
$viw = Views::getView($view_machine_name);
$view->style_plugin->options['your_extra_settings'] = $extra_settings;
$variables['view'] = $view->render($view_display);
```

Later on in the preprocess function you can use it as follow:
```php
/**
 * Implements template_preprocess_views_view_table().
 */
function theme_preprocess_views_view_table(&$variables) {
  if ($variables['view']->id() == 'your_view') {
    $extra_settings = $variables['view']->style_plugin->options['your_extra_settings'];
  }
}
```
