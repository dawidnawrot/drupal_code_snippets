# Drupal code snippets

 - [Views](views.md)
 - [Entity](entity.md)
 - [Batch](batch.md)
 - [Logger](logger.md)
 - [Accessing data](accessing_data.md)
 - [Migrate](migrate.md)

# Other useful commands
**PHPCS:**
 - `phpcs --extensions=php,module,inc,theme --standard=Drupal,DrupalPractice your_path/to/file/or/dir`
 
**Drupal console:**
1. Go to lando ssh
2. Go to dir where drupal install exists (whether it's a `/web` or dir up)
3. Type `php /app/sites/training/vendor/drupal/console/bin/drupal.php generate:module`

**Running just appserver_nginx_1**
 - `lando start -s appserver_nginx_1`
