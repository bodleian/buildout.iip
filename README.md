Introduction
============

This Loris build is intended for Ubuntu 12.0, IIP 0.9.9 and Kakadu 7.2. These versions can be changed, see ```development[_docker].cfg``` and ```Dockerfile```. 

BDLSS Docker CI
https://registry.hub.docker.com/u/calvinbutcher/buildout.iip/

Github IIP source https://github.com/ruven/iipsrv

Installation
============

To deploy IIP on a server, follow these instructions. Whenever this GIT account is updated, Docker will run a test deployment at ```https://registry.hub.docker.com/u/calvinbutcher/buildout.iip/```. Please see **Continuous Integration** section below for more details.

Create user "bodl-iip-svc"
--------------------------

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

Checkout the buildout
---------------------
```bash
su - bodl-iip-srv
mkdir -p ~/sites/bodl-iip-srv
cd ~/sites/bodl-iip-srv
git clone https://github.com/BDLSS/buildout.iip.git ./
```
Setup server 
------------

```bash
su - <sudo user>
sudo apt-get install $(cat /home/bodl-iip-srv/sites/bodl-iip-srv/ubuntu_requirements)
su - bodl-iip-srv
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
wget https://pypi.python.org/packages/source/d/distribute/distribute-0.6.49.tar.gz
tar zxfv distribute-0.6.49.tar.gz
~/python/2.7.6/bin/python distribute-0.6.49/distribute_setup.py
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

Create a virtualenv and run the buildout
----------------------------------------

Add _docker to development.cfg if running in docker environment (or remove [...] from code below).

```bash
cd ~/sites/bodl-iip-srv
~/python/2.7.6/bin/virtualenv .
. bin/activate
pip install zc.buildout
pip install distribute
buildout init
buildout -c development[_docker].cfg
```

Upload Kakadu source to server for compilation
----------------------------------------------

You can retrieve the source from databank (you will need a user account for databank):

```bash
cd ~/Downloads
curl --user <username>:<password> -o Kakadu_v72.zip https://databank.ora.ox.ac.uk/dmt/datasets/Kakadu/Kakadu_v72.zip 
unzip -d kakadu Kakadu_v72.zip
```

Otherwise you will need to ```scp```, ```wget``` or ```curl``` your licensed Kakadu source into the ```~/Downloads``` directory as a folder called 'kakadu'.

Add the following to ~/Downloads/kakadu/managed/make/Makefile-Linux-x86-64-gcc (in place of the non-specific java include directives there)

```bash
INCLUDES += -I/usr/lib/jvm/java-7-openjdk-amd64/include       # or wherever the Java
INCLUDES += -I/usr/lib/jvm/java-7-openjdk-amd64/include/linux # includes are on your system
```

If you later get a 'undefined reference to ssse3...' error, add the following at the end of ~/Downloads/kakadu/apps/make/Makefile-Linux-x86-64-gcc:

```bash
DEFINES += -DKDU_NO_SSSE3
```

And compile...

```bash
cd ~/Downloads/kakadu/make
make -f Makefile-Linux-x86-64-gcc
```

If there are no errors, compile the IIP server.

Compile IIP server
-----------------

Make sure there is no trailing slash in the --with-kakadu param value; also, that it is an absolute path.

```bash
cd ~/sites/bodl-iip-srv/src/iipsrv
./autogen.sh
./configure --with-kakadu=/home/bodl-iip-srv/Downloads/kakadu
make 
```

Test images
-----------

```bash
mkdir -p /home/bodl-iip-srv/sites/bodl-iip-srv/var/images
```

Copy your ```.tif``` and ```.jp2``` images into this directory. 

e.g. 

```bash
cd /home/bodl-iip-srv/sites/bodl-iip-srv/var/images
wget http://iiif-test.stanford.edu/67352ccc-d1b0-11e1-89ae-279075081939.jp2
wget http://merovingio.c2rmf.cnrs.fr/iipimage/PalaisDuLouvre.tif
```

Check their permissions or the viewer may hang!

Amend MooViewer image path
--------------------------

```bash
vi /home/bodl-iip-srv/sites/bodl-iip-srv/src/www/index.html
```

Amend the parameter as follows:

```bash
var image = /home/bodl-iip-srv/sites/bodl-iip-srv/var/images/<image name>
```

e.g.

```bash
var image = /home/bodl-iip-srv/sites/bodl-iip-srv/var/images/PalaisDuLouvre.tif
```

Start Apache
------------

```bash
su - <sudo user>
sudo chmod +x /home/bodl-iip-srv/sites/bodl-iip-srv/bin/iipctl
sudo /home/bodl-iip-srv/sites/bodl-iip-srv/bin/iipctl start
```

Browse to http://&lt;your server&gt;:8080/index.html

The IIP image server is located at (You should receive a welcome screen at this URL):

http://&lt;your server&gt;:8080/fcgi-bin/iipsrv.fcgi

If there is something wrong, check the logs at ```/home/bodl-iip-srv/sites/bodl-iip-srv/parts/iipsrv/logs/error.log```

Setup the reboot script in the sudo crontab
-------------------------------------------

```bash
su - <sudo user>
sudo crontab /home/bodl-iip-svc/sites/bodl-iip-svc/bin/cron.txt
su - bodl-iip-svc
```

Startup scripts and cron jobs
-----------------------------

The following script can be run manually. 

```bash
su - <sudo user>
/home/bodl-iip-svc/sites/bodl-iip-svc/bin/iipctl [start|stop|restart]
```

It will stop/start/restart iip. It runs under a @reboot directive in the sudo crontab to ensure the service comes back up in the event of a server shutdown/restart. It logs progress in ```var/log/reboot.log```.

```bash
@reboot /home/bodl-iip-svc/sites/bodl-iip-svc/bin/iipctl start > /home/bodl-iip-svc/sites/bodl-iip-svc/var/log/reboot.log 2>&1
```

Continuous Integration
----------------------

The Dockerfile will run the ```_docker.cfg``` version of development.cfg. This just ensures that users are named properly (the 'env' recipe does not work inside containers) and that the localhost is pointed to all IPs (as this cannot be dictated or predicted when creating a container).

Docker
https://registry.hub.docker.com/u/calvinbutcher/buildout.iip/

If any of the 21 IIIF validation tests fail, Docker will exit with a non-zero result. This means the Docker build will fail and read "Error".

Functional and Unit Testing
---------------------------

Pytest is executed in the docker run.

This runs all test scripts using the filename format of ``test_<something>.py`` in the ``tests/`` folder.