Hello everyone I hope you are doing well , I am going to show how you can make your own CTF or if you have an idea to introduce a vulnerability that you want to teach people that shows how severe that  vulnerability is and how you can exploit it so you can do it by making a box and then configure it according to the vulnerability. So I am going to show a basic example of how you can make it. Its' a basic idea that I come up with when I was creating my first vulnerable box.

Now the one which I am creating there won't be anything new because I will be using an old version of ubuntu which is vulnerable to dirty cow exploit but the important thing is that how you may configure the box , opening FTP,SSH and HTTP services on it or maybe changing the port number.

#### 1.Research

First of all research what vulnerability you want someone to exploit. Look up on exploit-db if you want to find any vulnerability.

<img src="https://i.imgur.com/WkrD3Oc.png"/>

So there is an exploit for an older version of Linux kernel , we might have to install an older version ubuntu for this to work.

Visit `http://old-releases.ubuntu.com/releases/` if you want to download old releases of ubuntu.

<img src="https://i.imgur.com/sDad2QF.png"/>

<img src="https://i.imgur.com/NtGg6R8.png"/>

Now we want the server image because it will give us an option to use GUI or CLI and we only want to be working with CLI.

#### 2.Installation

To install this image you can use VMware or Virtual box but I recommend you to use virtual box because it is easy to export the OVA file that you later will upload on some CTF platforms.

<img src="https://i.imgur.com/1aQmeIh.png"/>

<img src="https://i.imgur.com/Dvrt4mt.png"/>

Now let it go through the installation process.

<img src="https://i.imgur.com/5ECYszl.png"/>

Select `No automatic updates`

<img src="https://i.imgur.com/4RWVEoG.png"/>

Now here you can install some services if you want to save time, you can however install them later.

I am selecting OpenSSH server as we can ssh into the box, LAMP server which is Linux, Apache, MySQL and PHP server and Samba is for file sharing.

<img src="https://i.imgur.com/Ob87p23.png"/>

Now that you have the OS installed, enable the root login through this command `sudo -i passwd root`

<img src="https://i.imgur.com/u0jhaZI.png"/>

####  3.Configuration

Now you want to have some tools to be installed on the box first of all update the repository through which it will fetch the packages

`apt update`

Since we will exploit this box through dirty cow and it's written in C language so we need a compiler for this which is `gcc` and we want to install it `apt install gcc`.

<img src="https://i.imgur.com/qXnjnMH.png"/>

<img src="https://i.imgur.com/LZOvUyI.png"/>

We have all the tools we need to exploit this box. In order to place flags just type any text and pipe into md5sum to get a md5 hash and store it in files but make sure to give it permissions that only specific user can read those flags.

<img src="https://i.imgur.com/8gxU0pZ.png"/>

<img src="https://i.imgur.com/4ql8Yrs.png"/>

Remember to take a snapshot of your box so if you screw up at any point you may revert it back so you won't have to install everything from the beginning.

Before we setup other services or the way to get a foothold let's test privilege escalation exploit that is it working or not .Visit the same site where you find the exploit have it on your machine 

<img src="https://i.imgur.com/TBV7Q8b.png"/>

<img src="https://i.imgur.com/mv8pNFY.png"/>

We escalated our privileges to root through this exploit so it worked as expected.

###### 3.1 Configure FTP 
If you want an ftp server on your box you can do it by installing `vsftpd`

<img src="https://i.imgur.com/b0Tw8Os.png"/>

Make a backup of the configuration file of vsftpd

`cp /etc/vsftpd.conf /etc/vsftpd.conf.orig` (use sudo before the command if not root)

<img src="https://i.imgur.com/gOS67rW.png"/>

Make sure to enable this option if you want anonymous user to login into ftp also add two more commands

<img src="https://i.imgur.com/9yKL7bQ.png"/>

The first one will tell that where the root directory of ftp is and other is for not prompting password for `anonymous` user.

Make the folder for ftp in the same directory as we have setup in the configuration file of vsftpd and change it's owner to `nobody:nogroup`.

<img src="https://i.imgur.com/qS7qY4G.png"/>

Now you can add a file to the `ftp` folder you want the anonymous user to read also if you want to change the default ftp port this can be done by adding this `listen_port=XX` to configuration where "XX" is just the number you can give.

<img src="https://i.imgur.com/27QyH4L.png"/>

Here I added a file to check if we login as `anonymous` we should be in the `/var/ftp` directory.

We also want to allow FTP traffic because it is blocked by default so we can easily do that

<img src="https://i.imgur.com/lYw6N8J.png"/>

