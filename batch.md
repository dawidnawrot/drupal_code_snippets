Check ck_fuel_ui_subscription module
## Batch

Batch is a method which allows to chunk some work to separate requests. Let's say you have to update 10000 nodes at once. If you'd provide it in just a single request it would probably failed due to php time execution exceeded or memory allocate issue. That's why Drupal by default allows to divide your job into batches. There are two functions which you need to know in order to use it. First is a batch_set($batch) and second one is batch_process($redirect). First one is used for preparing your chunked data, second one is for fireing it. Once fired you should see a progress bar with the process.

<img src="https://i.imgur.com/xIo1hNG.png" alt="batch" />
