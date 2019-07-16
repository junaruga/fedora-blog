# Test modules/ruby

Refer below documents to test.

* https://docs.fedoraproject.org/en-US/modularity/using-modules/
  Installing modules
  Switching module streams

* https://developers.redhat.com/blog/2019/05/16/modular-perl-in-red-hat-enterprise-linux-8/


## Condition

* Fedora 30

## Preparation

Create below file referring https://src.fedoraproject.org/modules/ruby/blob/2.6/f/ruby.yaml /data/api/rpms element.

```
$ cat rpm_list.txt
ruby
ruby-devel
ruby-libs
rubygems
rubygems-devel
rubygem-bigdecimal
rubygem-bundler
rubygem-did_you_mean
rubygem-io-console
rubygem-irb
rubygem-json
rubygem-minitest
rubygem-net-telnet
rubygem-openssl
rubygem-power_assert
rubygem-psych
rubygem-rake
rubygem-rdoc
rubygem-test-unit
rubygem-xmlrpc
rubygem-abrt
rubygem-mysql2
rubygem-pg
rubygem-bson
rubygem-mongo
```

Then install non-moular RPMs that are included in the Ruby 2.6 module.

```
$ sudo dnf -y install $(cat rpm_list.txt)

$ sudo dnf -y upgrade $(cat rpm_list.txt)

$ rpm -q $(cat rpm_list.txt)
ruby-2.6.3-120.fc30.x86_64
ruby-devel-2.6.3-120.fc30.x86_64
ruby-libs-2.6.3-120.fc30.x86_64
rubygems-3.0.3-120.fc30.noarch
rubygems-devel-3.0.3-120.fc30.noarch
rubygem-bigdecimal-1.4.1-120.fc30.x86_64
rubygem-bundler-1.17.2-120.fc30.noarch
rubygem-did_you_mean-1.3.0-120.fc30.noarch
rubygem-io-console-0.4.7-120.fc30.x86_64
rubygem-irb-1.0.0-120.fc30.noarch
rubygem-json-2.2.0-200.fc30.x86_64
rubygem-minitest-5.11.3-203.fc30.noarch
rubygem-net-telnet-0.2.0-120.fc30.noarch
rubygem-openssl-2.1.2-120.fc30.x86_64
rubygem-power_assert-1.1.4-200.fc30.noarch
rubygem-psych-3.1.0-120.fc30.x86_64
rubygem-rake-12.3.2-201.fc30.noarch
rubygem-rdoc-6.1.0-120.fc30.noarch
rubygem-test-unit-3.3.3-200.fc30.noarch
rubygem-xmlrpc-0.3.0-120.fc30.noarch
rubygem-abrt-0.3.0-6.fc30.noarch
rubygem-mysql2-0.5.2-3.fc30.x86_64
rubygem-pg-1.1.4-3.fc30.x86_64
rubygem-bson-4.3.0-6.fc30.x86_64
rubygem-mongo-2.6.2-3.fc30.noarch
```

## Test

```
$ sudo dnf -y module reset ruby
```

Then, the result is not changed, and same with before.

```
$ rpm -q $(cat rpm_list.txt)
ruby-2.6.3-120.fc30.x86_64
...
```

```
$ sudo dnf module install ruby:2.5
```

Then, the result is not changed, and same with before.

```
$ rpm -q $(cat rpm_list.txt)
ruby-2.6.3-120.fc30.x86_64
...
```

You see `ruby 2.5 [e]`, that is enabled.

```
$ dnf module list
...
Fedora Modular 30 - x86_64
...
ruby                master         default       An interpreter of object-oriented scripting l
                                                 anguage
...

Fedora Modular 30 - x86_64 - Updates
...
ruby                master         default       An interpreter of object-oriented scripting l
                                                 anguage
ruby                2.5 [e]        default       An interpreter of object-oriented scripting l
                                                 anguage
ruby                2.6            default       An interpreter of object-oriented scripting l
                                                 anguage
...
```

