> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [medium.com](https://medium.com/@lesliedouglas23/how-to-install-mysql-5-0-on-ubuntu-20-04-or-later-4d27de464eef)

> Hey there,

[

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1*fnfBG828DOdXbhgvTzgEVg.jpeg)

](https://medium.com/@lesliedouglas23?source=post_page-----4d27de464eef--------------------------------)![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1*bCYJvD0hj9Uo_CCbX09Skw.png)Title image

Hey there,

Maybe Mysql5.7 is the requirement for the new startup you work for… maybe you prefer its syntax. Whatever your reasons are for sticking with 5.7 when there is a newer, shinier 8.0 waiting to be tapped, are yours and yours alone. Let me help you set it up.

**Step 1:**

Run the command below to update your system packages

```
sudo apt update

```

**Step 2:**

Install [wget](https://www.gnu.org/software/wget/) using the command below:

```
sudo apt install wget -y

```

**Step 3 :**

You can then download the MySQL repository using the command below:

```
wget https://dev.mysql.com/get/mysql-apt-config_0.8.12-1_all.deb

```

**Step 4 :**

If you have downloaded the repository, you can then install it using the command :

```
sudo dpkg -i mysql-apt-config_0.8.12-1_all.deb

```

**Step 5 :**

Once you are done installing from step 4, a prompt will appear :

Step 5 a:  
Select Ubuntu bionic from this first option

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1*KqHH0Fm_FAKALKs7b4msQA.png)

Step 5 b:

The options below will show MySQL 8.0 as the default option. Click on the first option and press enter

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1*_LZwZwtQzHggG4dGJWrNMw.png)options when configuring mysql

Step 5 c:  
Upon clicking on the first option, you can then change the mysql server and cluster you want installed. Click on MySQL 5.7 and press enter:

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1*j5eQVFcp-fR5n-kEKuCq8Q.png)option page to select mysql version

Step 5 d:

MySQL 5.7 should now be selected as your default … navigate to the “OK” button and click

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1*TUOK7bszDb5DkNCX-LufbA.png)options when configuring mysql with version 5 selected

Step 6:

Update all your repositories

```
sudo apt-get update

```

Step 7:

Run the command below :

```
sudo apt-cache policy mysql-server

```

Your result should contain MySQL 5.7 as seen below:

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1*JjflCivXGfcKnFlpxkZl1w.png)versions of mysql in cache

If your result does not contain MySQL 5.* it means that your update from step 6 failed and because of that, MySQL 5.7 was not added to the cache.

Try running the update command again :

```
sudo apt update

```

You would notice that the MySQL bionic InRelease package failed to update:

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1*tTTTwIs-tJf9CMAh0QQ7yg.png)MySQL Package failed update

If this is the case, there are 2 ways around this:

Step 7 a:

You can run :

```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 467B942D3A79BD29

```

to import the missing gpg key so that your system can update securely.

after which you then try to update once again:

```
sudo apt update

```

OR

Step 7 b:

You can decide to update all your packages regardless of the security status of the packages

```
 sudo apt update --allow-insecure-repositories

```

You would notice a warning but the packages have all been updated:

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1*oMojD0fQ5f2n37Z0589feg.png)MySQL Package update successful

After a successful update , Run the [command](#aa4d) again

Step 8:

At this point, you have the mysql 5.7 repository in your system so you can now proceed to install it

```
sudo apt install -f mysql-client=5.7* mysql-community-server=5.7* mysql-server=5.7*

```

If you do not encounter any error, proceed to Step 9

On the off chance that you are using a version of Ubuntu > = 24, when you finally try to install mysql 5.7, you might encounter the error below :

```
mysql-community-client : Depends: libaio1 (>= 0.3.93) but it is not installable
                      Depends: libtinfo5 (>= 6) but it is not installable
mysql-community-server : Depends: libaio1 (>= 0.3.93) but it is not installable

```

If that is the case, check out this [stack exchange question](https://askubuntu.com/questions/1335720/cannot-install-mysql-server/1520455#1520455) about resolving said error

Step 9:

Press Y to begin the installation.  
Set the root password for your mysql when the prompt comes up.

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1*R-PNuj8Yc0L6VtZvO3tgYA.png)Enter password for root user

Remember the password you enter as this is the password of the root user and changing it can be a hassle.

Tada! .. you have completed your installation of MySQL version 5.0

To check your MySQL version, run:

```
mysql --version

```

To login to your SQL DB, run:

```
mysql -u root -p

```

If you found this helpful and you are a kind person, feel free to leave a clap.