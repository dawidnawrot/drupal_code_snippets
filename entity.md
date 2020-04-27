# Loading entities
## Loading entity from url
```php
if ($node = \Drupal::request()->attributes->get('node')) {
  ksm($node); // Prints out array of node object.
}
```
## Loading subscription entity procedular way:

```php
$subscription = \Drupal::entityManager()->getStorage('subscription');
// Load all entities.
$entities = $subscription->loadMultiple();
// Load specific (id = 5) entity.
$single_entity = $subscription->load(5);
// Load title field (check list of methods or investigate php entity definition file to see available methods).
$title = $single_entity->getTitle();
// Load field_date value.
$date = $single_entity->get('field_date')->getValue();
// Or.
$date = $single_entity->get('field_date')->value;
// Load entity_reference field value.
// This will load just first value. If the field has cardinality set to > 1 then you need to
// call getValue() method - see below.
$reference = $single_entity->get('field_products_reference')->target_id; 
// Getting multiple reference values
$references = $single_entity->get('field_products_reference')->getValue();
```

## Loading entity by getQuery method and getting values from referenced entity:
We can load enities using getQuery method. It's quite simple and effective. Although it's not as powerfull as direct query yet it's still pretty handy. Especially part with loading referenced entities - it's very straightforward. Here is the example:

```php
$query = $this->entityManager()->getStorage('price')->getQuery();
$query->condition('status', 1, '=');
// Here is the handy part. We know that price entity is connected to product by 
// field_price_product_reference reference field. Therefore we can get any value
// from product knowing only the ID of the referenced product. 
// So, `field_price_product_reference` representing an ID of a product in price entity.
// Then we add `.entity` - we always do it this way, that's what the syntax is. Then we
// add `:` and then machine name of an entity, in out case it's `product` and after that
// we call any field we want from product entity in this case `field_product_category`.
$query->condition('field_price_product_reference.entity:product.field_product_category', $final_categories, 'IN');
$ids = $query->execute();
```
# Updating entities
## Updating entities programmatically

The point of this tip is to make just one entry published and unpublish automatically rest of entries. Let's say you have your price entity provided in prices/src/Entity/Price.php file (this might be applied to nodes too, but let's stick with price) and the class is the extension of ContentEntityBase class. So this entity has its published/status field. The method we're gonna use is postSave. Within this method during the save process we will check if the entity has status set to 1 and if it has we will unpublish all the rest of entities with status flag set to 1. This is the code we're gonna use:

```php
/**
   * {@inheritdoc}
   */
  public function postSave(EntityStorageInterface $storage, $update = TRUE) {
    parent::postSave($storage, $update);
    // Note, if you want to access the entity you're working on use just $this
    // It'll give you loaded entity with all its properties, like so
    // $this->get('your_field')->value or entity reference field like so
    // $this->get('field_price_product_reference')->target_id.
    // Get all published prices.
    $query = $this->entityManager()->getStorage('price')->getQuery()
      ->condition('status', 1, '=')
    $ids = $query->execute();

    // Unset current price from array. Because this is a postSave values already
    // exists in the DB, so by querying price entity we get a list of entities
    // including the one we're working on.
    unset($ids[$this->id()]);

    // If current price is set to published we unpublish all the rest of
    // prices, so we end up with just one published price within
    // certain product (field_price_product_reference).
    if ($this->getStatus()) {
      foreach ($ids as $id) {
        $price = $this->entityManager()->getStorage('price')->load($id);
        $price->set('status', 0);
        // This is required here, however if you make updates in hook_entity_update
        // it might be ignored.
        $price->save(); 
      }
    }
  }
  ```

Additionally this can be also done with preSave method. Also related hooks can be used it's [hook_entity_presave](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Entity%21entity.api.php/function/hook_entity_presave/8.2.x) and [hook_entity_update](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Entity%21entity.api.php/function/hook_entity_update/8.2.x) - which corresponds to postSave method.

## Updating entity anywhere

Simple example of updaing entity:

```php
// Load entity.
$subscriber = $this->entityTypeManager->getStorage('subscription')->load(5);
// Set new value for the specific field added by UI, fields added in code
// has their own methods to update.
// $update_product_ids is an array of same keys and values, like so
// [0 => '0', 1 => '1'] etc.
$subscriber->set('field_subscription_fuel_types', $update_product_ids);
$subscriber->save();
```