Run `dnf upgrade` to upgrade to install RPMs in modules/ruby:2.5
`dnf update` is a deprecated alias for the upgrade command according to `man dnf`.

```
$ sudo dnf upgrade $(cat rpm_list.txt)
...
==============================================================================================
 Package           Arch      Version                                 Repository          Size
==============================================================================================
Upgrading:
 rubygem-abrt      noarch    0.3.0-6.module_f30+4125+c677839c        updates-modular     11 k
 rubygem-mongo     noarch    2.6.2-3.module_f30+4125+c677839c        updates-modular    240 k
Skipping packages with conflicts:
(add '--best --allowerasing' to command line to force their upgrade):
 ruby-libs         x86_64    2.5.5-105.module_f30+4125+c677839c      updates-modular    2.7 M
Skipping packages with broken dependencies:
 rubygem-bson      x86_64    4.3.0-6.module_f30+4125+c677839c        updates-modular     50 k
 rubygem-mysql2    x86_64    0.5.2-3.module_f30+4125+c677839c        updates-modular     43 k
 rubygem-pg        x86_64    1.1.4-3.module_f30+4125+c677839c        updates-modular     89 k

Transaction Summary
==============================================================================================
Upgrade  2 Packages
Skip     4 Packages

Total download size: 251 k
Is this ok [y/N]:
```
```
$ sudo dnf downgrade $(cat rpm_list.txt)
...
Error:
 Problem 1: installed package rubygem-irb-1.0.0-120.fc30.noarch obsoletes ruby-irb < 2.6.3-120.fc30 provided by ruby-irb-2.5.5-105.module_f30+4125+c677839c.noarch
...
(try to add '--allowerasing' to command line to replace conflicting packages or '--skip-broken' to skip uninstallable packages)
```

