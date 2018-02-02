Title: Howto create a dev server
Date: 2009-05-19 23:26
Author: Surya
Category: Rant
Slug: howto-create-a-dev-server
Status: published

From childhood i have always been fascinated by computers and my first
box was a 386 machine with a monochrome monitor. Whereas now i have a
powerful Quad core machine ^\_^ I have a home network which connects
couple of wired desktops, XBox (which is now a media center with XBMC)
and also wireless iPhone and Macbook pro. I have dedicated a box which
is not publicly accessible to be a Dev server, and i recommend you doing
the same. In this box i have Windows XP and the first thing i installed
were Avira AntiVir ([best free antivirus](http://av-comparatives.org/)),
SuperAntiSpyware and Microsoft BitDefender.

## Installing XAMPP as WAMP Server

Moving on, i installed
[XAMPP](http://www.apachefriends.org/en/xampp-windows.html) in c:\\xampp
because it makes your life easier to install Apache Web Server 2.x with
Openssl, PHP 5.x, MySQL 5.x, phpmyadmin, FTP server and a Mail server in
one go. I installed all except FTP and mail server as a service. And if
you want you can configure each piece separately by changing their ini
files once installed. So this takes care of Webserver with php
Scripting, Database with Web based GUI (BTW phpmyadmin is the web based
GUI if you haven't already figured it out ^\_~ ). After installing go to
[http://localhost](http://localhost/ "http://localhost") and by default
you will see xampp's page its due to index.php which redirects the
browser to xampp's folder. its recommended to change all of the default
settings i.e change password because sometimes people forget to change
it and a hackers first attempt is to try the default password. its
pretty simple, click security and follow instructions. one more
advantage of using xampp is that you can also plugin other components
perl, python if you want.Also it comes with a cool control panel from
which you can control turning on and off the server.i will show
apache+php configuration when i start a project

## Installing Tomcat as app server

Now lets install [tomcat](http://tomcat.apache.org/download-60.cgi), a
servlet container which will act as app server for now. but first you
have to install the latest stable
[JDK](http://java.sun.com/javase/downloads/index.jsp). My preference is
to create a folder called c:\\java and keep all my java related software
and library there so its easy to organize and find java related stuff,
plus i don't like spaces in my folder names i.e. C:\\Program Files ,
since there are few programmes out there which fails due to this reason
alone O\_O. make sure to change the path in the wizard for both jdk and
jre its installed as C:\\java\\jdk1.6.0\_13 and C:\\java\\jre6 also in
your environment variable set JAVA\_HOME value to
C:\\java\\jdk1.6.0\_13. Once that's done get the latest tomcat's windows
service installer and install tomcat as a service. so i installed it as
C:\\java\\apache-tomcat-6.0.18. During install it will confirm for new
port (8080 by default) and admin password. I will show tomcat
configuration when i start a project

## Installing Subversion as server for revision control system

Next step is to install revision control system, I choose
[subversion](http://subversion.tigris.org/) cause i feel this is the
next generation of cvs, and they have a good number of quality and easy
to use clients like [Tortoisesvn](http://tortoisesvn.net/downloads)
(windows GUI client), [Slik
subersion](http://www.sliksvn.com/en/download) (command line client),
[Subclipse](http://subclipse.tigris.org/install.html) (Eclipse),
[Anksvn](http://ankhsvn.open.collab.net/) (visual studio) i have
installed all four. On my server i installed [CollabNet
Subversion](http://www.open.collab.net/downloads/subversion/) (Certified
binary for subversion) because it was easy and i just followed its
readme for the setup. Couple of things to note, you have register for
free at collabnet before downloading and there are multiple ways of
installing and since i am using it for internal dev only i am installing
the svnserve (svn:// protocol) instead of apache as svn server(http://
protocol)

``` 




svnserve's readme





svnserve's readme


A. Using svnserve
      ==============

   To use svnserve as your server, follow these steps:

     1. Open a new terminal (command prompt).

        NOTE: If you have an old command prompt open (prior to the Subversion
              installation), remember to open a brand new command prompt.

     2. Create a subversion repository.

        cd
        svnadmin create 

        For example:
        cd \repositories
        svnadmin create my-first-repos

     3. Setup a password database.

        Using notepad, edit the svnserve.conf file inside the conf directory of your
        repository.

        For example:
        If your repository is C:\repositories\my-first-repos
        svnserve.conf is:
        C:\repositories\my-first-repos\conf\svnserve.conf

        Inside svnserve.conf, you see the following information:

         ### The password-db option controls the location of the password
         ### database file.  Unless you specify a path starting with a /,
         ### the file's location is relative to the conf directory.
         ### Uncomment the line below to use the default password file.
         #password-db = passwd

       Follow the above instructions, and uncomment the "password-db=passwd"
       line, so that it simply says:

          password-db = passwd

     4. Setup usernames and passwords.

       Next, edit the passwd file. This passwd file is located in the
       same directory as svnserve.conf.

       Inside the passwd file, you see the following information:

       ### This file is an example password file for svnserve.
       ### Its format is similar to that of svnserve.conf. As shown in the
       ### example below it contains one section labelled [users].
       ### The name and password for each user follow, one account per line.

       [users]
       #harry = harryssecret
       #sally = sallyssecret

       To add a new user account, add your own username and password
       inside the [users] section. For example, if your name is "joe",
       and you want to set your password to "super-secret", add a
       new line like this:

         joe = super-secret

       Add as many users as you like.

     5. Open Port on Windows firewall.

        Before starting the server, the firewall must be notified that
        this particular port is going to be used. To enable this port in the
        Windows firewall, follow the instructions found here:

    http://www.microsoft.com/windowsxp/using/security/internet/sp2_wfexcepti...

        Note: svnserve.exe is the program name which needs to be added to the
        exceptions list. Alternatively, you can also use the port where
        you decide to run the server. By default, svnserve runs on 3690.

     6. Start svnserve.

        If you elected to have the installer setup svnserve as a service, then open
        the Services application, find the entry for the Subversion server, and take
        the Start option. The service has been configured to start automatically
        on reboot. You can also run this command from the command line:

        net start svnserve

        If you did not install svnserve as a service and want to start the server
        manually, run this command:

        svnserve -d -r 

        For example: svnserve -d -r C:\repositories

     7. To provide read and write access to anonymous users, modify the
        conf/svnserve.conf file inside the repository.

        anon-access = write

        To restrict an anonymous user from the repository:

        anon-access = none

     8. Check out the repository.

        svn co svn://localhost/

        For example: svn co svn://localhost/my-first-repos

     Tip: If you check out your Subversion repository from a different computer,
     replace 'localhost' with the IP address or hostname of the machine which
     hosts the Subversion repository.



 


 
```

## Installing Hudson, A Continuous Integration Server

Download [Hudson](http://hudson.gotdns.com/latest/hudson.war), if it
downloads as a zip rename the extension as war(basically jar, war and
ear are just zip files) and place it under tomcats webapps folder i.e.
C:\\java\\apache-tomcat-6.0.18\\webapps. Start tomcat and thats it you
have installed Hudson.

## Installing Maven

For Java using Ant is old school. You should use maven instead. Why?
because its so much better for project/dependency management and its
plugin system adds so much more functionality than ant, for example take
a look at [apache's plugins](http://maven.apache.org/plugins/index.html)
and [codehaus plugins](http://mojo.codehaus.org/plugins.html). So we are
going to create a Maven repository. First Download maven2 extract the
zip to say c:\\java\\apache-maven-2.1.0 then set the environment
variable M2\_HOME to c:\\java\\apache-maven-2.1.0 and Path as
%Path%;c:\\java\\apache-maven-2.1.0\\bin open
c:\\java\\apache-maven-2.1.0\\conf\\settings.xml uncomment
localrepository and change the path to C:\\java\\maven-repo create a
folder .m2 in your home folder which would be C:\\Documents and
Settings\\\\.m2 you can do this by command prompt C:\\Documents and
Settings\\surya\>mkdir .m2 then create settings.xml and copy paste
[Example 3.1](http://www.sonatype.com/books/nexus-book/reference/maven-sect-single-group.html)
(make sure to change the port) I also installed Nexus a Maven Repository
Manager (due to m2eclipse supports nexus) Downloaded the open source war
renamed to nexus.war and placed it inside tomcats webapp folder. logged
in as administrator with username/password as admin/admin123 If you can
see the repo that means its successfully installed.

## Installing Trac

Under Construction...

## In Conclusion

Right now in this article i am focused on installing the dev system.
Whenever i start working on my project, i will configure each system
individually and will write about it. You might have noticed, i am big
fan of prepackaged binaries or wizard based software since they save
time and do most of the work for you. Power user prefer to compile the
source or extract from zip and manually configure it, the pros is, it
increases your understanding and knowledge of the product , the cons
would be it takes time, learning and going through documentation (which
is not necessarily a bad thing ^\_^). So personally (since i am
developer and not administrator) if i can find a good wizard based
software i will use that, otherwise i will use the zip version. (Under
Construction...,Jira)
