# Other useful commands
**PHPCS:**
 - `phpcs --extensions=php,module,inc,theme --standard=Drupal,DrupalPractice your_path/to/file/or/dir`
 
**Drupal console:**
1. Go to lando ssh
2. Go to dir where drupal install exists (whether it's a `/web` or dir up)
3. Type `php /app/sites/training/vendor/drupal/console/bin/drupal.php generate:module`

**Drupal 9 check:**
1. Go to lando ssh
2. Go to site root (whther it's a `/web` or dir up)
3. Type `php vendor/mglaman/drupal-check/drupal-check --drupal-root=/app/sites/training/web/ web/profiles/custom/ck/modules/custom/ck_access/`

References: https://github.com/mglaman/drupal-check

**Screenshot**
 - `flameshot gui -d 3000`
More info here: https://github.com/lupoDharkael/flameshot

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

**VSC**

[Huge git files track](https://code.visualstudio.com/docs/setup/linux#_visual-studio-code-is-unable-to-watch-for-file-changes-in-this-large-workspace-error-enospc)

**Release**
1. Checkout to develop and pull
2. Checkout to master and pull
3. Checkout to develop and start new release (you don't have to provide any prefix just `0.3.0` is fine)
4. Finish new release - add tag description, so it's there eg.: Release 0.3.0
5. This should merge release both to develop and master and create a tag ck.0.3.0 and put you on develop
6. git push --tag (optionally if it's reejected just pull tags with git pull --tag) and then push
7. git push origin develop
7. Checkout to master and push changes to master (as it was merged from develop to master).

**Keyboard**
```xmodmap ~/.Xmodmap```
