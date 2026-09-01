# Практика c SELinux

1. Запустить Nginx на нестандартном порту 3-мя разными способами:
переключатели setsebool;
добавление нестандартного порта в имеющийся тип;
формирование и установка модуля SELinux.
К сдаче:
README с описанием каждого решения (скриншоты и демонстрация приветствуются). 

2. Обеспечить работоспособность приложения при включенном selinux.
развернуть приложенный стенд https://github.com/Nickmob/vagrant_selinux_dns_problems; 
выяснить причину неработоспособности механизма обновления зоны (см. README);
предложить решение (или решения) для данной проблемы;
выбрать одно из решений для реализации, предварительно обосновав выбор;
реализовать выбранное решение и продемонстрировать его работоспособность.

## 1.Подготовка 
```console
PS E:\ucheba> vagrant up
Bringing machine 'selinux' up with 'virtualbox' provider...
==> selinux: Importing base box 'a3b277da-9a39-11f1-afa6-1e3eae98cf98'...
==> selinux: Matching MAC address for NAT networking...
==> selinux: Setting the name of the VM: ucheba_selinux_1787834920846_56761
==> selinux: Clearing any previously set network interfaces...
==> selinux: Preparing network interfaces based on configuration...
    selinux: Adapter 1: nat
==> selinux: Forwarding ports...
    selinux: 4881 (guest) => 4881 (host) (adapter 1)
    selinux: 22 (guest) => 2222 (host) (adapter 1)
==> selinux: Running 'pre-boot' VM customizations...
==> selinux: Booting VM...
==> selinux: Waiting for machine to boot. This may take a few minutes...
    selinux: SSH address: 127.0.0.1:2222
    selinux: SSH username: vagrant
    selinux: SSH auth method: private key
    selinux:
    selinux: Vagrant insecure key detected. Vagrant will automatically replace
    selinux: this with a newly generated keypair for better security.
    selinux:
    selinux: Inserting generated public key within guest...
    selinux: Removing insecure key from the guest if it's present...
    selinux: Key inserted! Disconnecting and reconnecting using new SSH key...
==> selinux: Machine booted and ready!
==> selinux: Checking for guest additions in VM...
    selinux: The guest additions on this VM do not match the installed version of
    selinux: VirtualBox! In most cases this is fine, but in rare cases it can
    selinux: prevent things such as shared folders from working properly. If you see
    selinux: shared folder errors, please make sure the guest additions within the
    selinux: virtual machine match the version of VirtualBox you have installed on
    selinux: your host and reload your VM.
    selinux:
    selinux: Guest Additions Version: 7.1.18
    selinux: VirtualBox Version: 7.2
==> selinux: Setting hostname...
==> selinux: Mounting shared folders...
    selinux: E:/ucheba => /vagrant
==> selinux: Running provisioner: shell...
    selinux: Running: inline script
    selinux: AlmaLinux 10 - AppStream                        1.7 MB/s | 2.5 MB     00:01
    selinux: AlmaLinux 10 - BaseOS                           7.6 MB/s |  30 MB     00:03
    selinux: AlmaLinux 10 - CRB                              605 kB/s | 587 kB     00:00
    selinux: AlmaLinux 10 - Extras                           8.9 kB/s | 7.3 kB     00:00
    selinux: Dependencies resolved.
    selinux: ================================================================================
    selinux:  Package                         Arch     Version                Repo      Size
    selinux: ================================================================================
    selinux: Installing:
    selinux:  epel-release                    noarch   10-6.el10              extras    18 k
    selinux: Installing dependencies:
    selinux:  selinux-policy-targeted-extra   noarch   42.1.18-4.el10_2.3     crb      720 k
    selinux: Installing weak dependencies:
    selinux:  selinux-policy-extra            noarch   42.1.18-4.el10_2.3     crb       36 k
    selinux:
    selinux: Transaction Summary
    selinux: ================================================================================
    selinux: Install  3 Packages
    selinux:
    selinux: Total download size: 775 k
    selinux: Installed size: 822 k
    selinux: Downloading Packages:
    selinux: (1/3): selinux-policy-extra-42.1.18-4.el10_2.3. 363 kB/s |  36 kB     00:00
    selinux: (2/3): epel-release-10-6.el10.noarch.rpm        174 kB/s |  18 kB     00:00
    selinux: (3/3): selinux-policy-targeted-extra-42.1.18-4. 3.8 MB/s | 720 kB     00:00
    selinux: --------------------------------------------------------------------------------
    selinux: Total                                           521 kB/s | 775 kB     00:01
    selinux: Running transaction check
    selinux: Transaction check succeeded.
    selinux: Running transaction test
    selinux: Transaction test succeeded.
    selinux: Running transaction
    selinux:   Preparing        :                                                        1/1
    selinux:   Running scriptlet: selinux-policy-targeted-extra-42.1.18-4.el10_2.3.noa   1/3
    selinux:   Installing       : selinux-policy-targeted-extra-42.1.18-4.el10_2.3.noa   1/3
    selinux:   Installing       : selinux-policy-extra-42.1.18-4.el10_2.3.noarch         2/3
    selinux:   Installing       : epel-release-10-6.el10.noarch                          3/3
    selinux:   Running scriptlet: epel-release-10-6.el10.noarch                          3/3
    selinux: Many EPEL packages require the CodeReady Builder (CRB) repository.
    selinux: It is recommended that you run /usr/bin/crb enable to enable the CRB repository.
    selinux:
    selinux:   Running scriptlet: selinux-policy-targeted-extra-42.1.18-4.el10_2.3.noa   3/3
    selinux:   Running scriptlet: epel-release-10-6.el10.noarch                          3/3
    selinux:
    selinux: Installed:
    selinux:   epel-release-10-6.el10.noarch
    selinux:   selinux-policy-extra-42.1.18-4.el10_2.3.noarch
    selinux:   selinux-policy-targeted-extra-42.1.18-4.el10_2.3.noarch
    selinux:
    selinux: Complete!
    selinux: Extra Packages for Enterprise Linux 10 - x86_64 1.6 MB/s | 6.6 MB     00:04
    selinux: Dependencies resolved.
    selinux: ================================================================================
    selinux:  Package                  Arch      Version                  Repository    Size
    selinux: ================================================================================
    selinux: Installing:
    selinux:  nginx                    x86_64    2:1.26.3-6.el10_2.6      appstream     32 k
    selinux: Installing dependencies:
    selinux:  almalinux-logos-httpd    noarch    100.4-1.el10_2           appstream     18 k
    selinux:  nginx-core               x86_64    2:1.26.3-6.el10_2.6      appstream    660 k
    selinux:  nginx-filesystem         noarch    2:1.26.3-6.el10_2.6      appstream     11 k
    selinux:
    selinux: Transaction Summary
    selinux: ================================================================================
    selinux: Install  4 Packages
    selinux:
    selinux: Total download size: 722 k
    selinux: Installed size: 1.9 M
    selinux: Downloading Packages:
    selinux: (1/4): almalinux-logos-httpd-100.4-1.el10_2.noa 220 kB/s |  18 kB     00:00
    selinux: (2/4): nginx-1.26.3-6.el10_2.6.x86_64.rpm       306 kB/s |  32 kB     00:00
    selinux: (3/4): nginx-filesystem-1.26.3-6.el10_2.6.noarc 216 kB/s |  11 kB     00:00
    selinux: (4/4): nginx-core-1.26.3-6.el10_2.6.x86_64.rpm  3.2 MB/s | 660 kB     00:00
    selinux: --------------------------------------------------------------------------------
    selinux: Total                                           810 kB/s | 722 kB     00:00
    selinux: Running transaction check
    selinux: Transaction check succeeded.
    selinux: Running transaction test
    selinux: Transaction test succeeded.
    selinux: Running transaction
    selinux:   Preparing        :                                                        1/1
    selinux:   Running scriptlet: nginx-filesystem-2:1.26.3-6.el10_2.6.noarch            1/4
    selinux:   Installing       : nginx-filesystem-2:1.26.3-6.el10_2.6.noarch            1/4
    selinux:   Installing       : nginx-core-2:1.26.3-6.el10_2.6.x86_64                  2/4
    selinux:   Installing       : almalinux-logos-httpd-100.4-1.el10_2.noarch            3/4
    selinux:   Installing       : nginx-2:1.26.3-6.el10_2.6.x86_64                       4/4
    selinux:   Running scriptlet: nginx-2:1.26.3-6.el10_2.6.x86_64                       4/4
    selinux:
    selinux: Installed:
    selinux:   almalinux-logos-httpd-100.4-1.el10_2.noarch
    selinux:   nginx-2:1.26.3-6.el10_2.6.x86_64
    selinux:   nginx-core-2:1.26.3-6.el10_2.6.x86_64
    selinux:   nginx-filesystem-2:1.26.3-6.el10_2.6.noarch
    selinux:
    selinux: Complete!
    selinux: Last metadata expiration check: 0:00:10 ago on Thu 27 Aug 2026 12:50:20 PM UTC.
    selinux: Dependencies resolved.
    selinux: ================================================================================
    selinux:  Package                        Arch     Version              Repository   Size
    selinux: ================================================================================
    selinux: Installing:
    selinux:  policycoreutils-newrole        x86_64   3.10-2.el10_2        baseos       22 k
    selinux:  policycoreutils-python-utils   noarch   3.10-2.el10_2        appstream    45 k
    selinux:  selinux-policy-mls             noarch   42.1.18-4.el10_2.3   baseos      3.9 M
    selinux:  setools-console                x86_64   4.6.0-3.el10         baseos       47 k
    selinux:  setroubleshoot-server          x86_64   3.3.35-4.el10        appstream   377 k
    selinux: Installing dependencies:
    selinux:  mcstrans                       x86_64   3.10-1.el10          baseos      144 k
    selinux:  python3-dasbus                 noarch   1.7-8.el10           appstream   126 k
    selinux:  python3-libxml2                x86_64   2.12.5-10.el10_2.2   baseos      222 k
    selinux:  selinux-policy-mls-extra       noarch   42.1.18-4.el10_2.3   crb         447 k
    selinux:  setroubleshoot-plugins         noarch   3.3.14-11.el10       appstream   375 k
    selinux:
    selinux: Transaction Summary
    selinux: ================================================================================
    selinux: Install  10 Packages
    selinux:
    selinux: Total download size: 5.6 M
    selinux: Installed size: 16 M
    selinux: Downloading Packages:
    selinux: (1/10): policycoreutils-python-utils-3.10-2.el1 372 kB/s |  45 kB     00:00
    selinux: (2/10): python3-dasbus-1.7-8.el10.noarch.rpm    971 kB/s | 126 kB     00:00
    selinux: (3/10): setroubleshoot-plugins-3.3.14-11.el10.n 1.6 MB/s | 375 kB     00:00
    selinux: (4/10): setroubleshoot-server-3.3.35-4.el10.x86 2.8 MB/s | 377 kB     00:00
    selinux: (5/10): mcstrans-3.10-1.el10.x86_64.rpm         869 kB/s | 144 kB     00:00
    selinux: (6/10): policycoreutils-newrole-3.10-2.el10_2.x 294 kB/s |  22 kB     00:00
    selinux: (7/10): python3-libxml2-2.12.5-10.el10_2.2.x86_ 2.5 MB/s | 222 kB     00:00
    selinux: (8/10): setools-console-4.6.0-3.el10.x86_64.rpm 430 kB/s |  47 kB     00:00
    selinux: (9/10): selinux-policy-mls-extra-42.1.18-4.el10 2.7 MB/s | 447 kB     00:00
    selinux: (10/10): selinux-policy-mls-42.1.18-4.el10_2.3.  11 MB/s | 3.9 MB     00:00
    selinux: --------------------------------------------------------------------------------
    selinux: Total                                           1.5 MB/s | 5.6 MB     00:03
    selinux: Running transaction check
    selinux: Transaction check succeeded.
    selinux: Running transaction test
    selinux: Transaction test succeeded.
    selinux: Running transaction
    selinux:   Running scriptlet: selinux-policy-mls-42.1.18-4.el10_2.3.noarch           1/1
    selinux:   Preparing        :                                                        1/1
    selinux:   Installing       : python3-libxml2-2.12.5-10.el10_2.2.x86_64             1/10
    selinux:   Installing       : policycoreutils-newrole-3.10-2.el10_2.x86_64          2/10
    selinux:   Installing       : mcstrans-3.10-1.el10.x86_64                           3/10
    selinux:   Running scriptlet: mcstrans-3.10-1.el10.x86_64                           3/10
    selinux:   Running scriptlet: selinux-policy-mls-42.1.18-4.el10_2.3.noarch          4/10
    selinux:   Installing       : selinux-policy-mls-42.1.18-4.el10_2.3.noarch          4/10
    selinux:   Installing       : python3-dasbus-1.7-8.el10.noarch                      5/10
    selinux:   Installing       : setroubleshoot-plugins-3.3.14-11.el10.noarch          7/10
    selinux:   Running scriptlet: setroubleshoot-server-3.3.35-4.el10.x86_64            8/10
    selinux:   Installing       : setroubleshoot-server-3.3.35-4.el10.x86_64            8/10
    selinux:   Running scriptlet: setroubleshoot-server-3.3.35-4.el10.x86_64            8/10
    selinux:   Running scriptlet: selinux-policy-mls-extra-42.1.18-4.el10_2.3.noarch    9/10
    selinux:   Installing       : selinux-policy-mls-extra-42.1.18-4.el10_2.3.noarch    9/10
    selinux:   Installing       : setools-console-4.6.0-3.el10.x86_64                  10/10
    selinux:   Running scriptlet: selinux-policy-mls-42.1.18-4.el10_2.3.noarch         10/10
    selinux:   Running scriptlet: selinux-policy-mls-extra-42.1.18-4.el10_2.3.noarch   10/10
    selinux:   Running scriptlet: setools-console-4.6.0-3.el10.x86_64                  10/10
    selinux:
    selinux: Installed:
    selinux:   mcstrans-3.10-1.el10.x86_64
    selinux:   policycoreutils-newrole-3.10-2.el10_2.x86_64
    selinux:   policycoreutils-python-utils-3.10-2.el10_2.noarch
    selinux:   python3-dasbus-1.7-8.el10.noarch
    selinux:   python3-libxml2-2.12.5-10.el10_2.2.x86_64
    selinux:   selinux-policy-mls-42.1.18-4.el10_2.3.noarch
    selinux:   selinux-policy-mls-extra-42.1.18-4.el10_2.3.noarch
    selinux:   setools-console-4.6.0-3.el10.x86_64
    selinux:   setroubleshoot-plugins-3.3.14-11.el10.noarch
    selinux:   setroubleshoot-server-3.3.35-4.el10.x86_64
    selinux:
    selinux: Complete!
    selinux: Job for nginx.service failed because the control process exited with error code.
    selinux: See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
    selinux: × nginx.service - The nginx HTTP and reverse proxy server
    selinux:      Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
    selinux:      Active: failed (Result: exit-code) since Thu 2026-08-27 12:50:55 UTC; 30ms ago
    selinux:  Invocation: b0ca40faf50340dd9d3efe627cc6352f
    selinux:     Process: 5065 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    selinux:     Process: 5067 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
    selinux:    Mem peak: 3.2M
    selinux:         CPU: 41ms
    selinux:
    selinux: Aug 27 12:50:54 selinux systemd[1]: Starting nginx.service - The nginx HTTP and reverse proxy server...
    selinux: Aug 27 12:50:55 selinux nginx[5067]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    selinux: Aug 27 12:50:55 selinux nginx[5067]: nginx: [emerg] bind() to 0.0.0.0:4881 failed (13: Permission denied)
    selinux: Aug 27 12:50:55 selinux nginx[5067]: nginx: configuration file /etc/nginx/nginx.conf test failed
    selinux: Aug 27 12:50:55 selinux systemd[1]: nginx.service: Control process exited, code=exited, status=1/FAILURE
    selinux: Aug 27 12:50:55 selinux systemd[1]: nginx.service: Failed with result 'exit-code'.
    selinux: Aug 27 12:50:55 selinux systemd[1]: Failed to start nginx.service - The nginx HTTP and reverse proxy server.
The SSH command responded with a non-zero exit status. Vagrant
assumes that this means the command failed. The output for this command
should be in the log above. Please read the output to determine what
went wrong.
PS E:\ucheba> vagrant ssh
```
## 2.Запуск nginx на нестандартном порту 3-мя разными способами 
### способ 1

