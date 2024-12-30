# Apt pinning

This is a short overview of how to use apt-pinning on Debian. 
## Preliminary Note

I'm using a Debian bullseye (stable) system here. I will explain apt-pinning on the basis of the package phpmyadmin which is available in in the stable, testing, and unstable repositories - see http://packages.debian.org/search?keywords=phpmyadmin&searchon=names&suite=all&section=all:
  * bulsseye (stable): version 4:5.0.4
  * bookworm (testing): version 4:5.1.1
  * sid (unstable): version 4:5.1.1
## sources.list
Add the testing and unstable repositories to /etc/apt/sources.list so that it looks as follows:
```
vi /etc/apt/sources.list
```
```
## Stable
deb http://ftp.belnet.be/debian/ stable main contrib non-free
deb http://security.debian.org/debian-security stable-security main
deb http://ftp.belnet.be/debian/ stable-updates main contrib non-free
## Testing
deb http://ftp.belnet.be/debian/ testing main contrib non-free
deb http://security.debian.org/debian-security testing-security main
deb http://ftp.belnet.be/debian/ testing-updates main contrib non-free
## Sid / Unstable
deb http://ftp.belnet.be/debian/ sid main contrib non-free
```
Open /etc/apt/apt.conf...
```
vi /etc/apt/apt.conf
```
... and put the following line into it, so you don't get the error Dynamic MMap ran out of room:
```
APT::Cache-Limit "100000000";
```
Then run
```
apt update
```
to update the package database.

With the current version, apt would always try to install the newest version of a package which usually comes from unstable or testing - this could lead to a messed-up system. With apt-pinning, we can define priorities so that a package gets installed from unstable or testing only if there's no such package from stable.\\
We can check apt priorities with the command apt policy. The Candidate: line shows the version that would be installed.
```
apt policy phpmyadmin
```
As you can see, stable, testing, and unstable all have the same priority (500) which means that the newest version of a package would be installed. In the case of our phpmyadmin package this is from unstable:
```
root@penguin:~# apt policy phpmyadmin
phpmyadmin:
  Installed: (none)
  Candidate: 4:5.1.1+dfsg1-4
  Version table:
     4:5.1.1+dfsg1-4 500
        500 http://ftp.belnet.be/debian bookworm/main amd64 Packages
        500 http://ftp.belnet.be/debian sid/main amd64 Packages
     4:5.0.4+dfsg2-2 500
        500 http://ftp.belnet.be/debian bullseye/main amd64 Packages
```

This is how priorities are defined (see man 5 apt_preferences):
  * P > 1000: causes a version to be installed even if this constitutes a downgrade of the package
  * 990 < P < 1000: causes a version to be installed even if it does not come from the target release, unless the installed version is more recent
  * 500 < P < 990: causes a version to be installed unless there is a version available belonging to the target release or the installed version is more recent
  * 100 < P < 500: causes a version to be installed unless there is a version available belonging to some other distribution or the installed version is more recent
  * 0 < P < 100: causes a version to be installed only if there is no installed version of the package
  * P < 0: prevents the version from being installed

## Apt-Pinning

Now let's define that we prefer stable packages over testing packages and testing packages over unstable packages, and that packages from testing and unstable get installed only if there's no such package from stable:
```
vi /etc/apt/preferences
```
```
Package: *
Pin: release a=stable
Pin-Priority: 700

Package: *
Pin: release a=stable-security
Pin-Priority: 700

Package: *
Pin: release a=stable-updates
Pin-Priority: 700

Package: *
Pin: release a=testing
Pin-Priority: 650

Package: *
Pin: release a=unstable
Pin-Priority: 600

Package: *
Pin: release o=Docker
Pin-Priority: 700
```

## Holding A Package

Let's assume you've installed the phpmyadmin package from stable, and you want to tell apt to keep the current package under all circumstances. This is how you tell apt to never ever install another phpmyadmin package again:
```
vi /etc/apt/preferences
```
```
Package: phpmyadmin
Pin: version 4:5.1.1*
Pin-Priority: 1001
```
(This works with or without the stable, testing, and unstable stanzas from the previous chapter. So even if stable, testing, and unstable have the same priority, phpmyadmin is set to hold.)
 

## Installing Packages From A Specific Release/Overriding Priorities

Let's assume you prefer stable over testing and testing over unstable, but just not for the keepalived package because the version from unstable has a feature that you absolutely need. So this is how you override your previously set up priorities and install phpmyadmin from unstable.

There are two ways of doing this, with a slight difference:
```
apt install keepalived/unstable
```

This would install keepalived from unstable, but apt would try to satisfy all keepalived dependencies with packages from stable.
```
apt-get -t unstable install keepalived
```
This would install keepalived from unstable, and all keepalived dependencies would also be installed from unstable.
