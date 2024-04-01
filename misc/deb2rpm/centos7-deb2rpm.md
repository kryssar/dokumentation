---
title: CentOS 7 - Converting .deb to .rpm
date: "2022-02-10"
description: How to convert a .deb package to .rpm
tldr: deb 2 rpm
tags: [linux,deb,rpm,convert]
---

## CentOS 7 - Converting .deb to .rpm
Sometimes there are packages which are lagging behind on certain distributions but are newer in other. Because of this it's sometimes necessary to convert a package in order to try it out in the testing environment. 


Inspired by/from:  
[fedingo](https://fedingo.com/how-to-convert-deb-to-rpm-files-in-linux/)  
[thegeekstuff](https://www.thegeekstuff.com/2010/11/alien-command-examples/)

## Install alien
```shell
sudo yum install -y alien 

Dependencies Resolved

================================================================================================================
 Package                                Arch            Version                          Repository        Size
================================================================================================================
Installing:
 alien                                  noarch          8.95-3.el7                       epel              92 k
Installing for dependencies:
 autoconf                               noarch          2.69-11.el7                      base             701 k
 automake                               noarch          1.13.4-3.el7                     base             679 k
 cpp                                    x86_64          4.8.5-44.el7                     base             5.9 M
 debconf                                noarch          1.5.79-1.el7                     epel             189 k
 debhelper                              noarch          11.4-2.el7                       epel             939 k
 dpkg                                   x86_64          1.18.25-10.el7                   epel             1.3 M
 dpkg-dev                               noarch          1.18.25-10.el7                   epel             706 k
 dpkg-perl                              noarch          1.18.25-10.el7                   epel             248 k
 dwz                                    x86_64          0.11-3.el7                       base              99 k
 gcc                                    x86_64          4.8.5-44.el7                     base              16 M
 gettext-common-devel                   noarch          0.19.8.1-3.el7                   base             410 k
 gettext-devel                          x86_64          0.19.8.1-3.el7                   base             320 k
 git                                    x86_64          1.8.3.1-23.el7_8                 base             4.4 M
 glibc-devel                            x86_64          2.17-325.el7_9                   updates          1.1 M
 glibc-headers                          x86_64          2.17-325.el7_9                   updates          691 k
 intltool                               noarch          0.50.2-7.el7                     base              59 k
 kernel-headers                         x86_64          3.10.0-1160.53.1.el7             updates          9.0 M
 m4                                     x86_64          1.4.16-10.el7                    base             256 k
 patch                                  x86_64          2.7.1-12.el7_7                   base             111 k
 perl-Business-ISBN                     noarch          2.06-2.el7                       base              25 k
 perl-Business-ISBN-Data                noarch          20120719.001-2.el7               base              24 k
 perl-CGI                               noarch          3.63-4.el7                       base             250 k
 perl-Compress-Raw-Bzip2                x86_64          2.061-3.el7                      base              32 k
 perl-Compress-Raw-Zlib                 x86_64          1:2.061-4.el7                    base              57 k
 perl-Convert-BinHex                    noarch          1.119-20.el7                     epel              44 k
 perl-Data-OptList                      noarch          0.107-9.el7                      base              23 k
 perl-Devel-GlobalDestruction           noarch          0.12-1.el7                       epel              18 k
 perl-Digest-SHA3                       x86_64          0.24-1.el7                       epel              32 k
 perl-Email-Date-Format                 noarch          1.002-15.el7                     epel              17 k
 perl-Email-Simple                      noarch          2.214-1.el7                      epel              36 k
 perl-Encode-Locale                     noarch          1.03-5.el7                       base              16 k
 perl-Error                             noarch          1:0.17020-2.el7                  base              32 k
 perl-FCGI                              x86_64          1:0.74-8.el7                     base              42 k
 perl-File-FcntlLock                    x86_64          0.22-6.el7                       epel              41 k
 perl-File-Listing                      noarch          6.04-7.el7                       base              13 k
 perl-File-Remove                       noarch          1.52-6.el7                       base              26 k
 perl-Font-AFM                          noarch          1.20-13.el7                      base              19 k
 perl-Git                               noarch          1.8.3.1-23.el7_8                 base              56 k
 perl-HTML-Format                       noarch          2.10-7.el7                       base              51 k
 perl-HTML-Parser                       x86_64          3.71-4.el7                       base             115 k
 perl-HTML-Tagset                       noarch          3.20-15.el7                      base              18 k
 perl-HTML-Tree                         noarch          1:5.03-2.el7                     base             218 k
 perl-HTTP-Cookies                      noarch          6.01-5.el7                       base              26 k
 perl-HTTP-Daemon                       noarch          6.01-8.el7                       base              21 k
 perl-HTTP-Date                         noarch          6.02-8.el7                       base              14 k
 perl-HTTP-Message                      noarch          6.06-6.el7                       base              82 k
 perl-HTTP-Negotiate                    noarch          6.01-5.el7                       base              17 k
 perl-IO-Compress                       noarch          2.061-2.el7                      base             260 k
 perl-IO-HTML                           noarch          1.00-2.el7                       base              23 k
 perl-IO-Socket-IP                      noarch          0.21-5.el7                       base              36 k
 perl-IO-Socket-SSL                     noarch          1.94-7.el7                       base             115 k
 perl-LWP-MediaTypes                    noarch          6.02-2.el7                       base              24 k
 perl-MIME-Lite                         noarch          3.030-1.el7                      epel              96 k
 perl-MIME-Types                        noarch          1.38-2.el7                       epel              38 k
 perl-MIME-tools                        noarch          5.505-1.el7                      epel             256 k
 perl-Mail-Box                          noarch          2.120-2.el7                      epel             1.1 M
 perl-Mail-IMAPClient                   noarch          3.37-1.el7                       epel             221 k
 perl-Mail-Sendmail                     noarch          0.79-21.el7                      epel              29 k
 perl-Mail-Transport-Dbx                x86_64          0.07-28.el7                      epel              41 k
 perl-MailTools                         noarch          2.12-2.el7                       base             108 k
 perl-Mozilla-CA                        noarch          20130114-5.el7                   base              11 k
 perl-Net-HTTP                          noarch          6.06-2.el7                       base              29 k
 perl-Net-LibIDN                        x86_64          0.12-15.el7                      base              28 k
 perl-Net-SMTP-SSL                      noarch          1.01-13.el7                      base             9.1 k
 perl-Net-SSLeay                        x86_64          1.55-6.el7                       base             285 k
 perl-Object-Realize-Later              noarch          0.19-6.el7                       epel              21 k
 perl-Package-Generator                 noarch          0.103-14.el7                     base              23 k
 perl-Params-Util                       x86_64          1.07-6.el7                       base              38 k
 perl-Parse-RecDescent                  noarch          1.967009-5.el7                   base             203 k
 perl-Sub-Exporter                      noarch          0.986-2.el7                      base              70 k
 perl-Sub-Exporter-Progressive          noarch          0.001011-1.el7                   epel              13 k
 perl-Sub-Install                       noarch          0.926-6.el7                      base              21 k
 perl-TeX-Hyphen                        noarch          1.01-1.el7                       epel              35 k
 perl-TermReadKey                       x86_64          2.30-20.el7                      base              31 k
 perl-Test-Harness                      noarch          3.28-3.el7                       base             302 k
 perl-Text-Autoformat                   noarch          1.669004-1.el7                   epel              37 k
 perl-Text-CharWidth                    x86_64          0.04-18.el7                      base              15 k
 perl-Text-Iconv                        x86_64          1.7-18.el7                       base              23 k
 perl-Text-Reform                       noarch          1.20-9.el7                       epel              43 k
 perl-Text-WrapI18N                     noarch          0.06-17.el7                      base              12 k
 perl-Thread-Queue                      noarch          3.02-2.el7                       base              17 k
 perl-Time-Piece                        x86_64          1.20.1-299.el7_9                 updates           70 k
 perl-TimeDate                          noarch          1:2.30-2.el7                     base              52 k
 perl-URI                               noarch          1.60-9.el7                       base             106 k
 perl-User-Identity                     noarch          0.96-1.el7                       epel              88 k
 perl-WWW-RobotRules                    noarch          6.02-5.el7                       base              18 k
 perl-XML-Parser                        x86_64          2.41-10.el7                      base             223 k
 perl-gettext                           x86_64          1.05-28.el7                      base              21 k
 perl-libwww-perl                       noarch          6.05-2.el7                       base             205 k
 perl-srpm-macros                       noarch          1-8.el7                          base             4.6 k
 po-debconf                             noarch          1.0.20-5.el7                     epel             147 k
 python-srpm-macros                     noarch          3-34.el7                         base             8.8 k
 python3-html2text                      noarch          2019.9.26-3.el7                  epel              58 k
 redhat-rpm-config                      noarch          9.1.0-88.el7.centos              base              81 k
 rpm-build                              x86_64          4.11.3-48.el7_9                  updates          150 k
 xz-lzma-compat                         x86_64          5.2.2-1.el7                      base              18 k
Updating for dependencies:
 gettext                                x86_64          0.19.8.1-3.el7                   base             1.0 M
 gettext-libs                           x86_64          0.19.8.1-3.el7                   base             502 k
 glibc                                  x86_64          2.17-325.el7_9                   updates          3.6 M
 glibc-common                           x86_64          2.17-325.el7_9                   updates           12 M
 libgcc                                 x86_64          4.8.5-44.el7                     base             103 k
 libgomp                                x86_64          4.8.5-44.el7                     base             159 k
 rpm                                    x86_64          4.11.3-48.el7_9                  updates          1.2 M
 rpm-build-libs                         x86_64          4.11.3-48.el7_9                  updates          108 k
 rpm-libs                               x86_64          4.11.3-48.el7_9                  updates          279 k
 rpm-python                             x86_64          4.11.3-48.el7_9                  updates           84 k

Transaction Summary
================================================================================================================
Install  1 Package  (+96 Dependent packages)
Upgrade             ( 10 Dependent packages)

Total size: 68 M
Total download size: 38 M
```


## Convert a test package
Download a testing .deb package and try to convert it. 

```shell
wget http://ftp.de.debian.org/debian/pool/main/s/snmptt/snmptt_1.5~beta2-1_all.deb

alien --to-rpm --scripts --keep-version --verbose snmptt_1.5~beta2-1_all.deb
	dpkg-deb --info 'snmptt_1.5~beta2-1_all.deb' control 2>/dev/null
	dpkg-deb --info 'snmptt_1.5~beta2-1_all.deb' control 2>/dev/null
	dpkg-deb --info 'snmptt_1.5~beta2-1_all.deb' conffiles 2>/dev/null
	dpkg-deb --fsys-tarfile 'snmptt_1.5~beta2-1_all.deb' | tar tf -
	dpkg-deb --info 'snmptt_1.5~beta2-1_all.deb' postinst 2>/dev/null
	dpkg-deb --info 'snmptt_1.5~beta2-1_all.deb' postrm 2>/dev/null
	dpkg-deb --info 'snmptt_1.5~beta2-1_all.deb' preinst 2>/dev/null
	dpkg-deb --info 'snmptt_1.5~beta2-1_all.deb' prerm 2>/dev/null
	mkdir snmptt-1.5~beta2
	chmod 755 snmptt-1.5~beta2
	dpkg-deb -x snmptt_1.5~beta2-1_all.deb snmptt-1.5~beta2
	rpm --showrc
	cd snmptt-1.5~beta2; rpmbuild --buildroot='/root/deb2rpm/snmptt-1.5~beta2' -bb --target noarch 'snmptt-1.5~beta2-1.spec' 2>&1
snmptt-1.5~beta2-1.noarch.rpm generated
```


## Test installation
try to install it , sometimes during the installation there can be errors, and a rpm-rebuild needs to be run in order to change the script(s) inside the RPM.

```shell
yum localinstall snmptt-1.5~beta2-1.noarch.rpm 

Loaded plugins: fastestmirror, langpacks
Examining snmptt-1.5~beta2-1.noarch.rpm: snmptt-1.5~beta2-1.noarch
Marking snmptt-1.5~beta2-1.noarch.rpm as an update to snmptt-1.4.2-1.el7.noarch
Resolving Dependencies
--> Running transaction check
---> Package snmptt.noarch 0:1.4.2-1.el7 will be updated
---> Package snmptt.noarch 0:1.5~beta2-1 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

==========================================================================================================================================================================
 Package                           Arch                              Version                                  Repository                                             Size
==========================================================================================================================================================================
Updating:
 snmptt                            noarch                            1.5~beta2-1                              /snmptt-1.5~beta2-1.noarch                            1.0 M

Transaction Summary
==========================================================================================================================================================================
Upgrade  1 Package

Total size: 1.0 M
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test


Transaction check error:
  file / from install of snmptt-1.5~beta2-1.noarch conflicts with file from package filesystem-3.2-25.el7.x86_64
  file /lib from install of snmptt-1.5~beta2-1.noarch conflicts with file from package filesystem-3.2-25.el7.x86_64
  file /usr/lib from install of snmptt-1.5~beta2-1.noarch conflicts with file from package filesystem-3.2-25.el7.x86_64
  file /usr/sbin from install of snmptt-1.5~beta2-1.noarch conflicts with file from package filesystem-3.2-25.el7.x86_64
  file /etc/init.d from install of snmptt-1.5~beta2-1.noarch conflicts with file from package chkconfig-1.7.6-1.el7.x86_64

Error Summary
-------------
```


## Rebuild the package
Because of the transaction check, we need to rebuild the package

```shell
sudo yum install -y rpmrebuild
rpmrebuild -pe snmptt-1.5~beta2-1.noarch.rpm
```

It will open the package settings in text editor. Go to `%files%` section and delete the lines that refer to the conflicting files listed above.

try to just comment the lines instead: 

```shell
(Converted from a deb package by alien version 8.95.)
%files
#%dir %attr(0755, root, root) "/"
#%dir %attr(0755, root, root) "/etc/init.d"
#%dir %attr(0755, root, root) "/lib"
#%dir %attr(0755, root, root) "/usr/lib"
#%dir %attr(0755, root, root) "/usr/lib"
#%dir %attr(0755, root, root) "/usr/sbin"
```

Save and exit the file (exit with `:wq`  (it's vim)). When you exit, you will be asked if you want to continue with the rebuild. Enter `Y` to proceed.

```shell
rpmrebuild -pe snmptt-1.5~beta2-1.noarch.rpm

Do you want to continue ? (y/N) y
result: /root/rpmbuild/RPMS/noarch/snmptt-1.5~beta2-1.noarch.rpm
```

Once the rebuild is complete, you can install it properly

## Test installation (again)

```shell
sudo yum localinstall /root/rpmbuild/RPMS/noarch/snmptt-1.5~beta2-1.noarch.rpm

yum localinstall /root/rpmbuild/RPMS/noarch/snmptt-1.5~beta2-1.noarch.rpm

Loaded plugins: fastestmirror, langpacks
Examining /root/rpmbuild/RPMS/noarch/snmptt-1.5~beta2-1.noarch.rpm: snmptt-1.5~beta2-1.noarch
Marking /root/rpmbuild/RPMS/noarch/snmptt-1.5~beta2-1.noarch.rpm as an update to snmptt-1.4.2-1.el7.noarch
Resolving Dependencies
--> Running transaction check
---> Package snmptt.noarch 0:1.4.2-1.el7 will be updated
---> Package snmptt.noarch 0:1.5~beta2-1 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================
 Package              Arch                 Version                   Repository                                Size
====================================================================================================================
Updating:
 snmptt               noarch               1.5~beta2-1               /snmptt-1.5~beta2-1.noarch               1.0 M

Transaction Summary
====================================================================================================================
Upgrade  1 Package

Total size: 1.0 M
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : snmptt-1.5~beta2-1.noarch                                                                        1/2 
warning: /etc/snmp/snmptt.ini saved as /etc/snmp/snmptt.ini.rpmsave
  Cleanup    : snmptt-1.4.2-1.el7.noarch                                                                        2/2 
  Verifying  : snmptt-1.5~beta2-1.noarch                                                                        1/2 
  Verifying  : snmptt-1.4.2-1.el7.noarch                                                                        2/2 

Updated:
  snmptt.noarch 0:1.5~beta2-1                                                                                       

Complete!
```

Do observe though that there might still be some permission issues depending on whether the package was installed previously because of the edits in the RPM scripts.

## Additional changes
As an example, this package had some more requirements that were missing after installation, and it was sorted like this:

```shell
yum install -y perl-Sys-Syslog
yum install -y perl-Net-IP.noarch
```

get some permissions sorted
```shell 
cd /var/spool/
chown -R snmptt:snmptt snmptt/
systemctl restart snmptt
```

and the pids had to be sorted as well
```shell
chown snmptt:snmptt /var/run/snmptt.pid 
chown snmptt:snmptt /run/snmptt.pid 
```

After that , all that's left was to go through the configuration file for the package and change accordingly in order to get everything up and running again. The previous configuration file was overwritten, so remember to always have a backup. 


## EOF 


