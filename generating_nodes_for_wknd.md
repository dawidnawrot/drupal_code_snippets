```
<?php

/**
* Implements hook_update_N().
*
* Add 60 articles with pagination term.
*/
function weekend_articles_update_9003(&$sandbox) {
  $taxonomy = \Drupal::entityTypeManager()->getStorage('taxonomy_term')->create([
    'name' => 'Pagination ' . rand(1, 100),
    'vid' => 'channel',
    'field_seo_h1_title' => 'Pagination',
    'field_seo_title_page' => 'Pagination',
    'status' => 1,
  ]);
  $taxonomy->save();

  for ($i = 0; $i <= 60; $i++) {
    $node = \Drupal\node\Entity\Node::create([
      'type' => 'article',
      'title' => 'PA-' . $i,
      'created' => 1720441052 + ($i * 60),
      'field_seo_title' => 'article '. $i,
      'field_channel_primary' => [
        'target_id' => $taxonomy->id(),
      ],
      'status' => 1,
      'moderation_state' => 'published'
    ]);
    $node->save();
  }
}
```