```
$ sudo dnf --allowerasing downgrade $(cat rpm_list.txt)
...
==============================================================================================
 Package              Arch   Version                                 Repository          Size
==============================================================================================
Upgrading:
 rubygem-bson         x86_64 4.3.0-6.module_f30+4125+c677839c        updates-modular     50 k
 rubygem-mysql2       x86_64 0.5.2-3.module_f30+4125+c677839c        updates-modular     43 k
 rubygem-pg           x86_64 1.1.4-3.module_f30+4125+c677839c        updates-modular     89 k
Installing dependencies:
 ruby-irb             noarch 2.5.5-105.module_f30+4125+c677839c      updates-modular     57 k
Removing dependent packages:
 bughunting-server    noarch 2018.0-1.git.31.253dd14.fc29            @thozza-bughunting 294 k
 rubygem-childprocess noarch 0.5.9-5.fc29                            @fedora             91 k
 rubygem-domain_name  noarch 0.5.20180417-3.fc30                     @fedora            230 k
 rubygem-ffi          x86_64 1.10.0-2.fc30                           @fedora            480 k
 rubygem-fog-libvirt  noarch 0.5.0-3.fc30                            @fedora             70 k
 rubygem-fog-xml      noarch 0.1.3-2.fc30                            @fedora            8.7 k
 rubygem-http-cookie  noarch 1.0.3-6.fc30                            @fedora             73 k
 rubygem-irb          noarch 1.0.0-120.fc30                          @updates           169 k
 rubygem-listen       noarch 3.1.5-7.fc30                            @fedora             63 k
 rubygem-nokogiri     x86_64 1.10.2-1.fc30                           @fedora            568 k
 rubygem-rb-inotify   noarch 0.10.0-1.fc30                           @fedora             25 k
 rubygem-rest-client  noarch 2.0.0-6.fc30                            @fedora             81 k
 rubygem-ruby-libvirt x86_64 0.7.1-5.fc30                            @fedora            295 k
 rubygem-sqlite3      x86_64 1.3.13-11.fc30                          @fedora            109 k
 rubygem-unf          noarch 0.1.4-13.fc30                           @fedora            6.6 k
 rubygem-unf_ext      x86_64 0.0.7.5-4.fc30                          @fedora            428 k
 vagrant              noarch 2.2.3-1.fc30                            @fedora            2.1 M
 vagrant-libvirt      noarch 0.0.45-1.fc30                           @fedora            214 k
 vagrant-sshfs        noarch 1.3.1-3.fc30                            @fedora             82 k
 weechat              x86_64 2.4-1.fc30                              @updates            21 M
Downgrading:
 ruby                 x86_64 2.5.5-105.module_f30+4125+c677839c      updates-modular     40 k
 ruby-devel           x86_64 2.5.5-105.module_f30+4125+c677839c      updates-modular     81 k
 ruby-libs            x86_64 2.5.5-105.module_f30+4125+c677839c      updates-modular    2.7 M
 rubygem-bigdecimal   x86_64 1.3.4-105.module_f30+4125+c677839c      updates-modular     50 k
 rubygem-bundler      noarch 1.16.1-5.module_f30+4125+c677839c       updates-modular    347 k
 rubygem-did_you_mean noarch 1.2.0-105.module_f30+4125+c677839c      updates-modular     45 k
 rubygem-io-console   x86_64 0.4.6-105.module_f30+4125+c677839c      updates-modular     20 k
 rubygem-json         x86_64 2.1.0-105.module_f30+4125+c677839c      updates-modular     43 k
 rubygem-minitest     noarch 5.10.3-105.module_f30+4125+c677839c     updates-modular     77 k
 rubygem-net-telnet   noarch 0.1.1-105.module_f30+4125+c677839c      updates-modular     25 k
 rubygem-openssl      x86_64 2.1.2-105.module_f30+4125+c677839c      updates-modular    136 k
 rubygem-power_assert noarch 1.1.1-105.module_f30+4125+c677839c      updates-modular     24 k
 rubygem-psych        x86_64 3.0.2-105.module_f30+4125+c677839c      updates-modular     49 k
 rubygem-rake         noarch 12.3.0-105.module_f30+4125+c677839c     updates-modular     94 k
 rubygem-rdoc         noarch 6.0.1-105.module_f30+4125+c677839c      updates-modular    441 k
 rubygem-test-unit    noarch 3.2.7-105.module_f30+4125+c677839c      updates-modular    137 k
 rubygem-xmlrpc       noarch 0.3.0-105.module_f30+4125+c677839c      updates-modular     36 k
 rubygems             noarch 2.7.6.2-105.module_f30+4125+c677839c    updates-modular    263 k
 rubygems-devel       noarch 2.7.6.2-105.module_f30+4125+c677839c    updates-modular     15 k
...
Upgraded:
  rubygem-bson-4.3.0-6.module_f30+4125+c677839c.x86_64                                        
  rubygem-mysql2-0.5.2-3.module_f30+4125+c677839c.x86_64                                      
  rubygem-pg-1.1.4-3.module_f30+4125+c677839c.x86_64                                          

Downgraded:
  ruby-2.5.5-105.module_f30+4125+c677839c.x86_64                                              
  ruby-devel-2.5.5-105.module_f30+4125+c677839c.x86_64                                        
  ruby-libs-2.5.5-105.module_f30+4125+c677839c.x86_64                                         
  rubygem-bigdecimal-1.3.4-105.module_f30+4125+c677839c.x86_64                                
  rubygem-bundler-1.16.1-5.module_f30+4125+c677839c.noarch                                    
  rubygem-did_you_mean-1.2.0-105.module_f30+4125+c677839c.noarch                              
  rubygem-io-console-0.4.6-105.module_f30+4125+c677839c.x86_64                                
  rubygem-json-2.1.0-105.module_f30+4125+c677839c.x86_64                                      
  rubygem-minitest-5.10.3-105.module_f30+4125+c677839c.noarch                                 
  rubygem-net-telnet-0.1.1-105.module_f30+4125+c677839c.noarch                                
  rubygem-openssl-2.1.2-105.module_f30+4125+c677839c.x86_64                                   
  rubygem-power_assert-1.1.1-105.module_f30+4125+c677839c.noarch                              
  rubygem-psych-3.0.2-105.module_f30+4125+c677839c.x86_64                                     
  rubygem-rake-12.3.0-105.module_f30+4125+c677839c.noarch                                     
  rubygem-rdoc-6.0.1-105.module_f30+4125+c677839c.noarch                                      
  rubygem-test-unit-3.2.7-105.module_f30+4125+c677839c.noarch                                 
  rubygem-xmlrpc-0.3.0-105.module_f30+4125+c677839c.noarch                                    
  rubygems-2.7.6.2-105.module_f30+4125+c677839c.noarch                                        
  rubygems-devel-2.7.6.2-105.module_f30+4125+c677839c.noarch                                  

Installed:
  ruby-irb-2.5.5-105.module_f30+4125+c677839c.noarch                                          

Removed:
  bughunting-server-2018.0-1.git.31.253dd14.fc29.noarch                                       
  rubygem-childprocess-0.5.9-5.fc29.noarch                                                    
  rubygem-domain_name-0.5.20180417-3.fc30.noarch                                              
  rubygem-ffi-1.10.0-2.fc30.x86_64                                                            
  rubygem-fog-libvirt-0.5.0-3.fc30.noarch                                                     
  rubygem-fog-xml-0.1.3-2.fc30.noarch                                                         
  rubygem-http-cookie-1.0.3-6.fc30.noarch                                                     
  rubygem-irb-1.0.0-120.fc30.noarch                                                           
  rubygem-listen-3.1.5-7.fc30.noarch                                                          
  rubygem-nokogiri-1.10.2-1.fc30.x86_64                                                       
  rubygem-rb-inotify-0.10.0-1.fc30.noarch                                                     
  rubygem-rest-client-2.0.0-6.fc30.noarch                                                     
  rubygem-ruby-libvirt-0.7.1-5.fc30.x86_64                                                    
  rubygem-sqlite3-1.3.13-11.fc30.x86_64                                                       
  rubygem-unf-0.1.4-13.fc30.noarch                                                            
  rubygem-unf_ext-0.0.7.5-4.fc30.x86_64                                                       
  vagrant-2.2.3-1.fc30.noarch                                                                 
  vagrant-libvirt-0.0.45-1.fc30.noarch                                                        
  vagrant-sshfs-1.3.1-3.fc30.noarch                                                           
  weechat-2.4-1.fc30.x86_64                                                                   

Complete!
```

