#ownCloud & Nginx on OS X El Capitan

OwnCloud is the ultimate tool for people that want to host their own cloud storage service, according to documentation it's better to run it on linux-based systems, although i have it running on my macmini for a few time now and haven't had any issues yet.

This tutorial gathers information from various sources that helped to have running ownCloud on OS X El Capitan (v10.11.5).

##XCode Command Line Tools
First of all we need to install the XCode Command Line Tools, to do so type the next command on a terminal to install the lastes version
```
xcode-select --install
```

##Homebrew
This is a package manager for OS X, kind of like `apt` is one for Linux. `brew` works the same, just for Mac operating systems. It will make sure that you receive the latest updates of your installed packages, so you don't need to worry about outdated versions or vulnerable security flaws, etc.

First, you need to download and install Homebrew using the following command:
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Check for any conflicts or problems (If you have conflicts, sort them out before you continue with this guide):
```
brew doctor
```
Make sure the doctor responds with something along the lines:
> Your system is ready to brew.

In case you already had Homebrew installed, update the existing Homebrew installation as well as the installed packages:
```
brew update && brew upgrade
```

##Nginx

Install the default Nginx with:
```
brew install nginx
```
###Setup auto start
Since you want to use port 80 (default HTTP port), you have to run the Nginx process with root privileges:
```
sudo cp -v /usr/local/opt/nginx/*.plist /Library/LaunchDaemons/
sudo chown root:wheel /Library/LaunchDaemons/homebrew.mxcl.nginx.plist
```
(Only the root user is allowed to open listening ports < 1024)

###Test web server
Start Nginx with:
```
sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.nginx.plist
```
The default configuration currently active so it will listen on port 8080 instead of the HTTP default port 80. Ignore that for now:
```
curl -IL http://127.0.0.1:8080
```
The output should look like this:
```
HTTP/1.1 200 OK
Server: nginx/1.6.2
Date: Mon, 19 Oct 2014 19:07:47 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Mon, 19 Oct 2014 19:01:32 GMT
Connection: keep-alive
ETag: "5444dea7-264"
Accept-Ranges: bytes
```
Stop Nginx again:
```
sudo launchctl unload /Library/LaunchDaemons/homebrew.mxcl.nginx.plist
```
###More configuration
####nginx.conf
Create these bunch of folders which we're going to use for the upcoming configuration:
```
mkdir -p /usr/local/etc/nginx/logs
mkdir -p /usr/local/etc/nginx/sites-available
mkdir -p /usr/local/etc/nginx/sites-enabled
mkdir -p /usr/local/etc/nginx/conf.d
mkdir -p /usr/local/etc/nginx/ssl
sudo mkdir -p /var/www
sudo chown :staff /var/www
sudo chmod 775 /var/www
```
This nginx folder distribution we just created use to be the same in linux systems.

##PHP-FPM
Since ownCloud is written in PHP an interpreter for it is needed in order to make it run alongside Nginx.

Homebrew doesn't come with a formula for PHP-FPM by default, you need to `tap` (or register) a special PHP repository first:
```
brew tap homebrew/dupes
brew tap homebrew/php
brew doctor
brew update
```
Now you can install PHP using the following command:

If you want to use MySQL, use this command, the arguments make sure it compiles with MySQL support and doesn't configure the default Apache:
```
brew install --without-apache --with-fpm --with-mysql php56
```

The lightest installation is to use SQLite, because SQLite is a serverless database and it doesn't use so many resources of the computer.
In case you want to use SQLite just use the following command
```
brew install --without-apache --with-fpm php56
```

Homebrew is now going to download and compile the PHP-FPM source code for you. Give it some time, it could take some minutes.

###Setup PHP CLI binary
If you want to use the PHP command line tools, you need to update the `$PATH` environment variable of your shell profile.

If you use the default bash shell:
```
echo 'export PATH="/usr/local/sbin:$PATH"' >> ~/.bash_profile && . ~/.bash_profile
```

IF you use ZSH:
```
echo 'export PATH="/usr/local/sbin:$PATH"' >> ~/.zshrc && . ~/.zshrc
```

If you are not sure which one you use, run `echo $SHELL` in a Terminal.app window. Since I use ZSH it returns this:
> /bin/zsh

###Setup auto start

Create a folder for the LaunchAgents:
```
mkdir -p ~/Library/LaunchAgents
```
Once done that or if the folder already exists, create a symlink the start/stop service:
```
ln -sfv /usr/local/opt/php56/homebrew.mxcl.php56.plist ~/Library/LaunchAgents/
```
Now try to start PHP-FPM:
```
launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.php56.plist
```
Assure PHP-FPM is running. To do so, check if there is an open listener on port `9000`:
```
lsof -Pni4 | grep LISTEN | grep php
```
The output should look something like this:
```
php-fpm   69659  frdmn    6u  IPv4 0x8d8ebe505a1ae01      0t0  TCP 127.0.0.1:9000 (LISTEN)
php-fpm   69660  frdmn    0u  IPv4 0x8d8ebe505a1ae01      0t0  TCP 127.0.0.1:9000 (LISTEN)
php-fpm   69661  frdmn    0u  IPv4 0x8d8ebe505a1ae01      0t0  TCP 127.0.0.1:9000 (LISTEN)
php-fpm   69662  frdmn    0u  IPv4 0x8d8ebe505a1ae01      0t0  TCP 127.0.0.1:9000 (LISTEN)
```

###Additional modules 
Install some additional PHP Modules which will be needed for ownCloud later:
```
brew install -v php56-opcache
brew install php56-apcu
brew install mcrypt
brew install php56-mcrypt
brew install php56-intl
```

###Config File
Edit `php.ini` Config-File
```
nano -w /usr/local/etc/php/5.6/php.ini
```
And look for the following parameters. Set the Parameters to your needs – the Explanation of each Parameter can be found next to the Parameter. This are my settings:
```
upload_max_filesize = 250M
max_file_uploads = 50
default_charset = "UTF-8"
post_max_size = 0
default_socket_timeout = 3000
mysql.connect_timeout = 120
error_log = /usr/local/var/log/php-error.log
```

In my case i'm the only person using my ownCloud services, so i reduced the processes by setting them static and setting a fixed number for it. 
Edit `php-fpm.conf` File
```
nano -w /usr/local/etc/php/5.6/php-fpm.conf
```
And look for the following parameters. Set the Parameters to your needs – the Explanation of each Parameter can be found next to the Parameter. This are my settings:
```
pm = static
pm.max_children = 1
```
The processes are set dynamic by default and they come with some parameters enabled in order to work dynamically, you need to comment out these paramaters, to do so write a `;` before them:
```
; pm.start_servers = 3
; pm.min_spare_servers = 2
; pm.max_spare_servers = 5
```

##Databases

###SQLite
