1 - Install Nagios 4.2.0 Core

```yum install httpd php glibc glibc-common gd gd-devel make net-snmp```

   - Tạo user nagios và group \
```useradd nagios```\
```passwd nagios```\
```groupadd nagcmd```\
```usermod -a -G nagcmd nagios```\
```usermod -a -G nagcmd apache```

    - Biên dịch và cài đặt Nagios Core 4\
```tar zxfv nagios-4.0.5.tar.gz```\
```cd nagios-4.0.5```
```./configure --with-command-group=nagcmd```\
```make all```\
```make install```\
```make install-init```\
```make install-commandmode```\
```make install-config```\
```make install-webconf```\
```make install-exfoliation```\
```cp -R contrib/eventhandlers/ /usr/local/nagios/libexec```\
```chown -R nagios:nagios /usr/local/nagios/libexec/eventhandlers```\
    - Kiem tra cau hinh \
```/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg```\
    - Tao user/pass login\
```htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin```\
```chkconfig --add nagios && chkconfig --level 35 nagios on && chkconfig --level 35 httpd on```\
```service nagios restart && service httpd restart```\
    - http://ip-server/nagios/

2 - Nagios Plugins \

```yum install bind-utils net-snmp-devel net-snmp-utils net-snmp-perl```\
```yum install mysql-client mysql-devel samba-client samba-common postgresql-devel openldap-devel```\
```wget https://nagios-plugins.org/download/nagios-plugins-2.0.1.tar.gz```\
```cd nagios-plugins-2.0.1```\
```./configure -with-nagios-user=nagios -with-nagios-group=nagios```\
```make```\
```make install```\

3 - PNP4Nagios\

```yum install rrdtool perl-Time-HiRes rrdtool-perl php-gd```\
```wget https://sourceforge.net/projects/pnp4nagios/files/PNP-0.6/pnp4nagios-0.6.16.tar.gz```\
```tar zxfv pnp4nagios-0.6.16.tar.gz```\
```cd pnp4nagios-0.6.16```\
```./configure```\
```make all```\
```make fullinstall```\
```HTML URL: http://localhost/pnp4nagios```\
```- Apache Config File: /etc/httpd/conf.d/pnp4nagios.conf```\
```chkconfig --add npcd && chkconfig --level 35 npcd on```\
```service httpd reload```\
```mv /usr/local/pnp4nagios/share/install.php /usr/local/pnp4nagios/share/install.php.bk```\

    *- Neu xuat hien loi nay la lam tiep thep buoc duoi 

Please check the documentation for information about the following error.perfdata directory “/usr/local/pnp4nagios/var/perfdata/” is empty. Please check your Nagios config. Read FAQ online"\

```copy /usr/local/pnp4nagios/etc/nagios.cfg-sample /usr/local/nagios/etc/nagios.cfg or them doan nay duoi cung nagios.cfg```\

