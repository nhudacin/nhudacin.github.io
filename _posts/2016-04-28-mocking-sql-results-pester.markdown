---
layout: post
title:  "Mocking SQL Results in Pester"
date:   2016-04-28 00:00:00 -0600
categories: powershell, pester
---

A lot of my Powershell code interacts with databases and servers. Sometimes I'm grabbing a connection string, other times I'm check-pointing a long operation. When writing Powershell code that interfaces with databases, there's usually a lot to test for. Well, it's surprisingly simple to mock up SQL results using Pester.

I've written a simple function to begin testing with. It queries the (local) instance for a database name that you pass in.

```powershell
# Import the SqlPs Module (and set the location back to current)
$currentLocation = (Get-Location).Path
Import-Module SQLPS -DisableNameChecking
Set-Location $currentLocation

function Test-DatabaseExists {
param(
  [Parameter(Mandatory=$true,Position=0,ValueFromPipeline=$true)]
    [string]$DatabaseName,

  [Parameter(Position=1)]
    [string]$ServerInstance = '(local)'
  )

  # query to check if database exists
  $query = "SELECT COUNT(*) AS [Result] FROM sys.databases WHERE name = '$DatabaseName'"

  # Invoke-SqlCmd params
  $params = @{
    ServerInstance = $ServerInstance
    Database = 'master'
    OutputSqlErrors = $true
    Query = $query
  }

  try {
    $result = Invoke-Sqlcmd @params

    # check the result and proceed accordingly
    if($result.Result -eq 1) {
      Write-Output "Database $DatabaseName does exist on the server $ServerInstance"
    }
    else {
      Write-Output "Database $DatabaseName does NOT exist on the server $ServerInstance"
    }
  }
  catch {
    Throw "Something bad happened. Error Details: $_"
  }
}

```

After testing this a bit locally to make sure it returns what I want it to, I'll save the above function in a temporary folder as `C:\Temp\post-post\Test-DatabaseExists.ps1`. The function's only purpose is to write different output depending on the return value from Invoke-SqlCmd.

One quick note on the above code, I love the use of splatting because it makes the code look a little cleaner. A lot of times, I'll setup the `$params` hash with the basic settings and leave off the Query parameter so that it can be reused throughout the function...

```powershell
$params = @{
    ServerInstance = $ServerInstance
    Database = 'master'
    OutputSqlErrors = $true
  }

$query1 = "SELECT 1"

Invoke-SqlCmd @params -Query $query1

$query2 = "SELECT 2"

Invoke-SqlCmd @params -Query $query2
```

Pretty cool, right? The "splatting" is just shorthand for mapping your variables and inputs and is invoked with the @params notation (different than the $params declaration).

```powershell
Invoke-SqlCmd -ServerInstance $ServerInstance -Database 'master' -OutputSqlErrors $true -Query $query1
```

is the same thing as "splatting" it...

```powershell
Invoke-SqlCmd @params
```

Anyway, back to testing this function...

In a new script file, located in the same directory as the above function... `C:\Temp\posh-post\Test-DatabaseExists.Tests.ps1` I'm going to add some boilerplate code. It's important to name your unit test files with the .Tests.ps1 ending so that Pester can pick up and run these automatically.

```powershell
# Figure out where the function is being called from
$functionRoot = Split-Path $MyInvocation.MyCommand.Path

# Dot Source the function into the session
. &amp;quot;$functionRoot\Test-DatabaseExists.ps1&amp;quot;

Describe 'Test-DatabaseExists' {
    It 'correctly writes output when the database does exist on the server' {
        'AdventureWorks2012' |
            Test-DatabaseExists |
                Should Be 'Database AdventureWorks2012 does exist on the server (local)'
    }
}

```

The above code sets up the session by dot sourcing the function we want to test. Then it describes a context for all of the tests to occur in, and finally it asserts that something happens using the Should Be Pester assertion. Once I save my changes and run Pester, I'll get a result...

```powershell
Set-Location 'C:\Temp\posh-post'; Invoke-Pester
```

Result:

[posh-ss-first-pester-run](/assets/images/posh-ss-first-pester-run.png)

Now because I have the AdventureWorks2012 database on my  local instance, this test passes for me. Depending on your environment, the results may be different **AND** that is the reason we need to mock the results of `Invoke-SqlCmd`. Most times, you're scripts are going to be performing real operations... logging, deleting, moving, etc and we don't want our unit tests to be actually doing that stuff. Those are integration tests and will be covered in the future. For unit tests we want to setup all of the required contexts (data, variables, etc) right there in the unit test so that we can control and test our logical flows. We want to make sure that our `if` statements are catching where they should and our error logic is handling as we hoped thought it would.

As we see from the screen shot, my `if` statement is functioning and sees a database named AventureWorks2012 attached to my local instance. We've proven that with our assertion that the result of `Test-DatabaseExists` should be _"Database AdventureWorks2012 does exist on the server (local)"_. But how do we test the alternative? When the database does **NOT** exist, I want to make sure my output is _"Database AventureWorks2012 does NOT exist on the server (local)"_. We could remove that database, but then our first test would fail. We could change the name of the database we're looking for to _"ThisShouldNeverExistNotInAMillionYears"_ which most likely doesn't exist... but that causes the same problem, our first test would then fail. Let's instead, mock the return value of `Invoke-SqlCmd` so that we know for sure in both cases - exists or not exists - the code will function accordingly.

