Installation
============

Create user "bodl-iip-svc"
------------------

```bash
sudo useradd bodl-iip-srv
sudo passwd bodl-iip-srv
sudo mkdir -p /home/bodl-iip-srv/.ssh
cd /home
sudo chown -R bodl-iip-srv:bodl-iip-srv bodl-iip-srv/
sudo chsh -s /bin/bash bodl-iip-srv
su - bodl-iip-srv
ssh-keygen -t rsa
```

Copy and paste your key into gitlab by choosing My Profile (the grey person graphic link in the top right hand corner) then Add Public Key.

```bash
cat ~/.ssh/id_rsa.pub
```

Install and configure Git (Ubuntu)
----------------------------------
```bash
su - <sudo user>
sudo apt-get install git
```
```bash
git config --global user.email "my@address.com"
git config --global user.name "name in quotes"
```
Install and configure Git (RHEL>=6)
-----------------------------------
```bash
su
yum install git
exit
```
```bash
git config --global user.email "my@address.com"
git config --global user.name "name in quotes"
```

Checkout the buildout
---------------------
```bash
su - bodl-iip-srv
mkdir -p ~/sites/bodl-iip-srv
cd ~/sites/bodl-iip-srv
git clone gitlab@source.bodleian.ox.ac.uk:loris/buildout.iip.git ./
```
Setup server (Debian/Ubuntu)
----------------------------

```bash
su - <sudo user>
sudo apt-get install $(cat /home/bodl-iip-srv/sites/bodl-iip-srv/ubuntu_requirements)
su - bodl-iip-srv
```
Setup server (RHEL>=6)
----------------------------

```bash
su - <sudo user>
sudo cp /home/bodl-iip-srv/sites/bodl-iip-srv/redhat_requirements ~
sudo yum install $(cat redhat_requirements)
exit
```

Install Python
--------------
```bash
su - bodl-iip-srv
mkdir -p ~/Downloads
cd ~/Downloads
wget http://www.python.org/ftp/python/2.7.6/Python-2.7.6.tgz --no-check-certificate
tar zxfv Python-2.7.6.tgz
cd Python-2.7.6
./configure --prefix=$HOME/python/2.7.6 --enable-unicode=ucs4 --enable-shared LDFLAGS="-Wl,-rpath=/home/bodl-iip-srv/python/2.7.6/lib"
make
make install
cd ..
wget http://python-distribute.org/distribute_setup.py
~/python/2.7.6/bin/python distribute_setup.py
~/python/2.7.6/bin/easy_install pip
~/python/2.7.6/bin/pip install virtualenv
```

Setup the buildout cache
------------------------
```bash
mkdir ~/.buildout
cd ~/.buildout
mkdir eggs
mkdir downloads
mkdir extends
echo "[buildout]
eggs-directory = /home/bodl-iip-srv/.buildout/eggs
download-cache = /home/bodl-iip-srv/.buildout/downloads
extends-cache = /home/bodl-iip-srv/.buildout/extends" >> ~/.buildout/default.cfg
```
Change the IP address for apache config
---------------------------------------

edit development or production.cfg:

```bash

[hosts]
internalIP = <your server internal IP address>
externalIP = <your server external IP address>
```

Upload Kakadu source to server for compilation
----------------------------------------------

With no top layer directory (this buildout is designed for Kakadu versions 6.4 to 7.4).

On your loris server:

```bash
mkdir ~/Downloads/kakadu
```

From wherever your source files reside:

```bash
scp -r <kakadu source location>/* bodl-iip-srv@<your loris server>:/home/bodl-iip-srv/Downloads/kakadu
```

Buildout will compile the source and distribute the libraries and applications required (namely the shared object library and kdu_expand).

Create a virtualenv and run the buildout
----------------------------------------
```bash
cd ~/sites/bodl-iip-srv
~/python/2.7.6/bin/virtualenv .
source bin/activate
pip install zc.buildout
pip install distribute
buildout init
buildout -c development.cfg
```

Test images
-----------

Copy the test images into your image root:

```bash 
su - bodl-iip-srv
cp -R /home/bodl-iip-srv/sites/bodl-iip-srv/src/loris/tests/img/* /home/bodl-iip-srv/sites/bodl-iip-srv/var/images
```

Start Apache
------------

From within the virtual environment:

```bash
su - bodl-iip-srv
cd ~/sites/bodl-iip-srv
. bin/activate
/home/bodl-iip-srv/sites/bodl-iip-srv/parts/apache/bin/apachectl start
```

Reboot scripts
--------------

None are necessary. Just restart apache.