On modules/ruby:2.6 irb is rubygem-irb package.
On modules/ruby:2.5 irb is ruby-irb package.

RPMs in modules/ruby:2.5 are installed.

```
$ rpm -q $(cat rpm_list.txt) ruby-irb
ruby-2.5.5-105.module_f30+4125+c677839c.x86_64
ruby-devel-2.5.5-105.module_f30+4125+c677839c.x86_64
ruby-libs-2.5.5-105.module_f30+4125+c677839c.x86_64
rubygems-2.7.6.2-105.module_f30+4125+c677839c.noarch
rubygems-devel-2.7.6.2-105.module_f30+4125+c677839c.noarch
rubygem-bigdecimal-1.3.4-105.module_f30+4125+c677839c.x86_64
rubygem-bundler-1.16.1-5.module_f30+4125+c677839c.noarch
rubygem-did_you_mean-1.2.0-105.module_f30+4125+c677839c.noarch
rubygem-io-console-0.4.6-105.module_f30+4125+c677839c.x86_64
package rubygem-irb is not installed
rubygem-json-2.1.0-105.module_f30+4125+c677839c.x86_64
rubygem-minitest-5.10.3-105.module_f30+4125+c677839c.noarch
rubygem-net-telnet-0.1.1-105.module_f30+4125+c677839c.noarch
rubygem-openssl-2.1.2-105.module_f30+4125+c677839c.x86_64
rubygem-power_assert-1.1.1-105.module_f30+4125+c677839c.noarch
rubygem-psych-3.0.2-105.module_f30+4125+c677839c.x86_64
rubygem-rake-12.3.0-105.module_f30+4125+c677839c.noarch
rubygem-rdoc-6.0.1-105.module_f30+4125+c677839c.noarch
rubygem-test-unit-3.2.7-105.module_f30+4125+c677839c.noarch
rubygem-xmlrpc-0.3.0-105.module_f30+4125+c677839c.noarch
rubygem-abrt-0.3.0-6.fc30.noarch
rubygem-mysql2-0.5.2-3.module_f30+4125+c677839c.x86_64
rubygem-pg-1.1.4-3.module_f30+4125+c677839c.x86_64
rubygem-bson-4.3.0-6.module_f30+4125+c677839c.x86_64
rubygem-mongo-2.6.2-3.fc30.noarch
ruby-irb-2.5.5-105.module_f30+4125+c677839c.noarch
```