```console
[vagrant@selinux ~]$ sudo -i
[root@selinux ~]# systemctl status firewalld
Unit firewalld.service could not be found.
[root@selinux ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@selinux ~]# getenforce
Enforcing
[root@selinux ~]# cat /var/log/audit/audit.log | grep 4881
type=AVC msg=audit(1787835055.056:728): avc:  denied  { name_bind } for  pid=5067 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0
[root@selinux ~]# grep 1787835055.056:728 /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1787835055.056:728): avc:  denied  { name_bind } for  pid=5067 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0

        Was caused by:
        The boolean nis_enabled was set incorrectly.
        Description:
        Allow nis to enabled

        Allow access by executing:
        # setsebool -P nis_enabled 1
[root@selinux ~]# setsebool -P nis_enabled 1
[root@selinux ~]# systemctl restart nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Fri 2026-08-28 09:23:33 UTC; 11s ago
 Invocation: 4f75f6ec08504aebbedfa28f9cf4a135
    Process: 6607 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 6609 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 6610 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 6612 (nginx)
      Tasks: 3 (limit: 11837)
     Memory: 3.1M (peak: 3.2M)
        CPU: 76ms
     CGroup: /system.slice/nginx.service
             ├─6612 "nginx: master process /usr/sbin/nginx"
             ├─6613 "nginx: worker process"
             └─6614 "nginx: worker process"

Aug 28 09:23:33 selinux systemd[1]: Starting nginx.service - The nginx HTTP and reverse proxy server...
Aug 28 09:23:33 selinux nginx[6609]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Aug 28 09:23:33 selinux nginx[6609]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Aug 28 09:23:33 selinux systemd[1]: Started nginx.service - The nginx HTTP and reverse proxy server.
[root@selinux ~]# getsebool -a | grep nis_enabled
nis_enabled --> on
[root@selinux ~]#  setsebool -P nis_enabled off
[root@selinux ~]# getsebool -a | grep nis_enabled
nis_enabled --> off
```
### способ 2
```console
[root@selinux ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
http_port_t                    udp      80, 443
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
[root@selinux ~]# semanage port -a -t http_port_t -p tcp 4881
[root@selinux ~]# semanage port -l | grep  http_port_t
http_port_t                    tcp      4881, 80, 81, 443, 488, 8008, 8009, 8443, 9000
http_port_t                    udp      80, 443
pegasus_http_port_t            tcp      5988
[root@selinux ~]# systemctl restart nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Fri 2026-08-28 09:30:02 UTC; 9s ago
 Invocation: 318b929f05f847cba68323c421504bc4
    Process: 6649 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 6651 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 6652 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 6654 (nginx)
      Tasks: 3 (limit: 11837)
     Memory: 3.1M (peak: 3.2M)
        CPU: 66ms
     CGroup: /system.slice/nginx.service
             ├─6654 "nginx: master process /usr/sbin/nginx"
             ├─6655 "nginx: worker process"
             └─6656 "nginx: worker process"

Aug 28 09:30:02 selinux systemd[1]: Starting nginx.service - The nginx HTTP and reverse proxy server...
Aug 28 09:30:02 selinux nginx[6651]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Aug 28 09:30:02 selinux nginx[6651]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Aug 28 09:30:02 selinux systemd[1]: Started nginx.service - The nginx HTTP and reverse proxy server.
[root@selinux ~]# semanage port -d -t http_port_t -p tcp 4881
[root@selinux ~]# semanage port -l | grep  http_port_t
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
http_port_t                    udp      80, 443
pegasus_http_port_t            tcp      5988
```
### способ 3

