The cloud instances we will use for LSCS/LSDS are likely to be different than the internal servers.   Most importantly, the server will likely not be able to use the GUI installer (which is an X-Windows based installer).

As an example I built an EC-2 instance and went through the install process. This confluence page is a complete and repeatable set of steps to build another LSCS/LSDS server.

Step-by-step guideFirst we need to assemble the prerequisites:

Download from OT (assuming version 8.1 - copy all to /opt/vgi/Interwoven/installs):

iwovlivesite-linux-8.1.0.0.iwpkg
iwovlivesite-linux-8.1.0.0.iwpkg.md5sum
iwovinstaller-silent_install-8.1.0.0.0.zip
iwovinstaller-linux-8.1.0.0.0.bin
iwovopendeployrcvr.linux.8.1.0.tar.gz
Appropriate a DB instance for LSDS as well as LSCS

Example used a local copy of mysql (items listed are the example)
Get DB Hostname (localhost)
Get DB port (3306)
Get username ( iwdbuser)
Get password (encoded: VCjd5CuAnynhlEQzTz/WS4XO2SeAECux)
Get Appropriate JDBC interface (/opt/vgi/Interwoven/mysql-connector-java-5.1.30/mysql-connector-java-5.1.30-bin.jar)
Get instance names (LSCS LSDS)
Install JDK ( /opt/vgi/Interwoven/java/jdk1.7.0_80)
Install Tomcat (/opt/vgi/Interwoven/ls-tomcat/)
Install OD prereqs (OpenSSL was the only thing missing - but the OD install manual is a good reference) 
Make the LSCS Store directory (mkdir /opt/vgi/Interwoven/LSCSRT-Store) 
 

Database Setup: Oracle

Open a ticket with the BAM team to setup a hybrid id for database access http://appvgi38.vanguard.com/corp/BAMForms.nsf/xpBAMITSupport.xsp (for e.g. awcs301d)
Have the BAM team create a tablespace  (for e.g. awcs301d_data)
Have the BAM team provide complete access to that tablespace.
Access the database schema using the following informationid : awcs301d
pwd : <pwd_for_id>
connection string : jdbc:oracle:thin:@ldap://oidprd40:3060/wcsdev00_all,cn=OracleContext,dc=vanguard,dc=comldap://oidprd50:3060/wcsdev00_all,cn=OracleContext,dc=vanguard,dc=comldap://oidprd60:3060/wcsdev00_all,cn=OracleContext,dc=vanguard,dc=com
encrypt the password for the id using java -jar iwov-install-enc.jar <password>
Run the following sql scripts to validate database setuplscs_ddl_oracle.sql
If the results of step 6 is successful , the  database creation and setup is successful.
 

Start the install:

Extract/Install OpenDeploy (All as root)

mkdir /opt/vgi/Interwoven/installs/tmp
cd   /opt/vgi/Interwoven/installs/tmp
tar xvf  /opt/vgi/Interwoven/installs/iwovopendeployrcvr.linux.8.1.0.tar.gz
./startdeploy_odAccept all default values.
When completed, execute  service iwodserver start
Execute /opt/vgi/Interwoven/OpenDeployNG/bin/iwodserverstatusThis should return with 0 active deployments.
Encode DB password
 Extract silent installermkdir /opt/vgi/Interwoven/installs/silent
cd /opt/vgi/Interwoven/installs/silent
unzip /opt/vgi/Interwoven/installs/iwovinstaller-silent_install-8.1.0.0.0.zip
Run
java -jar  /opt/vgi/Interwoven/installs/silent/utility/JarEcode.jar
Type the password in,
Save the output
Update following lines: in installer.properties:
lscsRuntimeDbPwd=VCjd5CuAnynhlEQzTz/WS4XO2SeAECux
lsds_dbPwd=VCjd5CuAnynhlEQzTz/WS4XO2SeAECux
Download the attached file installer.properties as the starting point. 
Update other entries in installer.properties including:
lscstomcathome=/opt/vgi/Interwoven/ls-tomcat/
lscsruntimejavahome=/opt/vgi/Interwoven/java/jdk1.7.0_80/
lscsRuntimeDBType=mysql 
lscsRuntimeDBJarList=/opt/vgi/Interwoven/mysql-connector-java-5.1.30/mysql-connector-java-5.1.30-bin.jar 
lscsRuntimeDbServerName=localhost 
lscsRuntimeDbPort=3306 
lscsRuntimeDbUser=iwdbuser 
lsDBType=mysql 
lsds_dbJarList=/opt/vgi/Interwoven/mysql-connector-java-5.1.30/mysql-connector-java-5.1.30-bin.jar 
lsds_dbServerName=localhost 
lsds_dbPort=3306 
lsds_dbUser=iwdbuser 
lsds_jdkHome=/opt/vgi/Interwoven/java/jdk1.7.0_80/ 
lsds_tomcatInstallDir=/opt/vgi/Interwoven/ls-tomcat/ 
Execute installer:
Make certain you are root and not using sudo
cd  /opt/vgi/Interwoven/installs
unset DISPLAY (turns off GUI installer)
chmod +x iwovinstaller-linux-8.1.0.0.0.bin
./iwovinstaller-linux-8.1.0.0.0.bin -f ./installer.properties
In another window: tail -f /tmp/stderr /tmp/installer.log
When the command above stops updating runtail -f /opt/vgi/Interwovne/iwinstall/log/installer.log
  If you see no errors, run:grep COMPLETED /opt/vgi/Interwoven/iwinstall/private/config/inventory.xml
Expected results:
    <Component status="COMPLETED" id="LiveSiteCSRT8.1.0.0.0">
    <Component status="COMPLETED" id="LiveSiteDisplayServices8.1.0.0.0">
LSCS and LSDS should be installed and ready for configurations. 
Post installation patch update

For LiveSite 8.1Please download the patch from subversion http://prdsvnrepo:8080/svn/tip/ebs/wcs/config/trunk/Maintenance/LiveSite_pvlva0dg/opt/vgi/Interwoven/LiveSiteCSRT/runtime/webapps/lscs/WEB-INF/lib/rtserver.jar and install on the following path
/opt/vgi/Interwoven/LiveSiteCSRT/runtime/webapps/lscs/WEB-INF/lib/
Please download the xml configuration for https content routing from http://prdsvnrepo:8080/svn/tip/ebs/wcs/config/trunk/Maintenance/LiveSite_pvlva0dg/opt/vgi/Interwoven/LiveSiteCSRT/runtime/webapps/lscs/WEB-INF/context and install in the following path
 /opt/vgi/Interwoven/LiveSiteCSRT/runtime/webapps/lscs/WEB-INF/context
Update the lscs-static.xml file under /opt/vgi/Interwoven/ls-tomcat/conf/Catalina/localhost to have the right LSCSRT-Store path
Please follow the steps in LiveSite Log Rolling
For LiveSite 16.x Please download the xml configuration for https content routing from http://prdsvnrepo:8080/svn/tip/ebs/wcs/config/trunk/Maintenance/LiveSite_pvlva0dg/opt/vgi/Interwoven/LiveSiteCSRT/runtime/webapps/lscs/WEB-INF/context and install in the following path
 /opt/vgi/Interwoven/LiveSiteCSRT/runtime/webapps/lscs/WEB-INF/context
Please follow the steps in LiveSite Log Rolling
Update the lscs-static.xml file under /opt/vgi/Interwoven/ls-tomcat/conf/Catalina/localhost to have the right LSCSRT-Store path
