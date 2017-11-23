**参考**：
1.[*小喵爱你*的博客](http://www.xiaomlove.com/2014/09/20/php%e5%ae%89%e8%a3%85gearman%e6%89%a9%e5%b1%95%e5%ae%9e%e7%8e%b0%e5%bc%82%e6%ad%a5%e5%88%86%e6%ad%a5%e5%bc%8f%e4%bb%bb%e5%8a%a1/)
2.[PHP Manual](http://php.net/manual/zh/book.gearman.php)
##依赖
1.gcc44
2.boost >=1.39
3.libevent
4.php5.3+
5.update ld.so.conf
##安装依赖（Ubuntu 14.04 LTS）
```bash
$ sudo apt-get install libboost-dev
$ sudo apt-get install libevent-dev
$ sudo apt-get install libgearman-dev
$ sudo apt-get install gearman-job-server
```
##安装gearman到PHP
```bash
$ wget http://pecl.php.net/get/gearman-1.1.2.tgz
$ tar xvzf gearman-1.1.2.tgz
$ cd gearman-1.1.2/
$ /usr/local/php/bin/phpize
$ ./configure --with-php-config=/usr/local/php/bin/php-config
$ make
$ sudo make install
Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-non-zts-20121212/
```
编辑php.ini,添加：
```php
extension=/usr/local/php/lib/php/extensions/no-debug-non-zts-20121212/gearman.so
```
##重启apache，检查验证
```bash
$ sduo service httpd restart
$ /usr/local/php/bin/php --info | grep gearman
gearman
gearman support => enabled
libgearman version => 1.0.6
PWD => /home/www/Downloads/gearman-1.1.2
_SERVER["PWD"] => /home/www/Downloads/gearman-1.1.2
```
##启动gearman服务
```bash
$ sudo gearmand -d -u daemon --log-file=/tmp/gearman.log
```
##运行DEMO程序
GearmanClient.php
```php
<?php
# The client script

# create our gearman client
$gmc= new GearmanClient();

# add the default job server
$gmc->addServer();

# set a couple of callbacks so we can track progress
$gmc->setCompleteCallback("reverse_complete");
$gmc->setStatusCallback("reverse_status");

# add a task for the "reverse" function
$task= $gmc->addTask("reverse", "Hello World!", null, "1");

# add another task, but this one to run in the background
$task= $gmc->addTaskBackground("reverse", "!dlroW olleH", null, "2");

if (! $gmc->runTasks())
{
    echo "ERROR " . $gmc->error() . "\n";
    exit;
}

echo "DONE\n";

function reverse_status($task)
{
    echo "STATUS: " . $task->unique() . ", " . $task->jobHandle() . " - " . $task->taskNumerator() .
         "/" . $task->taskDenominator() . "\n";
}

function reverse_complete($task)
{
    echo "COMPLETE: " . $task->unique() . ", " . $task->data() . "\n";
}
?>
```
GearmanWorker.php
```php
<?php
# The worker script
echo "Starting\n";

# Create our worker object.
$gmworker= new GearmanWorker();

# Add default server (localhost).
$gmworker->addServer();

# Register function "reverse" with the server.
$gmworker->addFunction("reverse", "reverse_fn");

print "Waiting for job...\n";
while($gmworker->work())
{
  if ($gmworker->returnCode() != GEARMAN_SUCCESS)
  {
    echo "return_code: " . $gmworker->returnCode() . "\n";
    break;
  }
}

function reverse_fn($job)
{
  echo "Received job: " . $job->handle() . "\n";

  $workload = $job->workload();
  $workload_size = $job->workloadSize();
  echo "Workload: $workload ($workload_size)\n";

  # This status loop is not needed, just showing how it works
  for ($x= 0; $x < $workload_size; $x++)
  {
    echo "Sending status: " . ($x + 1) . "/$workload_size complete\n";
    $job->sendStatus($x+1, $workload_size);
    $job->sendData(substr($workload, $x, 1));
    sleep(1);

  }

  $result= strrev($workload);
  echo "Result: $result\n";

  # Return what we want to send back to the client.
  return $result;

}

?>
```
命令行执行后分别输出：
```bash
$ /usr/local/php/bin/php GearmanClient.php
STATUS: 1, H:hubery-VirtualBox:2 - 1/12
STATUS: 1, H:hubery-VirtualBox:2 - 2/12
STATUS: 1, H:hubery-VirtualBox:2 - 3/12
STATUS: 1, H:hubery-VirtualBox:2 - 4/12
STATUS: 1, H:hubery-VirtualBox:2 - 5/12
STATUS: 1, H:hubery-VirtualBox:2 - 6/12
STATUS: 1, H:hubery-VirtualBox:2 - 7/12
STATUS: 1, H:hubery-VirtualBox:2 - 8/12
STATUS: 1, H:hubery-VirtualBox:2 - 9/12
STATUS: 1, H:hubery-VirtualBox:2 - 10/12
STATUS: 1, H:hubery-VirtualBox:2 - 11/12
STATUS: 1, H:hubery-VirtualBox:2 - 12/12
COMPLETE: 1, !dlroW olleH
DONE

```
```bash
$ /usr/local/php/bin/php GearmanWorker.php
Starting
Waiting for job...
Received job: H:hubery-VirtualBox:1
Workload: !dlroW olleH (12)
Sending status: 1/12 complete
Sending status: 2/12 complete
Sending status: 3/12 complete
Sending status: 4/12 complete
Sending status: 5/12 complete
Sending status: 6/12 complete
Sending status: 7/12 complete
Sending status: 8/12 complete
Sending status: 9/12 complete
Sending status: 10/12 complete
Sending status: 11/12 complete
Sending status: 12/12 complete
Result: Hello World!
Received job: H:hubery-VirtualBox:2
Workload: Hello World! (12)
Sending status: 1/12 complete
Sending status: 2/12 complete
Sending status: 3/12 complete
Sending status: 4/12 complete
Sending status: 5/12 complete
Sending status: 6/12 complete
Sending status: 7/12 complete
Sending status: 8/12 complete
Sending status: 9/12 complete
Sending status: 10/12 complete
Sending status: 11/12 complete
Sending status: 12/12 complete
Result: !dlroW olleH

```