To mock the result of `Invoke-SqlCmd` we'll add this to the pester test:

```powershell
Mock Invoke-Sqlcmd {
    return @{
        result=1
    }
}
```

The complete unit test file:

```powershell
# Figure out where the function is being called from
$functionRoot = Split-Path $MyInvocation.MyCommand.Path

# Dot Source the function into the session
. "$functionRoot\Test-DatabaseExists.ps1"

Describe 'Test-DatabaseExists' {
    Mock Invoke-Sqlcmd {
        return @{
            result=1
        }
    }
    It 'correctly writes output when the database does exist on the server' {
        'AdventureWorks2012' |
            Test-DatabaseExists |
                Should Be 'Database AdventureWorks2012 does exist on the server (local)'
    }
}

```

If I save and rerun the command:

```powershell
Set-Location 'C:\Temp\posh-post'; Invoke-Pester
```

I get the same result as before, but this time you will too. Even if you don't have AventureWorks2012 attached. 

[posh-ss-first-pester-run](/assets/images/posh-ss-first-pester-run.png)


Adding to it a little bit, we should now test the `Else` statement in the function. This can be done (one of several ways) by adding another Mock `Invoke-SqlCmd { }` command in a lower scope like the `It` block. We could also add a `Context {} }` block if we needed to. Context is a cool concept, but it's really more useful for complex tests, so I'll add the new mock to the `It` block:

```powershell
it 'correctly writes output when the database does NOT exist on the server' {
    Invoke-Sqlcmd {
        return @{
            result=0
        }
    }

    'AdventureWorks2012' |
            Test-DatabaseExists |
                Should Be 'Database AdventureWorks2012 does NOT exist on the server (local)'
 }

```

The complete unit test file now at this point:

```powershell
# Figure out where the function is being called from
$functionRoot = Split-Path $MyInvocation.MyCommand.Path

# Dot Source the function into the session
. "$functionRoot\Test-DatabaseExists.ps1"

Describe 'Test-DatabaseExists' {
    Mock Invoke-Sqlcmd {
        return @{
            result=1
        }
    }
    It 'correctly writes output when the database does exist on the server' {
        'AdventureWorks2012' |
            Test-DatabaseExists |
                Should Be 'Database AdventureWorks2012 does exist on the server (local)'
    }

    it 'correctly writes output when the database does NOT exist on the server' {
        Mock Invoke-Sqlcmd {
            return @{
                result=0
            }
        }
        'AdventureWorks2012' |
            Test-DatabaseExists |
                Should Be 'Database AdventureWorks2012 does NOT exist on the server (local)'
     }
}

```

So now we've tested all of the logical flows inside the `Try {}` block, what about testing the actual `Catch {}` statement? We'll add another mock for the Invoke-SqlCmd that throws that exception:

```powershell
it 'catches and handles an error accordingly' {
    Mock Invoke-Sqlcmd { Throw 'Access Denied.'}

    { 'AdventureWorks2012' |
        Test-DatabaseExists } |
            Should Throw 'Something bad happened. Error Details: Access Denied.'
}
```

Notice two things. We've created another mocked version of the `Invoke-SqlCmd` that throws an error _"Access Denied"_ and we're wrapping some of the code in curly brackets ( `{ }` ). The curly brackets are required to send the scope down one level and allow Pester to handle/expect the error.

[posh-ss-post-pester-run](/assets/images/posh-ss-post-pester-run.png)

And finally, the complete code:

```powershell

# Figure out where the function is being called from
$functionRoot = Split-Path $MyInvocation.MyCommand.Path

# Dot Source the function into the session
. "$functionRoot\Test-DatabaseExists.ps1"

Describe 'Test-DatabaseExists' {
    Mock Invoke-Sqlcmd {
        return @{
            result=1
        }
    }
    It 'correctly writes output when the database does exist on the server' {
        'AdventureWorks2012' |
            Test-DatabaseExists |
                Should Be 'Database AdventureWorks2012 does exist on the server (local)'
    }

    it 'correctly writes output when the database does NOT exist on the server' {
        Mock Invoke-Sqlcmd {
            return @{
                result=0
            }
        }
        'AdventureWorks2012' |
            Test-DatabaseExists |
                Should Be 'Database AdventureWorks2012 does NOT exist on the server (local)'
     }

    it 'catches and handles an error accordingly' {
        Mock Invoke-Sqlcmd { Throw 'Access Denied.'}

        { 'AdventureWorks2012' |
            Test-DatabaseExists } |
                Should Throw 'Something bad happened. Error Details: Access Denied.'
    }
}
```


###The takeaway..

With the use of a simple hash table, formatted the way your data is actually returned, we can mock the results of `Invoke-SqlCmd`.

```powershell
Mock Invoke-Sqlcmd {
    return @{
        column1="This is column one"
        column2=2
        column3="Another column";
    }
}

```

In later series, I'll cover more in depth mocking of sql results with parameter filtering.

As usual, thanks for reading! Any comments or questions are always welcome!