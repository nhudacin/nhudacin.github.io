---
layout: post
title:  "The Beginning"
date:   2016-02-02 00:00:00 -0600
categories: chef, pester, test-kitchen
---

# This is the story of how I started to love Ruby.

The Setup

I was working on adding integration tests to one of our main chef cookbooks, the one that installs and manages our SQL Server instances. Being a Windows shop, I used [Pester](http://github.com/pester/Pester/) and the [Kitchen-Pester](https://github.com/test-kitchen/kitchen-pester) plugin.  I had a working set of Pester tests that verified all of the configuration settings on the server. Test-Kitchen passed when I ran the steps individually:

```ruby
Kitchen create
kitchen converge
kitchen verify
```

But when I ran `kitchen test` it produced the following error trying to install and load the Pester module:

```powershell
$$$$$$ import-module : The specified module 'Pester' was not loaded because no valid
$$$$$$ module file was found in any module directory.
```

Interestingly, when `kitchen test` failed,  I could run kitchen-verify immediately after and it would pass like there was no issue at all! The issue was the SQL Server 2012 install which does some very interesting things to the environment. I’ll write more about this in the future.

So why are we talking about this? Oh yea, the moment I started loving Ruby. What I needed was to restart the winrm connection after Chef converges but before the kitchen-pester verifier ran. I needed to tweak some code…  Ruby to the rescue!

# Modifying Kitchen-Pester Module

When in doubt, Google. Not sure where to start, I began frantically looking for answers. This is fairly specific stuff. I’m testing Chef code which runs on a remote node thru a winrm connection to a Windows system using a testing framework with a custom verifier. Bet that wasn’t fun to read. The stuff coming back from Google just made my head twist even more. I wasn’t going to find the answer anyway - usually my attitude once I get to page 3 of the search results.

I started implementing my favorite technique in Chef.. Adding log messages to the code. That’s the beauty of Ruby (hit you quick didn’t I?)!!! The ability to modify code on-the-fly without recompiling, moving files, or starting programs. It was the first time I had to really, and I mean really get down n’ dirty with Ruby code, and I loved it! In every method that I wanted to check a variable or a return value, I simply added:

```ruby
warn(&quot;The value of @instance.transport is #{intance.trasport.inspect}&quot;)

# or in the case of not having the warn object

Chef::Log.warn(&quot;The value of @instance.transport is #{intance.trasport.inspect}&quot;)
```

I use the "warn" level a lot when doing this so I can trap where I’m at in the end-to-end process without straining over all the info messages. When I troubleshoot like this, I still set the log level to info so that when a warn message comes across, I can get contextual stuff from the `info` entries. It also helps me cleanup before I get ready to submit a pull request or issue.

```ruby
# Found my kitchen-pester gem in a random folder -
# C:\Users\nhudacin\AppData\Local\chefdk\gem\ruby\2.1.0\gems\kitchen-pester-0.3.0
# Modified it right in-line:

def restart_winrm(state)
  warn('[Nick-Custom] Hello from inside a method!!')
  sleep = state[:connection_retry_sleep]
  sleep.nil? ? 2 : sleep + 1
  instance.transport.connection(state.merge(connection_retry_sleep: sleep))
end

# PS C:\_SourceControl\HLGIT\cookbook-sqlserver&gt; kitchen verify dev-windows -l info
# Output
-----&gt; Verifying &lt;dev-windows&gt;...
       Preparing files for transfer
       Preparing to copy files from C:/_SourceControl/HLGIT/cookbook-sqlserver/test/integration/dev to the SUT.
$$$$$$ [Nick-Custom] Hello from inside a method!!
       Transferring files to &lt;dev-windows&gt;

       Describing sqlserver instance spec
        [+] adds and configures the Maintenance solution 45ms [+] adds sp_AskBrent 14ms
        [+] adds sp_Blitz 14ms
        [+] adds sp_BlitzCache 14ms
        [+] adds sp_BlitzIndex 14ms
        [+] adds sp_BlitzTrace 14ms
        [+] adds sp_foreachdb 14ms
        [+] adds sp_help_revlogin 14ms
        [+] adds sp_WhoIsActive 22ms [+] configures the server in integrated login mode 26ms
       Tests completed in 7.72sPassed: 28 Failed: 0 Skipped: 0 Pending: 0 Inconclusive: 0
       Finished verifying &lt;dev-windows&gt; (0m23.60s).
-----&gt; Kitchen is finished. (0m33.98s)
```

So there it is. Ruby is awesome. Talk to me 6 months ago when I first picked up Chef, I would have told you quite a different story.