---
layout: post
title:  "Powershell - Break out of Loops!"
date:   2016-02-02 00:00:00 -0600
categories: powershell
---


## Going back to the basics...

Sometimes it’s good to pay homage to the simple things in programming. Today, we’re gonna do a lite surface scratch on `while($true)` loops and the `break` statement. This came about as I was adding some retry logic to one of my functions. I wrote several lines of very ugly code including several nested `try { } catch { }` blocks. Ugh. It was then I stepped back and thought, this is a perfect opportunity for a `while($true)` loop! These little guys are awesome but I always hesitate a bit thinking about putting my function into a never-ending loop. But with the help of a few well placed `break` statements, it’s the cleanest way to check the result of something and retry if it’s not what you expected.

**Let’s take a look...**

Just a simple while loop. By setting a variable `$var` before the `while($true)` loop and incrementing it inside the loop, we can count the loop iterations. With a simple `break` statement, we can break out of the never-ending while loop.

```powershell
# Simple while($true) loop
$var = 0
while($true) {

  # increment the loop count
  $var++

  # sleep to slow down the output
  Start-Sleep -Seconds 2

  Write-Output "`tThis is loop #$var, Hello!"

  # break out after 5 runs
  if ($var -ge 5) { break }
}

Write-Output "You are outside the loop now"

```

Output

```
This is loop #1, Hello!
This is loop #2, Hello!
This is loop #3, Hello!
This is loop #4, Hello!
This is loop #5, Hello!
You are outside the loop now
```

That's not very much fun but it does show how clean and simple these `while($true)` loops are to use. The next example shows how we poll a variable (or service, or process, etc) and break out of the loop before it terminates at the fifth iteration. This is especially usefule if you use a `start-sleep` to give your external resource any amount of time to complete.

```powershell
$var = 0
$api_response = @{content=@{name='Frank';description='bearded';};}
$return_value = $null

while($true) {

  # increment the loop count
  $var++

  Write-Output "This is loop #$var, Hello!"

  # on the second loop, let's assign the $return_value variable to
  # the $api_response value. This will simulate an unexpected result
  # during the first loop.
  if ($var -eq 2 ) {
    $return_value = $api_response
  }

  # Here we'll check for 'Frank' in the response's name
  if ($return_value.content.name -eq 'Frank') {

    # if found, don't continue with the loop
    break
  }

  # break out after 5 runs
  if ($var -ge 5) { break }

  # sleep to slow down the output
  Start-Sleep -Seconds 1
}

Write-Output "You are outside the loop now"
```

Output

```
This is loop #1, Hello!
This is loop #2, Hello!
You are outside the loop now
```

This loop only runs twice before the `$return_value` is set to return the value `Frank`. Pretty neat, right? What if `Frank` is never returned?Then the loop continues until the next `break` is hit.

### All breaks are equal...

No matter how many nested `if { } else { }` you have, the first break exits the loop. Unless you’re using a nested `while($true)` statement, just say `break`!!

```powershell
$var = 0
$inspect_this = @('A','B','C')

while($true) {

  # increment the loop count
  $var++

  Write-Output "This is loop #$var, Hello!"

  # we'll create another loop by looping over the $inspect_this array
  $inspect_this |
    %{
      Write-Output "--> Inner array value $_"

      # on the second iteration if the while($true) loop, do some
      # further digging
      if ($var -ge 2) {

        # if on the second iteration we find a 'B' value and it pisses
        # us off, we can break out of the while($true) loop two nested
        # levels above
        if ($_ -eq 'B') {
          Write-Output "`nATTENTION!!! We found the 'B' value. Notifying Houston. There is a problem"
          break
        }
      }
    }

  # break out after 5 runs
  if ($var -ge 5) { break }

  # sleep to slow down the output
  Start-Sleep -Seconds 1
}

Write-Output "You are outside the loop now"
```

Output

```
This is loop #1, Hello!
--> Inner array value A
--> Inner array value B
--> Inner array value C
This is loop #2, Hello!
--> Inner array value A
--> Inner array value B

ATTENTION!!! We found the 'B' value. Notifying Houston. There is a problem
You are outside the loop now
```

Pretty neat stuff huh? Leave me a comment if you have any questions. Thanks for reading!
