# Building Reusable Tools

## TOOL-01 Decide whether you're coding a "tool" or a "controller" script

For this discussion, it's important to have some agreed-upon terminology. While the terminology here isn't used universally, the community generally agrees that several types of "script" exist:

1. Some scripts contain tools, which are meant to be reusable. These are typically functions or advanced functions, and they are typically contained in a script module or in a function library of some kind. These tools are designed for a high level of re-use.
2. Some scripts are controllers, meaning they are intended to utilise one or more tools (functions, commands etc.) to automate a specific business process. A script is not intended to be reusable; it is intended to make use of other functions and commands.

For example, you might write a module that interfaces with a product's REST API and includes cmdlets for creating a new user, setting a user's permissions, configuring a user's dashboard etc. This module is reusable and falls into the first category described above.

If a new user is required to be configured on the system, you might write a controller script that utilises the module's cmdlets and adds additional business logic. This script has a specific purpose and falls into the second category above.

## TOOL-02 Make your code modular

Generally, people tend to feel that most working code - that is, your code which does things - should be modularized into functions and ideally stored in script modules which makes any functions easily reusable.

## TOOL-03 Make tools as re-usable as possible

Tools should accept input from parameters and should (in most cases) produce any output to the pipeline; this approach helps maximize reusability.

## TOOL-04 Use PowerShell standard cmdlet naming

Use the verb-noun convention, and use the PowerShell standard verbs.

You can get a list of the verbs by typing `Get-Verb` at the command line.

## TOOL-05 Use PowerShell standard parameter naming

Tools should be consistent with PowerShell native cmdlets in regards to parameter naming.

For example, use `$ComputerName` and `$ServerInstance` rather than something like `$Param_Computer` or `$InstanceName`

## TOOL-06 Tools should output raw data

The community generally agrees that tools should output raw data. That is, their output should be manipulated as little as possible. If a tool retrieves information represented in bytes, it should output bytes, rather than converting that value to another unit of measure. Having a tool output less-manipulated data helps the tool remain reusable in a larger number of situations.

## TOOL-07 Controllers should typically output formatted data

Controllers, on the other hand, may reformat or manipulate data because controllers do not aim to be reusable; they instead aim to do as good a job as possible at a particular task.

For example, a function named `Get-DiskInfo` would return disk sizing information in bytes, because that's the most-granular unit of measurement the operating system offers. A controller that was creating an inventory of free disk space might translate that into gigabytes, because that unit of measurement is the most convenient for the people who will view the inventory report.

An intermediate step is useful for tools that are packaged in script modules: views. By building a manifest for the module, you can have the module also include a custom .format.ps1xml view definition file. The view can specify manipulated data values, such as the default view used by PowerShell to display the output of `Get-Process`. The view does not manipulate the underlying data, leaving the raw data available for any purpose.

## WAST-01 Don't re-invent the wheel

There are a number of approaches in PowerShell that will "get the job done". In some cases, other community members may have already written the code to achieve your objectives. If that code meets your needs, then you might save yourself some time by using it, instead of writing it yourself.

For example:

```PowerShell
# Bad
function Ping-Computer ($computerName) {
    $ping = Get-WmiObject Win32_PingStatus -filter "Address='$computerName'"
    if ($ping.StatusCode -eq 0) {
        return $true
    } else {
        return $false
    }
}
```

This function has a few opportunities for enhancement. First of all, the parameter block is declared as a basic function; using advanced functions is generally preferred. Secondly, the command verb ("Ping") isn't an approved PowerShell command verb. This will cause warnings if the function is exported as part of a PowerShell module, unless they are otherwise suppressed on import.

Thirdly, there's little reason to write this function in PowerShell v2 or later. PowerShell v2 has a built-in command that will allow you to perform a similar function. Simply use:

```PowerShell
Test-Connection $computerName -Quiet
```

This built-in command accomplishes the exact same task with less work on your part; you don't have to write your own function.

It has been argued by some that, "I didn't know such-and-such existed, so I wrote my own". That argument typically fails with the community, which tends to feel that ignorance is no excuse. Before making the effort to write some function or other unit of code, find out if the shell can already do that. Ask around. And, if you end up writing your own, be open to someone pointing out a built-in way to accomplish it.

On the flip side, it's important to note that writing your own code from the ground up can be useful if you are trying to learn a particular concept, or if you have specific needs that are not offered by another existing solution.

## WAST-02 Report bugs to Microsoft

An exception: if you know that a built-in technique doesn't work properly (e.g. it is buggy or doesn't accomplish the exact task), then obviously it's fine to re-invent the wheel. However, in cases where you're doing so to avoid a bug or design flaw, then you should - as an upstanding member of the community - report the bug on [https://github.com/PowerShell/PowerShell/issues](https://github.com/PowerShell/PowerShell/issues).

TODO: The "PURE" section is dubious at best. We need to discuss it.

## PURE-01 Use native PowerShell where possible

This means not using COM, .NET Framework classes, and so on when there is a native Windows PowerShell command or technique that gets the job done.

## PURE-03 Document why you haven't used PowerShell

So when is it okay to move from one item on this list to another? Obviously, if a task can't be accomplished with a more-preferred way, you move on to a less-preferred way.

If a less-preferred approach offers far superior performance, and performance is a potential issue, then go for the better-performing approach. For example, Robocopy is superior in nearly every way to Copy-Item, and there are probably numerous circumstances where Robocopy would do a much better job.

Document the reason for using tools other than PowerShell in your comments.

## PURE-04 Wrap other tools in an advanced function or cmdlet

That said, you truly become a better PowerShell person if you take the time to wrap a less-preferred way in an advanced function or cmdlet. Then you get the best of both worlds: the ability to reach outside the shell itself for functionality, while keeping the advantages of native commands.

Ignorance, however, is no excuse. If you've written some big wrapper function around Ping.exe simply because you were unaware of Test-Connection, then you've wasted a lot of time, and that is not commendable. Before you move on to a less-preferred approach, make sure the shell doesn't already have a way to do what you're after.