```process_performance_data=1
service_perfdata_file=/usr/local/pnp4nagios/var/service-perfdata
service_perfdata_file_template=DATATYPE::SERVICEPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tSERVICEDESC::$SERVICEDESC$\tSERVICEPERFDATA::$SERVICEPERFDATA$\tSERVICECHECKCOMMAND::$SERVICECHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$\tSERVICESTATE::$SERVICESTATE$\tSERVICESTATETYPE::$SERVICESTATETYPE$
service_perfdata_file_mode=a
service_perfdata_file_processing_interval=15
service_perfdata_file_processing_command=process-service-perfdata-file
host_perfdata_file=/usr/local/pnp4nagios/var/host-perfdata
host_perfdata_file_template=DATATYPE::HOSTPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tHOSTPERFDATA::$HOSTPERFDATA$\tHOSTCHECKCOMMAND::$HOSTCHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$
host_perfdata_file_mode=a
host_perfdata_file_processing_interval=15
host_perfdata_file_processing_command=process-host-perfdata-file
# Load Livestatus Module
broker_module=/usr/lib/check_mk/livestatus.o /usr/local/nagios/var/rw/live
event_broker_options=-1
# added by setup.sh of check_mk
cfg_dir=/usr/local/nagios/etc/check_mk.d

#Define command.cfg#

*#* copy cp /usr/local/pnp4nagios/etc/misccommands.cfg-sample /usr/local/nagios/etc/objects/commands.c

define command{
        command_name    process-host-perfdata
        command_line    /usr/bin/printf "%b" "$LASTHOSTCHECK$\t$HOSTNAME$\t$HOSTSTATE$\t$HOSTATTEMPT$\t$HOSTSTATETYPE$\t$HOSTEXECUTIONTIME$\t$HOSTOUTPUT$\t$HOSTPERFDATA$\n" >> /usr/local/nagios/var/host-perfdata.out
        }

define command{
        command_name    process-service-perfdata
        command_line    /usr/bin/printf "%b" "$LASTSERVICECHECK$\t$HOSTNAME$\t$SERVICEDESC$\t$SERVICESTATE$\t$SERVICEATTEMPT$\t$SERVICESTATETYPE$\t$SERVICEEXECUTIONTIME$\t$SERVICELATENCY$\t$SERVICEOUTPUT$\t$SERVICEPERFDATA$\n" >> /usr/local/nagios/var/service-perfdata.out
        }

define command{
        command_name   process-service-perfdata-file
        command_line   /usr/local/pnp4nagios/libexec/process_perfdata.pl --bulk=/usr/local/pnp4nagios/var/service-perfdata
}
define command{
        command_name   process-host-perfdata-file
        command_line   /usr/local/pnp4nagios/libexec/process_perfdata.pl --bulk=/usr/local/pnp4nagios/var/host-perfdata
}

service npcd restart && service nagios restart

- Cau hinh link graph cho nagios    
- chinh sua /usr/local/nagios/etc/objects/templates.cfg\

define host {
        name host-pnp
        action_url /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=_HOST_
        register 0
}

define service {
        name srv-pnp
        action_url /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=$SERVICEDESC$
        register 0
}

cp contrib/ssi/status-header.ssi /usr/local/nagios/share/ssi/

service npcd restart && service nagios restart

  4 - Install Check_mk

  #Centos 6 mod_python
  # Install EPEL Repositories (if needed it)
  rpm --import https://fedoraproject.org/static/0608B895.txt
  wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
  yum localinstall epel-release-6-8.noarch.rpm
  # Install python module for apache
  yum install mod_python

  #Centos 7 mod_python
  wget http://repo.iotti.biz/CentOS/7/x86_64/mod_python-3.5.0-16.el7.lux.1.x86_64.rpm
  rpm -ivh mod_python-3.5.0-16.el7.lux.1.x86_64.rpm```
  
  wget http://archive.mathias-kettner.de/check_mk/check_mk-1.2.5i2p1.tar.gz
  tar zxfv check_mk-1.2.5i2p1.tar.gz
  cd check_mk-1.2.5i2p1```
  ./setup
  service httpd restart $$ service npcd restart && service nagios restart
  Chinh sua file nagios.cfg
# Load Livestatus Module`
  broker_module=/usr/lib/check_mk/livestatus.o /usr/local/nagios/var/rw/live
  event_broker_options=-1
  # added by setup.sh of check_mk
  cfg_dir=/usr/local/nagios/etc/check_mk.d
  #note 
  neu xuat hien loi 
  MK Livestatus Cannot connect to 'unix:/usr/local/nagios/var/rw/live 
  vim /usr/share/check_mk/modules/defaults (bo 3 dong dau + livestatus_unix_socket      = '/usr/local/nagios/var/rw/live')     vim /usr/share/check_mk/web/htdocs/defaults.py (bo 3 dong dau + livestatus_unix_socket      =       '/usr/local/nagios/var/rw/live')

  chown nagios.nagcmd /usr/local/nagios/var/rw 
  chmod g+rwx /usr/local/nagios/var/rw 
  chmod g+s /usr/local/nagios/var/rw

  URI http://ip-server/check_mk.
  tail /usr/local/nagios/var/nagios.log
