---
layout: post
title: "LAMP stack on Amazon EC2: Part 2"
localgov: false
tag:
date: "2014-04-29"
meta: "Set up a LAMP Stack on Amazon EC2's Free Tier: Part 2"
---

I'm kind of assuming you've set up an EC2 instance as per 'Setting up your first EC2 instance,' or at least something similar.

Now, we've got an EC2 instance up and running it's time to make use of it.

First we'll install the bits and pieces we need to get going, Apache, MySQL and PHP.

## Install

SSH into your instance using your key-pair file

Now we want to tell yum to update all the packages currently installed, if you're doing this step straight off the back of setting up your instance you won't need to run this step again.

{% highlight bash %}
sudo yum update -y
{% endhighlight %}

The -y flag just says 'install all updates without prompting me to decide,' it saves you having to say yes to each update.

Now we want to install apache but we know we need MySQL and PHP, may aswell do the lot together.

{% highlight bash %}
sudo yum groupinstall -y "Web Server" "MySQL Database" "PHP Support"
{% endhighlight %}

Again -y says, 'yes please install all packages without prompting me.' Yum will run the install for all the dependencies for a web server (Apache) MySQL and PHP which is nice and convenient.

Once Yum has finished doing its thing, we want to install the php-mysql package which doesn't come as part of the group install.

{% highlight bash %}
sudo yum install -y php-mysql
{% endhighlight %}

Once that's all done, we can start our newly installed Apache install

{% highlight bash %}
sudo service httpd start
{% endhighlight %}

We should then get confirmation that Apache has started, however, starting Apache manually means that each time the server restarts, we'd have to log in and start Apache ourselves. A better option is to have the server start Apache itself using the config parameters.

To have Apache start on boot, run:

{% highlight bash %}
sudo chkconfig httpd on
{% endhighlight %}

Once Apache's running you can check everything's okay by putting your Amazon Public DNS path into a browser, you should see an Amazon Apache test page (providing you've not yet put anything in /var/www/html).

Now we're up and running with Apache, it's probably best to give some permissions to our ec2-user so that we can copy files into the /var/www directory without any permissions issues.

If you run the following:

{% highlight bash %}
ls -l /var/www
{% endhighlight %}

You'll see that the root user owns the web root directory. Not ideal as you'd have to have root/su permissions to manipulate files in there which will prevent things like FTP/SFTP transfers.

First things first, we'll create a www group
{% highlight bash %}
sudo groupadd www
{% endhighlight %}

Then we'll add our ec2-user to that group

{% highlight bash %}
sudo usermod -a -G www ec2-user
{% endhighlight %}

To pick up our new group we have to exit and log back in so run

{% highlight bash %}
exit
{% endhighlight %}

Then log back into your instance and run 

{% highlight bash %}
groups
{% endhighlight %}

You'll then see 'www' as one of the listed groups for ec2-user.

Now that we're part of the www group, we want to set the group permissions of the web root directory to that www group, rather than root. We want to do it recursively too so we'll add in the -R flag.

{% highlight bash %}
sudo chown -R root:www /var/www
{% endhighlight %}

Now we need to change the web root directory permissions and its subdirectories so that the www group has write permissions

{% highlight bash %}
sudo chmod 2775 /var/www
{% endhighlight %}

We also need to set the same permissions for any future subdirectories

{% highlight bash %}
find /var/www -type d -exec sudo chmod 2775 {} +
{% endhighlight %}

Then we do the same for files within those directories and subdirectories

{% highlight bash %}
find /var/www -type f -exec sudo chmod 0664 {} +
{% endhighlight %}

Now we're all set up to put some files into the web root directory with the correct permissions, to check every things okay we can put in a quick PHP info file that'll give us the usual PHP install information. We'll quickly create a file, drop in the phpinfo statement and go and view the file on the server to check apache and php are both installed and running correctly.

To create the file and populate it with the phpinfo statement run:

{% highlight bash %}
echo "<?php phpinfo.php; ?>" /var/www/html/phpinfo.php
{% endhighlight %}

Now head over to your public dns address in the browser but this time append phpinfo.php to your url

<strong>http://your-amazon-public-dns/phpinfo.php</strong>

You should see a PHP info page, listing out all the installed modules and so forth. Pretty easy eh.

We don't really want this information hanging about on the server for anyone to view so we'll get rid of that file with a simple command:

{% highlight bash %}
rm /var/www/html/phpinfo.php
{% endhighlight %}

Done.

Last thing we need to do to finish the install is secure MySQL, so here goes...

{% highlight bash %}
sudo service mysqld start
{% endhighlight %}

MySQL will start and you'll get some shouty messsage about setting a password for your mysql root user etc but we'll get to that in a second.

First run

{% highlight bash %}
sudo mysql_secure_installation
{% endhighlight %}

You'll now be taken through a series of steps to securing the mysql installation:

1. You'll be asked for the password for the root account, there isn't one by default so just hit enter

2. Next it'll ask if you want to set a password, hit 'Y' and now you need to enter a password for your root mysql user. You'll need to type it in twice just to be sure.

3. Then type 'Y' to remove Anonymous user accounts

4. Then type 'Y' to remove remote login, that way you have to be on the server to log into mysql, just another layer of security for us.

5. Then type 'Y' again to remove the test database.

6. Finally type 'Y' again to reload the privileged's and save everything you've just done.

You can then start and stop the mysqld service as and when you'd like with:

{% highlight bash %}
sudo service mysqld start|stop
{% endhighlight %}

In the same way we set Apache to start on system boot, we can set MySQL to do the same thing, this is good for things like CMS installs that rely on the database to be available (Drupal / WordPress etc).

{% highlight bash %}
sudo chkconfig mysqld on
{% endhighlight %}

That's everything installed. Apache web server is running with the web root as /var/www/html so you can pop your website file in that directory and MySQL is running so you can install your database too.

