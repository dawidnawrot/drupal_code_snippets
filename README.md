# Drupal code snippets

 - [Views](views.md)
 - [Entity](entity.md)

# Other useful commands
**PHPCS:**
 - `phpcs --extensions=php,module,inc,theme --standard=Drupal,DrupalPractice your_path/to/file/or/dir`
 
**Drupal console:**
1. Go to lando ssh
2. Go to dir where drupal install exists (whether it's a /web or dir up)
3. Type php /app/sites/training/vendor/drupal/console/bin/drupal.php generate:module
