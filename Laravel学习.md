# 高级篇
## 1.Composer快速入门
### 安装
```bash
wget https://getcomposer.org/installer
php installer
sudo mv composer.phar /usr/local/bin/composer
composer --version
```
### 使用中国全量镜像
```bash
composer config -g repo.packagist.org
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

### 常用命令
初始化：`composer init`
查找：`composer search monolog`
`composer show -all  monolog/monolog`
安装： `composer require monolog/monolog`
更新： `composer update`

### composer 安装 Larevel
方法1:
`composer create-project --prefer-dist laravel/laravel laravel_advanced`

方法2:
`composer global require "laravel/installer"`
`~/.composer/vendor/bin/laravel new  laravel_advanced`

## 2.Artisan基本使用
创建控制器：`php artisan make:controller StudentController`
创建模型：`php artisan make:model Student`
创建中间件：`php artisan make:middleware Activity`

## 3.Laravel中的用户认证
创建表：
`php artisan make:migration create_students_table --create=students`
`vim database/migrations/2017_09_21_084656_create_students_table.php`
`php artisan migrate`
填充：
`php artisan make:seeder StudentsTableSeeder`
`vim database/seeds/StudentsTableSeeder.php`
`php artisan db:seed --class=StudentsTableSeeder`
`vim database/seeds/DatabaseSeeder.php`
`php artisan db:seed`
