## Example of CSV migration with entity reference field using migrate_lookup process

Here is the schema of dependent migrations. Although it might looks complex at first sight it really isn't. I just used my own files, so it has more fields then example actually needs. The most important one is the `id` of product in `ck_fuel_products_dk` migration and `product` in `ck_fuel_prices_dk` migration:

<img src="https://i.imgur.com/i4GcLzJ.jpg" alt="migration with migration_lookup" />

So, if you have a migration which is dependent on second migration and the field type used to connect both is an entity reference field it's pretty straightforward to migrate it. You don't really need any extra coding, all you have to do is to prepare yml files with migration.

So, let's assume we have a product entity and price entity. The price entitiy has a entity reference field which reference a product. So in that scenario first you need to provide list of products in order to create a price. So the first migration is a product migration which is pretty simple and it follows like so:

**Product migration:**
yml file:
```
id: ck_fuel_products_dk
migration_group: ck_fuel_ui
label: 'Create products'
source:
  plugin: csv
  path: /data/fuel_products_dk.csv
  ids: [id]
destination:
  plugin: entity:product
process:
  bundle:
    plugin: default_value
    default_value: product
  name: name
  field_product_label: label
  field_product_category: category
  field_product_icon: icon
  field_product_present_on_hp: homepage
  field_product_unit_of_measure: unit
```

And this is the contents of the csv file for products:
```
id,name,label,category,icon,homepage,unit
98,98,98,b2c,default,1,l
```
It has just one product provided for simplicity of the example.

**Price migration**
Yml file:
```
# Import menu links to default pages, like About us.
id: ck_fuel_prices_dk
migration_group: ck_fuel_ui
label: 'Create fuel prices for DK'
source:
  plugin: csv
  path: /data/fuel_prices_dk.csv
  ids: [id]
destination:
  plugin: entity:price
process:
  bundle:
    plugin: default_value
    default_value: price
  price_gross: price_gross
  field_price_price_net: price_net
  field_price_product_reference:
    plugin: migration_lookup
    migration: ck_fuel_products_dk
    source: product
    no_stub: true
  field_price_date: date
  field_price_currency: currency
migration_dependencies:
  required:
    - ck_fuel_products_dk
```

And this is the contents of the csv file for prices:
```
id,price_gross,price_net,product,date,currency
1,9.67,8.65,98,2020-03-31,pln
2,10.20,9.50,98,2020-03-30,pln
3,1,2.50,98,2020-03-29,pln
```

As you can see there's a `field_price_product_reference` field defined. This is the part where the magic happens. So the plugin is migration_lookup and there's an id of the dependent migration which is set to be `ck_fuel_products_dk`. As for the source we just need to put the name of the column from price csv file that reflects an id of the product which is the `product` column. That's all you have to do, no more, no less. It works like charm.
