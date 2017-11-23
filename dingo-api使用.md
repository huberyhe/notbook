**异常**
```bash
hubery@U16:/var/www/html/iksdb-5.2$ php artisan api:routes

[Symfony\Component\Debug\Exception\FatalThrowableError]
Call to undefined method Illuminate\Foundation\Console\RouteListCommand::handle()
```
**分析**
查看`storage/logs/laravel.log`
```php_logs
[2017-11-09 09:04:28] local.ERROR: Symfony\Component\Debug\Exception\FatalThrowableError: Call to undefined method Illuminate\Foundation\Console\RouteListCommand::handle() in /var/www/html/iksdb-5.2/vendor/dingo/api/src/Console/Command/Routes.php:86
Stack trace:
#0 [internal function]: Dingo\Api\Console\Command\Routes->handle()
#1 /var/www/html/iksdb-5.2/vendor/laravel/framework/src/Illuminate/Container/Container.php(507): call_user_func_array(Array, Array)
#2 /var/www/html/iksdb-5.2/vendor/laravel/framework/src/Illuminate/Console/Command.php(169): Illuminate\Container\Container->call(Array)
#3 /var/www/html/iksdb-5.2/vendor/symfony/console/Command/Command.php(256): Illuminate\Console\Command->execute(Object(Symfony\Component\Console\Input\ArgvInput), Object(Symfony\Component\Console\Output\ConsoleOutput))
#4 /var/www/html/iksdb-5.2/vendor/laravel/framework/src/Illuminate/Console/Command.php(155): Symfony\Component\Console\Command\Command->run(Object(Symfony\Component\Console\Input\ArgvInput), Object(Symfony\Component\Console\Output\ConsoleOutput))
#5 /var/www/html/iksdb-5.2/vendor/symfony/console/Application.php(794): Illuminate\Console\Command->run(Object(Symfony\Component\Console\Input\ArgvInput), Object(Symfony\Component\Console\Output\ConsoleOutput))
#6 /var/www/html/iksdb-5.2/vendor/symfony/console/Application.php(186): Symfony\Component\Console\Application->doRunCommand(Object(Dingo\Api\Console\Command\Routes), Object(Symfony\Component\Console\Input\ArgvInput), Object(Symfony\Component\Console\Output\ConsoleOutput))
#7 /var/www/html/iksdb-5.2/vendor/symfony/console/Application.php(117): Symfony\Component\Console\Application->doRun(Object(Symfony\Component\Console\Input\ArgvInput), Object(Symfony\Component\Console\Output\ConsoleOutput))
#8 /var/www/html/iksdb-5.2/vendor/laravel/framework/src/Illuminate/Foundation/Console/Kernel.php(107): Symfony\Component\Console\Application->run(Object(Symfony\Component\Console\Input\ArgvInput), Object(Symfony\Component\Console\Output\ConsoleOutput))
#9 /var/www/html/iksdb-5.2/artisan(35): Illuminate\Foundation\Console\Kernel->handle(Object(Symfony\Component\Console\Input\ArgvInput), Object(Symfony\Component\Console\Output\ConsoleOutput))
#10 {main}
```
看到异常原因是`RouteListCommand`类没有`handle()`方法，根据继承关系（`Routes extends RouteListCommand extends Command`）我追踪了三个文件：
`vendor/dingo/api/src/Console/Command/Routes.php`
`vendor/laravel/framework/src/Illuminate/Foundation/Console/RouteListCommand.php`
`vendor/laravel/framework/src/Illuminate/Console/Command.php`
发现确实没有`handle()`的实现
**解决**
版本兼容的问题，在laravel5.5中是有这个实现的，我们看到handle与fire具有类似的功能，经常混用，于是暂且就用fire代替了。
在`vendor/dingo/api/src/Console/Command/Routes.php`的86行，将`parent::handle()`改成`parent::fire()`。