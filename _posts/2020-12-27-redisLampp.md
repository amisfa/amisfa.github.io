---
layout: post
title: how to install redis extension on lampp (UBUNTU)
---

First you should run these commands in terminal and install these packages:
```markdown
sudo apt-get update
sudo apt-get install php-redis build-essential libtool autoconf unzip wget mlocate
```



You can use each redis version instead redis-5.3.2.tgz, check here:[pecl.php.net](https://pecl.php.net/package/redis)

then you must download redis-5.3.2.tgz with this command:
```markdown
wget https://pecl.php.net/get/redis-5.3.2.tgz
```

Get the location file with this command:
```markdown
locate redis-5.3.2.tgz
```


If "locate" can't found your package link you must update database of "locate" with this command(By default it is updated once in a day):
```markdown
sudo updatedb
```

Now get link and go to the file directory(you can save this file anywhere you want, we suppose file in this link : /home/user/Downloads/lampp_extensions/redis-5.3.2.tgz
):
```markdown
cd /home/user/Downloads/lampp_extensions
```
Now you should extract file
```markdown
tar xzf redis-5.3.2.tgz
```
Then run:
```markdown
cd redis-5.3.2
phpize
./configure --with-php-config=/opt/lampp/bin/php-config
make
sudo make install
```
###TIP:if you have access denied when using "make install" you should use "sudo make install"

Finally add below line to /opt/lampp/etc/php.ini
```markdown
extension="redis.so"
```

Restart your xampp with this command:
```markdown
sudo /opt/lampp/lampp restart
```
And search redis with ctrl+f in this page:
```markdown
localhost/phpinfo.php
```

congratulations you could add redis extension...

if you have problem, you can send it me:
```markdown
amisfaking@gmail.com
```
bye bye
