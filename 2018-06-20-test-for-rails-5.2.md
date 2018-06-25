# Testing Ruby on Rails 5.2.0 on Fedora rawhide

This article is what I tested for Rails 5.2.0 on Fedora rawhide.

I referred this document [1] "How To Test" section for the testing use cases.

There are 3 use cases to test.

* To test Rails from upstream
  * with normal user: [OK]
  * with root user: [OK]
* To test only Rails itself
  * with normal user: [OK]
  * with root user: [OK]
* To test the complete feature including generating a new Rails app using RPM
  * with normal user: [OK]
  * with root user: [OK]

Tested Ruby.

```
<mock-chroot># rpm -q ruby
ruby-2.5.1-93.fc29.x86_64
```


## To test Rails from upstream

### Pre-condition

Check if your mock setting install_weak_deps is True(1).
See `man dnf.cnf` install_weak_deps for detail.

```
$ vi /etc/mock/default.cfg
...
[main]
...
install_weak_deps=1
...
```

```
$ mock --scrub=all
$ mock -i dnf
$ mock shell
```

Then on mock environment.

```
<mock-chroot># dnf install ruby-devel sqlite-devel nodejs zlib-devel
```

Check `bigdecimal` that is a weak dependency of ruby is intalled.
It is required to run Rails server.

```
# rpm -q rubygem-bigdecimal
rubygem-bigdecimal-1.3.4-93.fc29.x86_64
```

### Test with normal user

```
<mock-chroot># su - mockbuild
```

```
[mockbuild]$ gem env
RubyGems Environment:
  - RUBYGEMS VERSION: 2.7.6
  - RUBY VERSION: 2.5.1 (2018-03-29 patchlevel 57) [x86_64-linux]
  - INSTALLATION DIRECTORY: /usr/share/gems
  - USER INSTALLATION DIRECTORY: /builddir/.gem/ruby
  - RUBY EXECUTABLE: /usr/bin/ruby
  - EXECUTABLE DIRECTORY: /usr/bin
  - SPEC CACHE DIRECTORY: /builddir/.gem/specs
  - SYSTEM CONFIGURATION DIRECTORY: /etc
  - RUBYGEMS PLATFORMS:
    - ruby
    - x86_64-linux
  - GEM PATHS:
     - /usr/share/gems
     - /builddir/.gem/ruby
     - /usr/local/share/gems
  - GEM CONFIGURATION:
     - :update_sources => true
     - :verbose => true
     - :backtrace => false
     - :bulk_threshold => 1000
     - "gem" => "--user-install --bindir /builddir/bin"
  - REMOTE SOURCES:
     - https://rubygems.org/
  - SHELL PATH:
     - /usr/local/bin
     - /usr/bin
     - /usr/local/sbin
     - /usr/sbin
     - /builddir/.local/bin
     - /builddir/bin
```

Install `rdoc`.

```
[mockbuild]$ gem install rdoc
Fetching: rdoc-6.0.4.gem (100%)
Successfully installed rdoc-6.0.4
Parsing documentation for rdoc-6.0.4
Installing ri documentation for rdoc-6.0.4
Done installing documentation for rdoc after 4 seconds
1 gem installed
```

```
[mockbuild]$ gem install rails -v 5.2.0
...
41 gems installed
```

```
[mockbuild]$ gem which rails
/builddir/.gem/ruby/gems/railties-5.2.0/lib/rails.rb
```

```
[mockbuild]$ export GEM_HOME="$(ruby -e 'print Gem.user_dir')"
[mockbuild]$ rails new app
[mockbuild]$ cd app
[mockbuild]$ rails s
...
* Listening on tcp://0.0.0.0:3000
Use Ctrl-C to stop
```

Access to http://0.0.0.0:3000/
=> ok


### Test with root user

