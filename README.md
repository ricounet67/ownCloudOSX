#ownCloud & Nginx on OS X

OwnCloud is the ultimate tool for people that want to host their own cloud storage service, according to documentation it's batter to run it on linux-based systems, i have it running on my macmini for a time now and haven't had any issues yet.

This tutorial gathers information from various sources that helped to have running ownCloud on OS X.

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

> HTTP/1.1 200 OK

> Server: nginx/1.6.2

> Date: Mon, 19 Oct 2014 19:07:47 GMT

> Content-Type: text/html

> Content-Length: 612

> Last-Modified: Mon, 19 Oct 2014 19:01:32 GMT

> Connection: keep-alive

> ETag: "5444dea7-264"

> Accept-Ranges: bytes