Now to enable ftp service just type
```
sudo service vsftp start 
```

or your running as root then

```
service vsftpd start
```

<img src="https://i.imgur.com/GtB8FLu.png"/>

Now let's try connecting to ftp service

<img src="https://i.imgur.com/dA940Bx.png"/>

As you can see we get the file that is `/var/ftp` directory. We can do a lot play around but this is the basic idea of how we can setup a ftp server. 

That's pretty much all the configuration I'll do with ftp now let's move on to configuring the http server.

##### 3.2 Configure Apache HTTP server

Since we already have installed apache when we were given an option of istalling LAMP server so all we have to do is  allow ufw (Uncomplicated Firewall) to enable traffic for http or port 80

```
ufw allow http
```

Or

```
ufw allow 80/tcp
```

<img src="https://i.imgur.com/RYlR8hv.png"/>

By default we will have an `index.html` page in directory `var/www/html`

<img src="https://i.imgur.com/IB2mnFf.png"/>

<img src="https://i.imgur.com/3BrWKcb.png"/>

Now let's see if we can run php on web pages or make sure that we have php installed which we would have.

<img src="https://i.imgur.com/mt6PjGy.png"/>

<img src="https://i.imgur.com/RsCr68x.png"/>

Important thing to take care of is that those webserver files must belong to user and group `www-data`:`www-data`not the `root`:`root` user to ensure that 

<img src="https://i.imgur.com/KkFbeo6.png"/>

Now let's change the permissons

<img src="https://i.imgur.com/aFV5jkv.png"/>

Here `-R` will recursively change ownership of files with in directory.

Inorder to create page vulnerable to RCE we need to do something like this

<img src="https://i.imgur.com/dkQWTGu.png"/>

The first line is just a simple HTML heading which tells that we have paramter named `cmd` and the next line is the *bad code* which will give us command execution because parameter is inside a `system` function which will execute system commands.

<img src="https://i.imgur.com/zxjemi1.png"/>

##### 3.3 Configure SSH

For ssh we must have installed  OpenSSH  but it was done at the beginning so we don't need to do that again. We can check it's version

<img src="https://i.imgur.com/haWT4Lt.png"/>

And like we did with FTP and HTTP just need to allow traffic for SSH as well

<img src="https://i.imgur.com/dVCvgrF.png"/>

<img src="https://i.imgur.com/M8xxxpL.png"/>

Make sure to test it with your host system if your running windows then test it with `PuTTY` or your on linux then you could type `ssh username@ip`  if it does not work you will need to allow the traffic for ssh as shown above.

This uses password base authenticattion you could just disable it and allow to login with `public key-based authentication` but for now let's keep it as password based.

Now one last thing it's optional not really necessary but when you will boot this machine in an OVA format it's better to modify the `motd` and replace it with machine IP that it will get. To do that first go to `/etc/update-motd.d` directory and remove all executable permissions from the scripts and make your own custom script

<img src="https://i.imgur.com/xKehYbN.png"/>

<img src="https://i.imgur.com/nAxB3DI.png"/>

<img src="https://i.imgur.com/ejBCY5Y.png"/>

And in order to turn off `.bash_history` just create a sylink to `/dev/null`

```
symlink -sf /dev/null ~/.bash_history
```

#### 4. Exporting the VM to an OVA format

Now when you have tested the machine if it's working as intended and when you ran scripts for enumeration gives what you want them to do (although I haven't showed running them) you then have to export it to an OVA format so you could then upload to submit and then user can download and import that machine to thier virtualbox or VMware. 

Go to `File > Export Appliance`
<img src="https://i.imgur.com/PpEfrUw.png"/>

Then select your vulnerable machine in this case mine is `Ubuntu 14.04 Server`

<img src="https://i.imgur.com/krfFaX6.png"/>

<img src="https://i.imgur.com/1Feg8tO.png"/>

There were many things we could have done like web application with flas,django or maybe setting up node js server , setting up nfs maybe rabbit holes but those all depends upon your thinking your idea for the vulnerable machine this was just a very basic machine that I made maybe you learned something from.

## References
https://linuxconfig.org/how-to-change-welcome-message-motd-on-ubuntu-18-04-server

https://websiteforstudents.com/setup-apahce2-with-php-support-on-ubuntu-servers/

https://phoenixnap.com/kb/install-ftp-server-on-ubuntu-vsftpd

https://www.exploit-db.com/exploits/37292

https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-14-04#:~:text=Check%20UFW%20Status%20and%20Rules,sudo%20ufw%20status%20verbose
