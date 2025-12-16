---
layout: post
title: How to Install Redis Extension on LAMPP (Ubuntu)
---

First, run these commands in the terminal to update packages and install the required tools:
```markdown
sudo apt-get update
sudo apt-get install php-redis build-essential libtool autoconf unzip wget mlocate
```



You can use any Redis extension version instead of redis-5.3.2.tgz. Check available versions here:[pecl.php.net](https://pecl.php.net/package/redis)

Then download redis-5.3.2.tgz with this command:

```markdown
wget https://pecl.php.net/get/redis-5.3.2.tgz
```

Find the file location with:

```markdown
locate redis-5.3.2.tgz
```

If locate cannot find the package, update its database (by default it is updated once a day):
```markdown
sudo updatedb
```

Go to the directory where the file is saved (for example, we assume the file is at /home/user/Downloads/lampp_extensions):

```markdown
cd /home/user/Downloads/lampp_extensions
```
Extract the archive:
```markdown
tar xzf redis-5.3.2.tgz
```
Then build and install the extension:
```markdown
cd redis-5.3.2
phpize
./configure --with-php-config=/opt/lampp/bin/php-config
make
sudo make install
```
###Tip: If you get "access denied" when running make install, use sudo make install.

Finally, add the following line to /opt/lampp/etc/php.ini:
```markdown
extension="redis.so"
```

Restart XAMPP / LAMPP:
```markdown
sudo /opt/lampp/lampp restart
```
Open your PHP info page and search for redis (Ctrl+F):
```markdown
localhost/phpinfo.php
```

Congratulations — the Redis extension should now be installed.

If you have any problem, you can send it to me at:
```markdown
amisfaking@gmail.com
```
Bye!
