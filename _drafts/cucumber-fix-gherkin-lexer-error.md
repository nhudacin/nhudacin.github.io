---
layout: post
title: 'Use Cucumber on Windows - Fix Gherkin Error'
tags:
- Windows
- Gherkin
- Cucumber
---

Hello

I've always had trouble getting [cucumber](https://github.com/cucumber/cucumber) to run on my development machine. Always the same frustrating error...

*** WARNING: cannot load such file --2.1/gherkin_lexer_en

Today, I finally figured out how to get cucumber to run on my machine so I decided to document it for future reference.

## Machine Specs
* Windows Server 2012 x64 (also works on Windows 7 x64)
* Ruby version 2.1 (version included in the ChefDK 0.6.2)
* Cucumber version 2.0
* Chrome Driver version 2.16

## Setup
Setting up a fresh VM Image. I'm using Windows Server 2012 but this always works for Windows 7 (x64).

1. Download and install [ChefDK](http://downloads.chef.io/chef-dk/windows/#/) at the time of this writing, I'm using ChefDK version 0.6.2

2. Open Powershell and initialize the shell for use with chefDK. [Pulled this from the online installation instructions](http://docs.chef.io/install_dk.html)

  ```powershell
  chef shell-init powershell | Invoke-Expression
  ```

3. Install Cucumber and other dependent gems (this may vary a bit as we use [Watir](https://github.com/watir/watir) and [Chrome](http://www.google.com/chrome/))

  ```powershell
  chef gem install cucumber
  chef gem install watir
  chef gem install byebug
  ```

4. Download and install [Google Chrome Browser](http://www.google.com/chrome/) - Again this may vary if you're using FireFox or IE for your cucumber tests.

5. Download [Chrome Driver](https://sites.google.com/a/chromium.org/chromedriver/), Unzip it, and add the path to your environment path

  ```powershell
  $env:Path +=";C:\_Utilities\chromedriver"
  ```

Assuming you already have some cucumber tests ready to go, if not I'll try to update this post with an example, you should be all set to work through the issue.

## Errors Errors and More Errors

Running cucumber with the above setup generates this error:

  ```powershell
  PS C:\cucumber> cucumber SITE="http://www.google.com"
  cucumber : *** WARNING: You must use ANSICON 1.31 or higher (https://github.com/adoxa/ansicon/) to get coloured output on Windows
  At line:1 char:1
  + cucumber SITE="http://www.google.com"
  + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
      + CategoryInfo          : NotSpecified: (*** WARNING: Yo...tput on Windows:String) [], RemoteException
      + FullyQualifiedErrorId : NativeCommandError

  WARNING: cannot load such file -- 2.1/gherkin_lexer_en
  Couldn't load 2.1/gherkin_lexer_en
  The $LOAD_PATH was:
  lib
  Z:/.chefdk/gem/ruby/2.1.0/gems/cucumber-2.0.0/bin/../lib
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/multi_json-1.11.1/lib
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/gherkin-2.12.2-x86-mingw32/lib
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/cucumber-core-1.1.3/lib
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/builder-3.2.2/lib
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/diff-lcs-1.2.5/lib
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/multi_test-0.1.2/lib
  Z:/.chefdk/gem/ruby/2.1.0/gems/cucumber-2.0.0/lib
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/extensions/x86-mingw32/2.1.0/yajl-ruby-1.2.1
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/yajl-ruby-1.2.1/lib
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/rspec-support-3.3.0/lib
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/rspec-core-3.3.0/lib
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/rspec-expectations-3.3.0/lib
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/rspec-mocks-3.3.0/lib
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/rspec-3.3.0/lib
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/rubyzip-1.1.7/lib
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/childprocess-0.5.6/lib
  Z:/.chefdk/gem/ruby/2.1.0/gems/websocket-1.2.2/lib
  Z:/.chefdk/gem/ruby/2.1.0/gems/selenium-webdriver-2.46.2/lib
  Z:/.chefdk/gem/ruby/2.1.0/gems/watir-webdriver-0.7.0/lib
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/ffi-1.9.8-x86-mingw32/lib
  Z:/.chefdk/gem/ruby/2.1.0/gems/columnize-0.9.0/lib
  Z:/.chefdk/gem/ruby/2.1.0/extensions/x86-mingw32/2.1.0/byebug-5.0.0
  Z:/.chefdk/gem/ruby/2.1.0/gems/byebug-5.0.0/lib
  C:/opscode/chefdk/embedded/lib/ruby/site_ruby/2.1.0
  C:/opscode/chefdk/embedded/lib/ruby/site_ruby/2.1.0/i386-msvcrt
  C:/opscode/chefdk/embedded/lib/ruby/site_ruby
  C:/opscode/chefdk/embedded/lib/ruby/vendor_ruby/2.1.0
  C:/opscode/chefdk/embedded/lib/ruby/vendor_ruby/2.1.0/i386-msvcrt
  C:/opscode/chefdk/embedded/lib/ruby/vendor_ruby
  C:/opscode/chefdk/embedded/lib/ruby/2.1.0
  C:/opscode/chefdk/embedded/lib/ruby/2.1.0/i386-mingw32. Reverting to Ruby lexer.
  No lexer was found for en (cannot load such file -- gherkin/lexer/en). Supported languages are listed in gherkin/i18n.json. (Gherkin::I18n::LexerNotFound)
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/gherkin-2.12.2-x86-mingw32/lib/gherkin/i18n.rb:108:in `rescue in lexer'
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/gherkin-2.12.2-x86-mingw32/lib/gherkin/i18n.rb:97:in `lexer'
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/gherkin-2.12.2-x86-mingw32/lib/gherkin/parser/parser.rb:139:in `transition_table'
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/gherkin-2.12.2-x86-mingw32/lib/gherkin/parser/parser.rb:128:in `build_transition_map'
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/gherkin-2.12.2-x86-mingw32/lib/gherkin/parser/parser.rb:124:in `transition_map'
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/gherkin-2.12.2-x86-mingw32/lib/gherkin/parser/parser.rb:91:in `initialize'
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/gherkin-2.12.2-x86-mingw32/lib/gherkin/parser/parser.rb:68:in `new'
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/gherkin-2.12.2-x86-mingw32/lib/gherkin/parser/parser.rb:68:in `push_machine'
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/gherkin-2.12.2-x86-mingw32/lib/gherkin/parser/parser.rb:31:in `parse'
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/cucumber-core-1.1.3/lib/cucumber/core/gherkin/parser.rb:22:in `document'
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/cucumber-core-1.1.3/lib/cucumber/core.rb:27:in `block in parse'
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/cucumber-core-1.1.3/lib/cucumber/core.rb:26:in `each'
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/cucumber-core-1.1.3/lib/cucumber/core.rb:26:in `parse'
  C:/opscode/chefdk/embedded/lib/ruby/gems/2.1.0/gems/cucumber-core-1.1.3/lib/cucumber/core.rb:18:in `compile'
  Z:/.chefdk/gem/ruby/2.1.0/gems/cucumber-2.0.0/lib/cucumber/runtime.rb:70:in `run!'
  Z:/.chefdk/gem/ruby/2.1.0/gems/cucumber-2.0.0/lib/cucumber/cli/main.rb:38:in `execute!'
  Z:/.chefdk/gem/ruby/2.1.0/gems/cucumber-2.0.0/bin/cucumber:9:in `<top (required)>'
  Z:/.chefdk/gem/ruby/2.1.0/bin/cucumber:23:in `load'
  Z:/.chefdk/gem/ruby/2.1.0/bin/cucumber:23:in `<main>'

  ```


Googling around a bit and looking at several issue lists on GitHub, I finally found [this](https://github.com/cucumber/gherkin/issues/273) which is the exact issue I was hitting. The fix is right there, a post by @mscharly:

  ```
  mscharley commented on Jan 18, 2014
  @aslakhellesoy The following works for me:

  $ ruby --version
  ruby 2.0.0p353 (2013-11-22) [x64-mingw32]
  $ gem install gherkin --platform ruby
  Modify lib/gherkin/c_lexer.rb:7 to read as

  prefix = ''
  ```

Let's try compiling the gherkin gem for the windows platform...

```powershell

```