```console
[root@selinux ~]# systemctl restart nginx
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
[root@selinux ~]# systemctl status nginx
× nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: failed (Result: exit-code) since Fri 2026-08-28 09:31:11 UTC; 21s ago
   Duration: 1min 8.183s
 Invocation: 01773c21db0a4b4eb615e1ac2e17a17f
    Process: 6673 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 6675 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
   Mem peak: 2.9M
        CPU: 43ms

Aug 28 09:31:11 selinux systemd[1]: Starting nginx.service - The nginx HTTP and reverse proxy server...
Aug 28 09:31:11 selinux nginx[6675]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Aug 28 09:31:11 selinux nginx[6675]: nginx: [emerg] bind() to 0.0.0.0:4881 failed (13: Permission denied)
Aug 28 09:31:11 selinux nginx[6675]: nginx: configuration file /etc/nginx/nginx.conf test failed
Aug 28 09:31:11 selinux systemd[1]: nginx.service: Control process exited, code=exited, status=1/FAILURE
Aug 28 09:31:11 selinux systemd[1]: nginx.service: Failed with result 'exit-code'.
Aug 28 09:31:11 selinux systemd[1]: Failed to start nginx.service - The nginx HTTP and reverse proxy server.
[root@selinux ~]# grep nginx /var/log/audit/audit.log
type=ADD_GROUP msg=audit(1787835027.907:606): pid=4875 uid=0 auid=1000 ses=3 subj=unconfined_u:unconfined_r:groupadd_t:s0-s0:c0.c1023 msg='op=add-group id=991 exe="/usr/sbin/groupadd" hostname=? addr=? terminal=? res=success'UID="root" AUID="vagrant" ID="nginx"
type=GRP_MGMT msg=audit(1787835027.921:607): pid=4875 uid=0 auid=1000 ses=3 subj=unconfined_u:unconfined_r:groupadd_t:s0-s0:c0.c1023 msg='op=add-shadow-group id=991 exe="/usr/sbin/groupadd" hostname=? addr=? terminal=? res=success'UID="root" AUID="vagrant" ID="nginx"
type=ADD_USER msg=audit(1787835027.976:608): pid=4878 uid=0 auid=1000 ses=3 subj=unconfined_u:unconfined_r:useradd_t:s0-s0:c0.c1023 msg='op=add-user acct="nginx" exe="/usr/sbin/useradd" hostname=? addr=? terminal=? res=success'UID="root" AUID="vagrant"
type=SOFTWARE_UPDATE msg=audit(1787835029.393:642): pid=4859 uid=0 auid=1000 ses=3 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=install sw="nginx-filesystem-2:1.26.3-6.el10_2.6.noarch" sw_type=rpm key_enforce=0 gpg_res=1 root_dir="/" comm="yum" exe="/usr/bin/python3.12" hostname=? addr=? terminal=? res=success'UID="root" AUID="vagrant"
type=SOFTWARE_UPDATE msg=audit(1787835029.393:643): pid=4859 uid=0 auid=1000 ses=3 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=install sw="nginx-core-2:1.26.3-6.el10_2.6.x86_64" sw_type=rpm key_enforce=0 gpg_res=1 root_dir="/" comm="yum" exe="/usr/bin/python3.12" hostname=? addr=? terminal=? res=success'UID="root" AUID="vagrant"
type=SOFTWARE_UPDATE msg=audit(1787835029.393:645): pid=4859 uid=0 auid=1000 ses=3 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=install sw="nginx-2:1.26.3-6.el10_2.6.x86_64" sw_type=rpm key_enforce=0 gpg_res=1 root_dir="/" comm="yum" exe="/usr/bin/python3.12" hostname=? addr=? terminal=? res=success'UID="root" AUID="vagrant"
type=AVC msg=audit(1787835055.056:728): avc:  denied  { name_bind } for  pid=5067 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0
type=SYSCALL msg=audit(1787835055.056:728): arch=c000003e syscall=49 success=no exit=-13 a0=6 a1=564e048aec58 a2=10 a3=7ffc55f0b7e0 items=0 ppid=1 pid=5067 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="nginx" exe="/usr/sbin/nginx" subj=system_u:system_r:httpd_t:s0 key=(null)ARCH=x86_64 SYSCALL=bind AUID="unset" UID="root" GID="root" EUID="root" SUID="root" FSUID="root" EGID="root" SGID="root" FSGID="root"
type=SERVICE_START msg=audit(1787835055.059:729): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=failed'UID="root" AUID="unset"
type=SERVICE_START msg=audit(1787909013.594:871): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'UID="root" AUID="unset"
type=SERVICE_STOP msg=audit(1787909402.852:880): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'UID="root" AUID="unset"
type=SERVICE_START msg=audit(1787909402.981:881): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'UID="root" AUID="unset"
type=SERVICE_STOP msg=audit(1787909471.190:885): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'UID="root" AUID="unset"
type=AVC msg=audit(1787909471.293:886): avc:  denied  { name_bind } for  pid=6675 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0
type=SYSCALL msg=audit(1787909471.293:886): arch=c000003e syscall=49 success=no exit=-13 a0=6 a1=564526066c58 a2=10 a3=7ffce1685f00 items=0 ppid=1 pid=6675 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="nginx" exe="/usr/sbin/nginx" subj=system_u:system_r:httpd_t:s0 key=(null)ARCH=x86_64 SYSCALL=bind AUID="unset" UID="root" GID="root" EUID="root" SUID="root" FSUID="root" EGID="root" SGID="root" FSGID="root"
type=SERVICE_START msg=audit(1787909471.299:887): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=failed'UID="root" AUID="unset"
[root@selinux ~]# grep nginx /var/log/audit/audit.log | audit2allow -M nginx
******************** IMPORTANT ***********************
To make this policy package active, execute:

semodule -i nginx.pp

[root@selinux ~]# semodule -i nginx.pp
[root@selinux ~]# systemctl start nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Fri 2026-08-28 09:34:31 UTC; 5s ago
 Invocation: f9e32933b27b4fcf8db6ea0ab09c8437
    Process: 6698 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 6700 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 6701 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 6704 (nginx)
      Tasks: 3 (limit: 11837)
     Memory: 3.1M (peak: 3.2M)
        CPU: 63ms
     CGroup: /system.slice/nginx.service
             ├─6704 "nginx: master process /usr/sbin/nginx"
             ├─6705 "nginx: worker process"
             └─6706 "nginx: worker process"

Aug 28 09:34:31 selinux systemd[1]: Starting nginx.service - The nginx HTTP and reverse proxy server...
Aug 28 09:34:31 selinux nginx[6700]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Aug 28 09:34:31 selinux nginx[6700]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Aug 28 09:34:31 selinux systemd[1]: Started nginx.service - The nginx HTTP and reverse proxy server.
[root@selinux ~]# semanage port -l | grep  http_port_t
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
http_port_t                    udp      80, 443
pegasus_http_port_t            tcp      5988
[root@selinux ~]# getsebool -a | grep nis_enabled
nis_enabled --> off
[root@selinux ~]# semodule -l
<details>
<summary>📦 Вывод:</summary> <code>
accountsd
acct
afs
afterburn
aide
alsa
amanda
anaconda
antivirus
apache
apcupsd
apm
application
arpwatch
asterisk
auditadm
authlogin
automount
avahi
awstats
bacula
base
bind
bitlbee
blkmapd
blueman
bluetooth
boinc
boltd
boothd
bootloader
bootupd
brctl
brltty
bugzilla
cachefilesd
calamaris
callweaver
canna
ccs
cdrecord
certmaster
certmonger
certwatch
cfengine
cgroup
chrome
chronyd
cifsutils
cinder
cipe
clock
clogd
cloudform
cmirrord
cobbler
collectd
colord
comsat
condor
conman
conntrackd
consolekit
coreos_installer
couchdb
courier
cpucontrol
cpufreqselector
cpuplug
cron
ctdb
cups
cvs
cyphesis
cyrus
daemontools
dbadm
dbskk
dbus
dcc
ddclient
denyhosts
devicekit
dhcp
dictd
dirsrv
distcc
dmesg
dmidecode
dnsmasq
dnssec
dovecot
drbd
dspam
entropyd
exim
fcoe
fdo
fedoratp
fetchmail
finger
firewalld
firewallgui
firstboot
fprintd
freeipmi
freqset
fstools
ftp
fwupd
games
gdomap
geoclue
getty
git
gitosis
glance
glusterd
gnome
gnome_remote_desktop
gpg
gpm
gpsd
gssproxy
guest
hddtemp
hostapd
hostname
hsqldb
hwloc
hypervkvp
ibacm
ica
icecast
iiosensorproxy
inetd
init
inn
insights_client
iodine
iotop
ipmievd
ipsec
iptables
irc
irqbalance
iscsi
isns
jabber
jetty
jockey
journalctl
kafs
kdump
kdumpgui
keepalived
kerberos
keyboardd
keystone
keyutils
kismet
kpatch
ksmtuned
ktalk
ktls
l2tp
ldap
libraries
likewise
lircd
livecd
lldpad
loadkeys
locallogin
lockdev
logadm
logging
logrotate
logwatch
lpd
lsm
lttng-tools
lvm
mailscanner
man2html
mandb
mcelog
mediawiki
memcached
milter
minidlna
minissdpd
miscfiles
mock
modemmanager
modutils
mojomojo
mon_statd
motion
mount
mozilla
mpd
mplayer
mptcpd
mrtg
mta
munin
mysql
mythtv
nagios
namespace
ncftool
netlabel
netutils
networkmanager
nginx
ninfod
nis
nova
nscd
nsd
nslcd
ntop
numad
nut
nvme_stas
nx
obex
oddjob
opafm
opendnssec
openfortivpn
openhpid
openshift
openshift-origin
opensm
openvpn
openvswitch
openwsman
oracleasm
osad
pads
passenger
pcm
pcmcia
pcscd
pdns
pegasus
permissivedomains
pesign
pingd
pkcs
pki
plymouthd
podsleuth
policykit
polipo
portmap
portreserve
postfix
postgresql
postgrey
powerprofiles
ppp
prelink
privoxy
procmail
prosody
psad
ptchown
pulseaudio
puppet
pwauth
qatlib
qgs
qmail
qpid
quantum
quota
rabbitmq
radius
radvd
raid
rasdaemon
rdisc
readahead
realmd
redfish-finder
redis
remotelogin
rhcd
rhcs
rhgb
rhnsd
rhsmcertd
ricci
rkhunter
rlogin
rngd
roundup
rpc
rpcbind
rpm
rrdcached
rshd
rshim
rssh
rsync
rtas
rtkit
rwho
samba
sambagui
sandboxX
sanlock
sap
sasl
sbd
sblim
screen
secadm
sectoolm
selinuxutil
sendmail
sensord
setrans
setroubleshoot
seunshare
shorewall
slocate
slpd
slrnpull
smartmon
smokeping
smoltclient
snapper
snmp
snort
sosreport
soundserver
spamassassin
speech-dispatcher
squid
ssh
sslh
sssd
staff
stalld
stapserver
stratisd
stunnel
su
sudo
svnserve
swift
switcheroo
sysadm
sysadm_secadm
sysnetwork
sysstat
systemd
systemd-homed
tangd
targetd
tcpd
tcsd
telepathy
telnet
tftp
tgtd
thin
thumb
tlp
tmpreaper
tomcat
tor
tuned
udev
ulogd
uml
unconfined
unconfineduser
unlabelednet
unprivuser
updfstab
usbmodules
usbmuxd
userdomain
userhelper
usermanage
usernetctl
uucp
uuidd
varnishd
vdagent
vhostmd
virt
virt_supplementary
vlock
vmtools
vmware
vnstatd
vpn
w3c
watchdog
wdmd
webadm
webalizer
wine
wireguard
wireshark
xen
xguest
xserver
zabbix
zarafa
zebra
zoneminder
zosremote </code> 
</details>
[root@selinux ~]# semodule -r nginx
libsemanage.semanage_direct_remove_key: Removing last nginx module (no other nginx module exists at another priority).
[root@selinux ~]# exit
logout
[vagrant@selinux ~]$ exit
logout
```
