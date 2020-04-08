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

**Running up Netis card**

Try: `sudo modprobe 8812au`. If that doesn't work try:
```
cd ~/Downloads/netis_wifi_card_driver/RTL8812AU_linux_v4.3.20_16317_20160108/rtl8812au
make
sudo make install
sudo modprobe 8812au
```
I've noticed that card stops working after running Windows. If the above will not work, go a dir up to `RTL8812AU_linux_v4.3.20_16317_20160108` and try `sudo ./install.sh`, after that try again above commands.
