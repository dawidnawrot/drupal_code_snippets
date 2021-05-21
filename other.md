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

With each kernet update (currently running on 5.4.0-53-generic version, type: `uname -r` in terminal to get to know which version is installed) the driver might not work and it needs to be updated. Now the driver is installed as part of kernel using dkms. You can check the drivers loaded into kernet by typing: `sudo dkms status`. The output for the time being is: 

```
8812au, 5.6.4.2_35491.20191025, 5.4.0-53-generic, x86_64: installed
nvidia, 450.66, 5.4.0-52-generic, x86_64: installed
nvidia, 450.66, 5.4.0-53-generic, x86_64: installed
```

Previously I had a warning there is some difference in 8812au row, so I had to uninstall it by typing:
```
sudo dkms remove 8812au/5.6.4.2_35491.20191025 -k 5.4.0-53-generic
```

So, after uninstalling it `sudo dkms status` showed only nvidia. The I cloned this repo: https://github.com/aircrack-ng/rtl8812au (it's downloaded into `~/Downloads/new_netis_driver/rtl8812au`) and followed the instructions in readme file:
```
sudo apt-get install dkms
sudo make dkms_install
```

And after reboot it started to working, however I tried many different solutions in order to make it work. 

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
```
xmodmap ~/.Xmodmap
```

**VScode setup - xdebug and symlinks 2021**
xdebug - for single site SE xdebug setup put a ./.vscode/launch.json file in main dir. This is the setup for xdebug > 3.0

```
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Listen for XDebug",
      "type": "php",
      "request": "launch",
      "port": 9003,
      "pathMappings": {
        "/app/profiles/ck": "/home/dawid/Documents/circlek/profiles/ck",
        "/app/sites/se/web": "/home/dawid/Documents/circlek/sites/se/web",
      },
      "xdebugSettings": {
        "show_hidden": 1
      }
    }
  ]
}
```

If you want to track different site just put another debug config file in different site.

Symlinks:
As for working symlinks in vscode the symlink needs to be done in parent os. It cannot be created in any dockerized system as it's going to be seen as single file. One the other hand it still should be treated as dockerized system as a symlink. This way it's easy to setup a symlinked local environment.

**AWS**
`avs-vault login cksites-test` - loguje w przegladarce (i tam uslugi S3 i CloudWatch ECS) - backupy sa na S3 w kat. cksites-prod-backups

`aws-vault exec cksites-test` - loguje w konsoli do AWS wybierajac region EU-WEST-1. 
Nastepnie `aws s3 ls` listuje to co na s3, przykladowo `aws s3 cp s3://cksites-prod-backups/cksitesdk.gz ./` sciaga baze.
