---
layout: post
title:  Simple Way To Install Laravel Homestead For A PHP 5.6 Project
date:   2017-01-24 22:00:00
categories: laravel
medium: https://medium.com/@chrisloftus/simple-way-to-install-laravel-homestead-for-a-php-5-6-project-2b5cc4710663
---
If you're running a recent version of Laravel Homestead (PHP 7) and you need to
install an old Laravel app, in my case 5.0. You'll run into difficulties as [5.0
is not compatible with PHP 7](https://laravel.com/docs/5.0/installation#server-requirements).

Here's how to set up Laravel Homestead with PHP 5.6 for the project.

SSH into your current Homestead box and change the directory to the Laravel 5.0
project.

```bash
cd ~/Homestead
vagrant ssh
cd ~/Code/laravel50
```

Require Homestead v2.2.2 for the project.
`composer require --dev laravel/homestead 2.2.2`

After it has installed, run the make command to generate the `Vagrantfile` and
`Homestead.yaml` files.

`php vendor/bin/homestead make`

Once this is done, exit the SSH session and change directory to the project path
on your host machine. Now boot the vagrant box:

`vagrant up`

This will download the v0.3.3 box and boot it.

*You may see a warning message about using a password to execute a MySQL query
on the command line.* This might be because the MySQL password has expired.

SSH into the box and login to MySQL:

```
vagrant ssh
mysql -u homestead -p
secret
SET PASSWORD = 'secret';
exit
```

Then to complete the provisioning:

`vagrant provision`

This will run the remaining scripts, then you should be good to go!