Switch modules/ruby:2.5 to modules/ruby:2.6.

```
$ dnf module list ruby
Last metadata expiration check: 0:58:39 ago on Tue 16 Jul 2019 06:39:33 PM CEST.
Fedora Modular 30 - x86_64
Name      Stream       Profiles      Summary                                                  
ruby      master       default       An interpreter of object-oriented scripting language     

Fedora Modular 30 - x86_64 - Updates
Name      Stream       Profiles      Summary                                                  
ruby      master       default       An interpreter of object-oriented scripting language     
ruby      2.5 [e]      default       An interpreter of object-oriented scripting language     
ruby      2.6          default       An interpreter of object-oriented scripting language     

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```

```
$ sudo dnf -y module reset ruby
```

There is no enabled modules/ruby.

```
$ dnf module list ruby
Last metadata expiration check: 0:59:31 ago on Tue 16 Jul 2019 06:39:33 PM CEST.
Fedora Modular 30 - x86_64
Name       Stream      Profiles      Summary                                                  
ruby       master      default       An interpreter of object-oriented scripting language     

Fedora Modular 30 - x86_64 - Updates
Name       Stream      Profiles      Summary                                                  
ruby       master      default       An interpreter of object-oriented scripting language     
ruby       2.5         default       An interpreter of object-oriented scripting language     
ruby       2.6         default       An interpreter of object-oriented scripting language     

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```

```
$ sudo dnf module install ruby:2.6
```

You see `ruby 2.6 [e]`, that is enabled.

```
$ dnf module list ruby
Last metadata expiration check: 1:00:58 ago on Tue 16 Jul 2019 06:39:33 PM CEST.
Fedora Modular 30 - x86_64
Name      Stream       Profiles      Summary                                                  
ruby      master       default       An interpreter of object-oriented scripting language     

Fedora Modular 30 - x86_64 - Updates
Name      Stream       Profiles      Summary                                                  
ruby      master       default       An interpreter of object-oriented scripting language     
ruby      2.5          default       An interpreter of object-oriented scripting language     
ruby      2.6 [e]      default       An interpreter of object-oriented scripting language     

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```

Run `dnf upgrade` to upgrade to install RPMs in modules/ruby:2.6