```
<mock-chroot># gem env
RubyGems Environment:
  - RUBYGEMS VERSION: 2.7.6
  - RUBY VERSION: 2.5.1 (2018-03-29 patchlevel 57) [x86_64-linux]
  - INSTALLATION DIRECTORY: /usr/share/gems
  - USER INSTALLATION DIRECTORY: /builddir/.gem/ruby
  - RUBY EXECUTABLE: /usr/bin/ruby
  - EXECUTABLE DIRECTORY: /usr/bin
  - SPEC CACHE DIRECTORY: /builddir/.gem/specs
  - SYSTEM CONFIGURATION DIRECTORY: /etc
  - RUBYGEMS PLATFORMS:
    - ruby
    - x86_64-linux
  - GEM PATHS:
     - /usr/share/gems
     - /builddir/.gem/ruby
     - /usr/local/share/gems
  - GEM CONFIGURATION:
     - :update_sources => true
     - :verbose => true
     - :backtrace => false
     - :bulk_threshold => 1000
     - "gem" => "--install-dir=/usr/local/share/gems --bindir /usr/local/bin"
  - REMOTE SOURCES:
     - https://rubygems.org/
  - SHELL PATH:
     - /usr/local/sbin
     - /usr/bin
     - /bin
     - /usr/sbin
     - /sbin
```

```
<mock-chroot># gem install rdoc
Fetching: rdoc-6.0.4.gem (100%)
Successfully installed rdoc-6.0.4
1 gem installed
```

```
<mock-chroot># gem install rails -v 5.2.0
...
Successfully installed rails-5.2.0
41 gems installed
```

```
<mock-chroot># gem which rails
/usr/local/share/gems/gems/railties-5.2.0/lib/rails.rb

<mock-chroot># /usr/local/bin/rails -v
Rails 5.2.0
```

```
<mock-chroot># /usr/local/bin/rails new app
<mock-chroot># cd app
<mock-chroot># /usr/local/bin/rails s
...
* Listening on tcp://0.0.0.0:3000
Use Ctrl-C to stop
```

Access to http://0.0.0.0:3000/
=> ok


## To test only Rails itself

### Pre-condition

```
$ mock --scrub=all
$ mock -i dnf
$ mock shell
```

Then on mock environment.

```
<mock-chroot># dnf install rubygem-rails
<mock-chroot># dnf install ruby-devel sqlite-devel nodejs zlib-devel
```

### Test with normal user

```
<mock-chroot># su - mockbuild
```

Below command is necessary. [2]

```
[mockbuild]$ export GEM_HOME="$(ruby -e 'print Gem.user_dir')"

[mockbuild]$ env | grep GEM_HOME
GEM_HOME=/builddir/.gem/ruby
```

```
[mockbuild]$ rails new app
[mockbuild]$ cd app
[mockbuild]$ rails s
...
* Listening on tcp://0.0.0.0:3000
Use Ctrl-C to stop
```

Access to http://0.0.0.0:3000/
=> ok


### Test with root user

In below command `rails new app`, `bundle install` is run at the created `app` directory.

```
<mock-chroot># rails new app
<mock-chroot># cd app
<mock-chroot># rails s
...
* Listening on tcp://0.0.0.0:3000
Use Ctrl-C to stop
```

Access to http://0.0.0.0:3000/
=> ok


## To test the complete feature including generating a new Rails app using RPM

### Pre-condition

As I tested on mock chroot environment this time, I installed necessary RPM packages like this.

```
$ mock --scrub=all
$ mock -i dnf
$ mock shell
```

Then on mock environment.

```
<mock-chroot># dnf group install 'Ruby on Rails'
```

### Test with normal user

```
<mock-chroot># su - mockbuild
[mockbuild]$ rails new app --skip-bundle
[mockbuild]$ cd app
[mockbuild]$ sed -i '/chromedriver-helper/ s/^/#/' Gemfile
[mockbuild]$ rails s
...
* Listening on tcp://0.0.0.0:3000
Use Ctrl-C to stop
```

Access to http://0.0.0.0:3000/
=> ok


### Test with root user

```
<mock-chroot># rails new app --skip-bundle
<mock-chroot># cd app
<mock-chroot># sed -i '/chromedriver-helper/ s/^/#/' Gemfile
<mock-chroot># rails s
...
* Listening on tcp://0.0.0.0:3000
Use Ctrl-C to stop
```

Access to http://0.0.0.0:3000/
=> ok



- [1] https://fedoraproject.org/wiki/Changes/Ruby_on_Rails_5.2
- [2] https://bugzilla.redhat.com/show_bug.cgi?id=1574594

