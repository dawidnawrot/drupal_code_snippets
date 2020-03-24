Check ck_fuel_ui_subscription module
## Batch

Batch is a method which allows to chunk some work to separate requests. Let's say you have to update 10000 nodes at once. If you'd provide it in just a single request it would probably failed due to php time execution exceeded or memory allocate issue. That's why Drupal by default allows to divide your job into batches. Although it might seem this functionality might be used for cron jobs it is not. For cron you should use Queue.

There are two functions which you need to know in order to use it. First is a batch_set($batch) and second one is batch_process($redirect). First one is used for preparing your chunked data, second one is for fireing it. Once fired you should see a progress bar with the process.

<img src="https://i.imgur.com/xIo1hNG.png" alt="batch" />

So let's assume you have a route and controller with `build` method which is fired once the route is triggered. Let's start very simple:
```php
<?php

namespace Drupal\batch_example\Controller;

use Drupal\Core\Controller\ControllerBase;
use Symfony\Component\DependencyInjection\ContainerInterface;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Drupal\ck_fuel_ui\Entity\Price;
use Drupal\ck_fuel_ui\Entity\Product;

/**
 * Class SubscriberQueueController.
 */
class SubscriberQueueController extends ControllerBase {
  /**
   * Build batches.
   *
   * @return string
   *   Return batch process.
   */
  public function build($hash) {
    if ($hash == $this->hash) {
      $batch = [
        'title' => $this->t('Processing subscribers queue'),
        'operations' => [],
        'init_message'     => $this->t('Preparring...'),
        'progress_message' => $this->t('Processed @current out of @total.'),
        'error_message'    => $this->t('An error occurred during processing'),
        'finished' => '\Drupal\ck_fuel_ui_subscription\Controller\SubscriberQueueController::queueFinished',
      ];

      $subscribers = array_chunk($this->entityTypeManager->getStorage('subscription')->loadMultiple(), $this->batchSize);

      foreach ($subscribers as $subscriber) {
        $batch['operations'][] = [
          '\Drupal\ck_fuel_ui_subscription\Controller\SubscriberQueueController::addToQueue',
          [$subscriber],
        ];
      }

      batch_set($batch);
      return batch_process('<front>');
    }
    else {
      try {
        throw new NotFoundHttpException();
      }
      catch (Exception $e) {
        return NULL;
      }
    }
  }
  
  /**
   * QueueFinished.
   */
  public static function queueFinished() {
    \Drupal::logger('queue')->notice('finished');
  }

  /**
   * Add to queue.
   */
  public static function addToQueue($subscribers) {
    $queue_factory = \Drupal::service('queue');
    $queue = $queue_factory->get('send_mail');
    foreach ($subscribers as $subscriber) {
      $item = new \stdClass();
      $item->id = $subscriber->id();
      $item->mail = $subscriber->getEmail();
      $product_ids = $subscriber->get('field_subscription_fuel_types')->getValue();
      foreach ($product_ids as $product_id) {
        $product_ids_for_db[] = $product_id['target_id'];
      }
      $ids = Price::getPriceIdsByProducts($product_ids_for_db);
      if (count($ids) > 0) {
        foreach (Price::loadMultiple($ids) as $price_key => $price_value) {
          $item->products[$price_key]['price_gross'] = $price_value->getPriceGross();
          $item->products[$price_key]['date'] = $price_value->get('field_price_date')->value;
          $product = Product::load($price_value->get('field_price_product_reference')->target_id);
          $item->products[$price_key]['product_label'] = $product->getName();
        }
      }
      $queue->createItem($item);
    }
  }
  
}
```