```
$ sudo dnf upgrade $(cat rpm_list.txt)
...
==============================================================================================
 Package               Arch    Version                                 Repository        Size
==============================================================================================
Upgrading:
 ruby                  x86_64  2.6.3-120.module_f30+4422+f50e013d      updates-modular   42 k
 ruby-devel            x86_64  2.6.3-120.module_f30+4422+f50e013d      updates-modular  199 k
 ruby-libs             x86_64  2.6.3-120.module_f30+4422+f50e013d      updates-modular  2.9 M
 rubygem-abrt          noarch  0.3.0-6.module_f30+3838+0c262d71        updates-modular   11 k
 rubygem-bigdecimal    x86_64  1.4.1-120.module_f30+4422+f50e013d      updates-modular   53 k
 rubygem-bson          x86_64  4.5.0-1.module_f30+4422+f50e013d        updates-modular   52 k
 rubygem-bundler       noarch  1.17.2-120.module_f30+4422+f50e013d     updates-modular  310 k
 rubygem-did_you_mean  noarch  1.3.0-120.module_f30+4422+f50e013d      updates-modular   38 k
 rubygem-io-console    x86_64  0.4.7-120.module_f30+4422+f50e013d      updates-modular   21 k
 rubygem-json          x86_64  2.1.0-120.module_f30+4422+f50e013d      updates-modular   44 k
 rubygem-minitest      noarch  5.11.3-120.module_f30+4422+f50e013d     updates-modular   81 k
 rubygem-mongo         noarch  2.8.0-1.module_f30+4422+f50e013d        updates-modular  260 k
 rubygem-mysql2        x86_64  0.5.2-3.module_f30+4422+f50e013d        updates-modular   43 k
 rubygem-net-telnet    noarch  0.2.0-120.module_f30+4422+f50e013d      updates-modular   26 k
 rubygem-openssl       x86_64  2.1.2-120.module_f30+4422+f50e013d      updates-modular  137 k
 rubygem-pg            x86_64  1.1.4-3.module_f30+4422+f50e013d        updates-modular   89 k
 rubygem-power_assert  noarch  1.1.3-120.module_f30+4422+f50e013d      updates-modular   25 k
 rubygem-psych         x86_64  3.1.0-120.module_f30+4422+f50e013d      updates-modular   50 k
 rubygem-rake          noarch  12.3.2-120.module_f30+4422+f50e013d     updates-modular   96 k
 rubygem-rdoc          noarch  6.1.0-120.module_f30+4422+f50e013d      updates-modular  442 k
 rubygem-test-unit     noarch  3.2.9-120.module_f30+4422+f50e013d      updates-modular  140 k
 rubygem-xmlrpc        noarch  0.3.0-120.module_f30+4422+f50e013d      updates-modular   37 k
 rubygems              noarch  3.0.3-120.module_f30+4422+f50e013d      updates-modular  271 k
 rubygems-devel        noarch  3.0.3-120.module_f30+4422+f50e013d      updates-modular   16 k
Installing dependencies:
 rubygem-irb           noarch  1.0.0-120.module_f30+4422+f50e013d      updates-modular   67 k
     replacing  ruby-irb.noarch 2.5.5-105.module_f30+4125+c677839c

Transaction Summary
==============================================================================================
Install   1 Package
Upgrade  24 Packages

Ctrl+C
```

