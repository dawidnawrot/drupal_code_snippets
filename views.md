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

## Custom field for views and custom grouping values

What we want to do is to provide a custom field for views and use this field to provide some custom grouping. Let's imagine we have a select field in the entity with some values to choose from. Each of the value has a prefix like so:

 - b2c_category1
 - b2c_category2
 - b2c_category3
 - b2c_category4
 - b2b_category1
 - b2b_category2
 - b2b_category3
 - b2b_category4
 
So, as you can see the common prefix for categories is b2b and b2c. We want to group some products with b2b and b2c. Obviously this cannot be done directly in a view as we don't have the field that returns b2b and b2c value. So we need to provide a new field. We can use drupal console to get an initial skeleton of view field. This is the command: `drupal generate:plugin:views:field`. Then you need to define where you want to store your new field by providing a module name, next the plugin field class name, plugin field description and that's it. Out of it you will get two files, one is a module_name.view.inc where you define how your new field is going to be storred and second one is a plugin file placed in `src/Plugin/views/field/YourField.php`.

So code for my field looks like this:

```php
<?php

namespace Drupal\ck_fuel_ui\Plugin\views\field;

use Drupal\Core\Form\FormStateInterface;
use Drupal\views\Plugin\views\field\FieldPluginBase;
use Drupal\views\ResultRow;

/**
 * A handler to provide a field that is completely custom by the administrator.
 *
 * @ingroup views_field_handlers
 *
 * @ViewsField("leading_category_field")
 */
class LeadingCategoryField extends FieldPluginBase {

  /**
   * {@inheritdoc}
   */
  public function usesGroupBy() {
    return FALSE;
  }

  /**
   * {@inheritdoc}
   */
  public function query() {
    // Do nothing -- to override the parent query.
  }

  /**
   * {@inheritdoc}
   */
  protected function defineOptions() {
    $options = parent::defineOptions();

    $options['hide_alter_empty'] = ['default' => FALSE];
    return $options;
  }

  /**
   * {@inheritdoc}
   */
  public function buildOptionsForm(&$form, FormStateInterface $form_state) {
    parent::buildOptionsForm($form, $form_state);
  }

  /**
   * {@inheritdoc}
   */
  public function render(ResultRow $values) {
    if ($category = $values->_entity->get('field_product_category')->value) {
      $leading_categories = [
        'b2b' => $this->t('Business (B2B)'),
        'b2c' => $this->t('Customer (B2C)'),
      ];
      return $leading_categories[substr($category, 0, 3)];
    }
    return NULL;
  }

}

```

The one and only thing I've changed here is the render method. So, the values needs to be rendered (accessible) first in order to use it in a view as a grouping field. Render method just gets the value of field_product_entity which contains one of b2b_categoryN or b2c_categoryN categories and transpile it into b2b and b2c values. As you can see there's also a leading_cateogries array, so we return just a nice string rather than just a key. And that's that as for the plugin itself.

The second file that is provided for you is your_module.views.inc. This module provides hook_views_data function. Inside of it you can define where the field is placed and how it's discovered. By default it's going to be placed in `views` structure, so the field is going to be accessible everywhere, however if you want the field to be visible only on certain entity you need to adjust it. The easy way is to declare hook_views_data_alter(&$data) and overview what's in $data as there comes all the groups. The based on `$data` you can adjust what's in hook_views_data, so for example this is my setup and it means that the field is going to be accessible only in product entity data:

```php
/**
 * Implements hook_views_data().
 */
function ck_fuel_ui_views_data() {
  $data['product']['table']['group'] = t('Product');
  $data['product']['leading_category_field'] = [
    'title' => t('Leading category field'),
    'help' => t('Provides prefix (leading category, eg. b2b or b2c) views field plugin.'),
    'field' => [
      'id' => 'leading_category_field',
    ],
  ];
  return $data;
}
```

As for the view itself, after clearing a cache you can just add your custom field as usual and use it for grouping in format settings.

