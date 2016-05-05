---
layout: post
title:  How To Disable Xdebug with PHP 7 CLI Ubuntu
date:   2016-05-05 22:00:00
categories: tooling
---
If you are running PHP 7 with Xdebug, or like me, use Laravel Homestead, you
might have seen this error message:

> You are running composer with xdebug enabled. This has a major impact on
runtime performance. See https://getcomposer.org/xdebug

And Composer will take an age to do anything.

To fix this, prior to PHP 7 people would suggest to comment out the extension
from your `php.ini` file. However, in PHP 7 they are no longer in there.

Instead, we use the `phpdismod` command.

```bash
$ sudo phpdismod -s cli xdebug
```

The `-s` flag tells it to disable Xdebug for the CLI SAPI (`/etc/php/7.0/cli`)
and not FPM.

And just like that, the warning message should be gone. No need to restart PHP.