```
$ sudo dnf -y upgrade $(cat rpm_list.txt)
...
Upgraded:
  ruby-2.6.3-120.module_f30+4422+f50e013d.x86_64                                              
  ruby-devel-2.6.3-120.module_f30+4422+f50e013d.x86_64                                        
  ruby-libs-2.6.3-120.module_f30+4422+f50e013d.x86_64                                         
  rubygem-abrt-0.3.0-6.module_f30+3838+0c262d71.noarch                                        
  rubygem-bigdecimal-1.4.1-120.module_f30+4422+f50e013d.x86_64                                
  rubygem-bson-4.5.0-1.module_f30+4422+f50e013d.x86_64                                        
  rubygem-bundler-1.17.2-120.module_f30+4422+f50e013d.noarch                                  
  rubygem-did_you_mean-1.3.0-120.module_f30+4422+f50e013d.noarch                              
  rubygem-io-console-0.4.7-120.module_f30+4422+f50e013d.x86_64                                
  rubygem-json-2.1.0-120.module_f30+4422+f50e013d.x86_64                                      
  rubygem-minitest-5.11.3-120.module_f30+4422+f50e013d.noarch                                 
  rubygem-mongo-2.8.0-1.module_f30+4422+f50e013d.noarch                                       
  rubygem-mysql2-0.5.2-3.module_f30+4422+f50e013d.x86_64                                      
  rubygem-net-telnet-0.2.0-120.module_f30+4422+f50e013d.noarch                                
  rubygem-openssl-2.1.2-120.module_f30+4422+f50e013d.x86_64                                   
  rubygem-pg-1.1.4-3.module_f30+4422+f50e013d.x86_64                                          
  rubygem-power_assert-1.1.3-120.module_f30+4422+f50e013d.noarch                              
  rubygem-psych-3.1.0-120.module_f30+4422+f50e013d.x86_64                                     
  rubygem-rake-12.3.2-120.module_f30+4422+f50e013d.noarch                                     
  rubygem-rdoc-6.1.0-120.module_f30+4422+f50e013d.noarch                                      
  rubygem-test-unit-3.2.9-120.module_f30+4422+f50e013d.noarch                                 
  rubygem-xmlrpc-0.3.0-120.module_f30+4422+f50e013d.noarch                                    
  rubygems-3.0.3-120.module_f30+4422+f50e013d.noarch                                          
  rubygems-devel-3.0.3-120.module_f30+4422+f50e013d.noarch                                    

Installed:
  rubygem-irb-1.0.0-120.module_f30+4422+f50e013d.noarch                                       

Complete!
```

```
$ rpm -q $(cat rpm_list.txt) ruby-irb
ruby-2.6.3-120.module_f30+4422+f50e013d.x86_64
ruby-devel-2.6.3-120.module_f30+4422+f50e013d.x86_64
ruby-libs-2.6.3-120.module_f30+4422+f50e013d.x86_64
rubygems-3.0.3-120.module_f30+4422+f50e013d.noarch
rubygems-devel-3.0.3-120.module_f30+4422+f50e013d.noarch
rubygem-bigdecimal-1.4.1-120.module_f30+4422+f50e013d.x86_64
rubygem-bundler-1.17.2-120.module_f30+4422+f50e013d.noarch
rubygem-did_you_mean-1.3.0-120.module_f30+4422+f50e013d.noarch
rubygem-io-console-0.4.7-120.module_f30+4422+f50e013d.x86_64
rubygem-irb-1.0.0-120.module_f30+4422+f50e013d.noarch
rubygem-json-2.1.0-120.module_f30+4422+f50e013d.x86_64
rubygem-minitest-5.11.3-120.module_f30+4422+f50e013d.noarch
rubygem-net-telnet-0.2.0-120.module_f30+4422+f50e013d.noarch
rubygem-openssl-2.1.2-120.module_f30+4422+f50e013d.x86_64
rubygem-power_assert-1.1.3-120.module_f30+4422+f50e013d.noarch
rubygem-psych-3.1.0-120.module_f30+4422+f50e013d.x86_64
rubygem-rake-12.3.2-120.module_f30+4422+f50e013d.noarch
rubygem-rdoc-6.1.0-120.module_f30+4422+f50e013d.noarch
rubygem-test-unit-3.2.9-120.module_f30+4422+f50e013d.noarch
rubygem-xmlrpc-0.3.0-120.module_f30+4422+f50e013d.noarch
rubygem-abrt-0.3.0-6.module_f30+3838+0c262d71.noarch
rubygem-mysql2-0.5.2-3.module_f30+4422+f50e013d.x86_64
rubygem-pg-1.1.4-3.module_f30+4422+f50e013d.x86_64
rubygem-bson-4.5.0-1.module_f30+4422+f50e013d.x86_64
rubygem-mongo-2.8.0-1.module_f30+4422+f50e013d.noarch
package ruby-irb is not installed
```

