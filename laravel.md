`./vendor/bin/sail artisan make:migration create_flights_table` - create migration<br>
`./vendor/bin/sail artisan migrate` - run all migrations<br>
`./vendor/bin/sail artisan migrate:status` - check migrations status<br>
`./vendor/bin/sail artisan migrate:rollback` - rollbacks last migration (not nessecarily all)<br>
`./vendor/bin/sail artisan migrate:reset` - rollbacks all migrations<br>
`./vendor/bin/sail artisan migrate:refresh` - rollbacks and import all migrations<br>
`./vendor/bin/sail artisan migrate:refresh --seed` - rollbacks and import all migrations and migrate data<br>

**How to get mysql host to connect to?**

First if the container is started let's list it: `docker ps`. Then get the id of the mysql container, and then just put:

`docker inspect <container-id> | grep Gateway`

You should get something like this in response:

```
 "Gateway": "",
            "IPv6Gateway": "",
                    "Gateway": "172.29.0.1",
                    "IPv6Gateway": "",
```

And there you have it. 

**How to fix "Public Key Retrieval is not allowed" issue in DBeaver***

For DBeaver users v7.0.0+

Right click your connection, choose "Edit Connection"
On the "Connection settings" screen (main screen) click on "Driver Properties" tab
Set the property allowPublicKeyRetrieval to true and useSSL to false



