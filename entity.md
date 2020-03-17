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
        $price->save();
      }
    }
  }
  ```
