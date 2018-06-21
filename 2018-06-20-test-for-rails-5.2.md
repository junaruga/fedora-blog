# Testing Ruby on Rails 5.2.0 on Fedora rawhide

This article is what I tested for Rails 5.2.0 on Fedora rawhide.

I referred this document [1] "How To Test" section for the testing use cases.

There are 3 use cases to test.

* To test Rails from upstream
  * with normal user: [ERROR]
  * with root user: [ERROR]
* To test only Rails itself
  * with normal user: [OK]
  * with root user: [OK]
* To test the complete feature including generating a new Rails app using RPM
  * with normal user: [OK]
  * with root user: [OK]


## To test Rails from upstream

### Pre-condition

```
$ mock -r fedora-rawhide-x86_64 --scrub=all
$ mock -r fedora-rawhide-x86_64 -i ruby-devel sqlite-devel nodejs zlib-devel
```

If we would test it on normal environment, that could be like this.

```
# dnf install ruby-devel sqlite-devel nodejs zlib-devel
```

### Test with normal user

```
<mock-chroot># su - mockbuild
```

Install `rdoc`. It seems that the error message is no problem.

```
[mockbuild]$ gem install rdoc
Fetching: rdoc-6.0.4.gem (100%)
Successfully installed rdoc-6.0.4
ERROR:  While executing gem ... (NoMethodError)
    undefined method `reset' for RDoc::TopLevel:Class
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
	  1: from /builddir/.gem/ruby/gems/activesupport-5.2.0/lib/active_support/dependencies.rb:283:in `block in require'
/builddir/.gem/ruby/gems/bootsnap-1.3.0/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:32:in `require': cannot load such file -- bigdecimal (LoadError)
```
=> error.

```
[mockbuild]$ gem install bigdecimal

[mockbuild]$ gem which bigdecimal
/builddir/.gem/ruby/extensions/x86_64-linux/2.5.0/bigdecimal-1.3.5/bigdecimal.so

[mockbuild]$ rails s
...
	  1: from /builddir/.gem/ruby/gems/activesupport-5.2.0/lib/active_support/dependencies.rb:283:in `block in require'
/builddir/.gem/ruby/gems/bootsnap-1.3.0/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:32:in `require': cannot load such file -- bigdecimal (LoadError)
```

=> error.


### Test with root user

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
	  1: from /usr/local/share/gems/gems/activesupport-5.2.0/lib/active_support/dependencies.rb:283:in `block in require'
/usr/share/gems/gems/bootsnap-1.3.0/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:32:in `require': cannot load such file -- bigdecimal (LoadError)
```

=> error.

```
<mock-chroot># gem install bigdecimal

<mock-chroot># gem which bigdecimal
/usr/local/lib64/gems/ruby/bigdecimal-1.3.5/bigdecimal.so

<mock-chroot># /usr/local/bin/rails s
...
	  1: from /usr/local/share/gems/gems/activesupport-5.2.0/lib/active_support/dependencies.rb:283:in `block in require'
/usr/share/gems/gems/bootsnap-1.3.0/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:32:in `require': cannot load such file -- bigdecimal (LoadError)
```

=> error.


## To test only Rails itself

### Pre-condition

```
$ mock -r fedora-rawhide-x86_64 --scrub=all
$ mock -r fedora-rawhide-x86_64 -i rubygem-rails
```

Install necessary RPM packages to build dependency packages in following process `bundle install` of `rails new app`.

```
$ mock -r fedora-rawhide-x86_64 -i ruby-devel sqlite-devel nodejs zlib-devel
```

If we would test it on normal environment, that could be like this.

```
# dnf install rubygem-rails
# dnf install ruby-devel sqlite-devel nodejs zlib-devel
```

### Test with normal user

```
<mock-chroot># su - mockbuild
```

Below command is necessary. [2]

```
[mockbuild]$ export GEM_HOME="$(ruby -e 'print Gem.user_dir')"
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
$ mock -r fedora-rawhide-x86_64 --scrub=all
$ mock -r fedora-rawhide-x86_64 --dnf-cmd group install 'Ruby on Rails'
$ mock -r fedora-rawhide-x86_64 -i rubygem-bootsnap
$ mock -r fedora-rawhide-x86_64 -i rubygem-mini_magick
```

If we would test it on normal environment, that could be like this.

```
# dnf group install 'Ruby on Rails'
# dnf install rubygem-bootsnap
# dnf install rubygem-mini_magick
```

As `rubygem-bootsnap` and `rubygem-mini_magick` are needed to add "Ruby on Rails" group, I sent pull-request https://pagure.io/fedora-comps/pull-request/286 .


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

