## Add extra variables to already defined theme by hook_theme

```
/**
 * Implements hook_theme_registry_alter().
 */
function hook_theme_registry_alter(&$theme_registry) {
  $theme_registry['menu']['variables']['mobile'] = NULL;
}
```

## Theme table

```php
$table = [
  '#type' => 'table',
  '#caption' => t('Some title'),
  'attributes' => [
    'class' => [
      'fresh-food',
    ],
  ],
  '#header' => [
    t('Fruit'), // this might be set to NULL, if you want to mark there is a column but don't want to put any content there
    t('Vegatables'), // this might be set to NULL, if you want to mark there is a column but don't want to put any content there
  ],
];

// There are two ways of declaring rows, simple and more complex. Here is the simple one.
// Name of the key might be whatever you want. It's not displayed anywhere, just make sure you don't use # at the beginning.
// This table['0'] describes one row.
$table['0'] = [
  // key might be whatever you want, it's just gonna be matched to first row declared in $table #header.
  'fruit' => [
    '#markup' => 'Orange',
  ],
  'vegatable' => [
    '#markup' => 'Onion',
  ]
];

// And so on for 1,2,3...
// This would be the output:
// <table class='fresh-food'>
//   <tbody>
//     <tr>
//       <td>Orange</td>
//       <td>Onion</td>
//     </tr>
//   </tbody>
// </table>

// Complex one. This table[0] describes one row in a table.
$table[0] = [
  // Fruit
  'fruit' => [
    'data' => [
      '#type' => 'markup',
      '#markup' => 'Orange',
    ],
    '#wrapper_attributes' => [
      'class' => [
        'organe-color' // This will add orange-color class to td cell that keeps Orange value.
      ],
    ],
  ],
  'vegatable' => [
    'data' => [
      '#type' => 'markup',
      '#markup' => 'Onion',
    ],
    '#wrapper_attributes' => [
      'class' => [
        'white-color' // This will add white-color class to td cell that keeps Onion value.
      ],
    ],
  ],
  // These are attributes for tr.
  '#attributes' => [
    'class' => [
      'orange-and-white-food',
    ],
  ],
];

// This would be the output:
// <table class='fresh-food'>
//   <tbody>
//     <tr class="orange-and-white-food">
//       <td class="orange-color">Orange</td>
//       <td class="white-color">Onion</td>
//     </tr>
//   </tbody>
// </table>

```

More info here:
https://drupal.stackexchange.com/a/228993/14665
