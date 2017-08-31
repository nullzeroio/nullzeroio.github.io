---
title: 'WorkLog PowerShell Module'
date: 2015-02-06T15:50:06+00:00
author: Kevin Kirkpatrick
layout: post
categories:
  - PowerShell
tags:
  - DevOps
  - git
  - GitHub
  - Module
  - PowerShell
  - PS
  - WorkLog
---
As part of a new initiative, of sorts, I wanted a way to record daily accomplishments, which is something that I have thought about doing for quite some time, but never got enough motivation to actually do anything about it. That said, I decided to revisit, take action and come up with some requirements on what a workable solution would look like (for me):

  * It needs to be easy to record entries (if it's hard or cumbersome, it won't have sustainability)
  * It needs to fit into my daily workflow
  * The format needs to be somewhat open/easy to move between different platforms (Mainly Windows & Mac OS)
  * If possible, pick a solution that can sharpen a skill-set, in the process

Given each of those, my final solution ended up being quite simple: GitHub and GitHub Flavored Markdown (GFM) files. This was ideal, because it really meets all of the requirements, above.

  * It needs to be easy to record entries
      * Use Sublime Text to create/edit GFM files for each day.
  * It needs to fit into my daily workflow
      * I already use GitHub on a daily basis for source control of various scripts, projects, modules, etc.
  * The format needs to be somewhat open/easy to move between different platforms
      * Again, I already use GitHub as a central source control for all of my code repositories and have that synced between my Windows and Mac systems
  * If possible, pick a solution that can sharpen a skill-set, in the process
      * While familiar with GitHub and GFM files, I can always use some extra practice

Now, after I started building out how I wanted things setup and started creating entries into a log file, I realized that flipping over to a text editor to make entries was working fine, but I wanted deeper integration; why not use PowerShell!

What resulted was a PowerShell module comprised of 3 functions:

  * New-WorkLog
  * Add-WorkLog
  * Get-WorkLog

The main functionality I needed out of this module was:

  * Create a new Work Log file for each day
  * Standard file name format
  * Generate pre-formatted file layout (Title, fully formatted date and an initial bullet point)
  * Ability to add entries right from the PowerShell console
  * Ability to view the contents of the Work Log for the current day, from within the console
  * Basic support for bullet point indentation
  * Basic fail-safes so that files wouldn't get overwritten.

Before I continue, you can get more information as well as download, clone or fork the Module from <a href="https://github.com/vScripter/WorkLogModule" target="_blank">GitHub</a>

Lets take a closer look at each function.

* * *

## New-Worklog

### Function header

```
function New-WorkLog {

  [cmdletbinding()]
  param (
	[parameter(Mandatory = $false)]
	[System.String]$Path = "$ENV:USERPROFILE\Documents\GitHub\WorkLog"
  )
```

Pretty standard material getting things started. The main item worth noting is that I hard coded the working directory and assigned it to the -Path parameter.

### BEGIN Block
```
BEGIN {

  $now        = Get-Date
  $dateFormat = $now.tostring('yyyyMMdd')
  $dateDay    = $now.tostring('dddd')
  $fileName   = $dateFormat + '_' + $dateDay + '_' + 'WL.md'
  $filePath   = Join-Path $Path $fileName
  $nowLong    = $now.tostring('D')

} # end BEGIN block
```

- I need to get some date details and convert them to strings so that I can create a custom, standard file name as well as the date detail that will get stored in the actual file.
  - `$now` - Get and assign the current date information in the $now variable, so that we can use to construct our custom file format.
  - `$dateFormat` - Call the .tostring() method on the $now variable and format the output so it will look like '20150205'
  - `$dateDay` - Call the .tostring() method but specify 'dddd' in order to get the day of the week in long format, like 'Thursday'
  - `$fileName` - Build out the actual file name string. I'm adding the value stored in $dateFormat; then add an underscore '\_'; then add the value stored in $dateDay; then another underscore '\_'; finally, add the last piece 'WL.md' ('WL' just stands for Work Log)
  - `$filePath` - Join the value stored in `$Path` with the value stored in `$fileName`, and the end result is the full path of the daily Work log file, which looks something like: `C:\Users\%USERNAME%\Documents\GitHub\WorkLog\20150205_Thursday_WL.md`



### PROCESS Block

```
PROCESS {

		if (-not (Test-Path -LiteralPath $filePath -PathType Leaf)) {

			try {

				Write-Verbose -Message 'Creating worklog file'
				New-Item -Path $filePath -Type File -ErrorAction 'Stop' | Out-Null

				Write-Verbose -Message 'Adding message to Work Log'
				Write-Output -InputObject '## Work Log ' | Out-File -LiteralPath $filePath -Append
				Write-Output -InputObject "### $nowLong" | Out-File -LiteralPath $filePath -Append
				Write-Output -InputObject ' ' | Out-File -LiteralPath $filePath -Append
				Write-Output -InputObject '* ' | Out-File -LiteralPath $filePath -Append

				if (Test-Path -LiteralPath $filePath -PathType Leaf) {

					Write-Verbose -Message 'Work Log file created successfully'

				} else {

					Write-Verbose -Message 'Work Log file not created'

				} # end if/else Test-Path

			} catch {

				Write-Warning -Message "Error creating work log file. $_"

			} # end try/catch
		} else {

			Write-Warning -Message 'Worklog for today had already been created'

		} # end if/else

	} # end PROCESS block
  ```

*  First, we create a logical statement that tests for the existence of the log file and if it does not exist, we want to create it, but if it does exist, we want to display a message to the console saying that it already exists. Since I only want to take action if it DOES NOT exist, I start by using that criteria as the first validation via '-not' operator. `if (-not (Test-Path -LiteralPath $filePath -PathType Leaf)`

* I then create the desired file and use Write-Output to add the desired detail to the file. As a general comment, the spacing of the text does matter, to a certain degree, when creating these markdown files, since spaces and special characters actually mean something when the markdown files are read/displayed.
* If the file does exist, use Write-Warning to display some text in the console letting me know that one is already created. I used Write-Warning here because I wanted to avoid clouding up the 'Output/Success' data stream ([great article](http://blogs.technet.com/b/heyscriptingguy/archive/2014/03/30/understanding-streams-redirection-and-write-host-in-powershell.aspx) by June Blender on PowerShell streams). As a best practice, I ALWAYS try to avoid sending informational messages that are only useful to the user running the cmdlet/script/function, from going down the pipeline; Output from the 'Write-Output' cmdlet uses the 'Output/Success' stream (stream 1). Write-Host is forbidden and kills kittens, so it should never be used 🙂 (It also sends nothing to any stream, which can very problematic).
* We don't do anything in the END {} block so we can move on to the next function

* * *


## Add-Workload

### Function Header
```
function Add-WorkLog {

    [cmdletbinding()]
    param (
        [parameter(Mandatory = $true,
                   Position = 0)]
        [System.String]$Message,

        [parameter(Mandatory = $false,
                   Position = 1)]
        [System.String]$Path = "$ENV:USERPROFILE\Documents\GitHub\WorkLog",

        [parameter(Mandatory = $false,
                   Position = 2)]
        [ValidateSet('1', '2', '3', '4')]
        [System.String]$Indent
    )
```

* I needed some parameters to deal with the actual message/update to be entered into the Work Log, as well as an -Indent parameter to specify the level of indentation
    * `-Message` - This is the string data that will actually be appended into the Work Log file. I set the position at '0' so that I can quickly add entries without having to specify '-Message' before typing the message.
    * `-Indent` - This is the level of desired indentation and is only required if you want to indent the entry. I only want to offer 4 levels of indentation, so I specify a [ValidateSet()] attribute to the parameter with the only values I want to accept (1,2,3,4). These values get passed to a function that actually reads the value and performs the proper indentation spacing when it appends the entry to the Work Log file. More on that in the PROCESS block

### BEGIN Block
```
BEGIN {

		$now        = Get-Date
		$dateFormat = $now.tostring('yyyyMMdd')
		$dateDay    = $now.tostring('dddd')
		$fileName   = $dateFormat + '_' + $dateDay + '_' + 'WL.md'
		$filePath   = Join-Path $Path $fileName
		$nowLong    = $now.tostring('D')

		function Add-Indent {
			[cmdletbinding()]
			param (
				[parameter(Mandatory = $true)]
				[System.String]$Level
			)

			switch ($Level) {
				'1' { '  * ' }
				'2' { '    * ' }
				'3' { '      * ' }
				'4' { '        * ' }
			} # end switch

		} # end function indent

	} # end BEGIN block
```

* I won't go over the date variables since we reviewed that in the 'New-WorkLog' function
* I created the 'Add-Indent' function to handle the indentation spacing that results in a properly formatted, indented, markdown file.
    * I accept the -Indent parameter, if it was supplied, in the -Level parameter of this function
    * I use a basic 'switch' statement to define the spacing that gets outputted when the function is called. More on this functionality in the PROCESS block

### PROCESS Block
```
PROCESS {

  if (Test-Path -LiteralPath $filePath) {

      if ($Indent) {

          $indentMessage = $(Add-Indent -Level $Indent) + $Message
          Write-Verbose -Message 'Adding message to Work Log'
          Write-Output -InputObject $indentMessage | Out-File $filePath -Append

      } else {

          Write-Verbose -Message 'Adding message to Work Log'
          Write-Output -InputObject "* $Message" | Out-File $filePath -Append

      } # end if/else $Indent

  } elseif (-not (Test-Path -LiteralPath $filePath)) {

      Write-Warning -Message 'Work log not created'
      try {

          Write-Verbose -Message 'Creating worklog file'
          New-Item -Path $filePath -Type File -ErrorAction 'Stop' | Out-Null

          if ($Indent) {

              Write-Warning -Message "Since the file has not be created, your '-Indent' input of '$Indent' will be ignored for the first log entry."

          } # end if $Indent

          Write-Verbose -Message 'Adding message to Work Log'
          Write-Output -InputObject '## Work Log ' | Out-File -LiteralPath $filePath -Append
          Write-Output -InputObject "### $nowLong" | Out-File -LiteralPath $filePath -Append
          Write-Output -InputObject ' ' | Out-File -LiteralPath $filePath -Append
          Write-Output -InputObject "* $Message" | Out-File -LiteralPath $filePath -Append

          if (Test-Path -LiteralPath $filePath -PathType Leaf) {

              Write-Verbose -Message 'Work Log file created successfully'

          } else {

              Write-Verbose -Message 'Work Log file not created'

          } # end if/else

      } catch {

          Write-Warning -Message "Error creating work log file. $_"

      } # end try/catch

  } # end if/elseif

} # end PROCESS block
```

* I start with a conditional statement to check and see if the WorkLog file exists and if it does, then check to see if there was a desired indent level.
* `$indentMessage` - Stores the properly indented message that will be written to the Work Log file. It merges the desired indent with the provided -Message string
* If no indent is desired, it simply appends the message to the current Work Log file
* If the Work Log file DOES NOT exist, we want to go ahead and create it. The main differences the way the file gets created with 'Add-WorkLog' vs. 'New-WorkLog' are:
    * There is no asterisk (bullet point) created as part of the initial file create; if the file doesn't exist, we want our first entry to be the first bullet point.
    * Along the same lines, if we supplied an indent level and the file doesn't exist, we don't want to to indent the first entry, so, I just ignore indents if the file has to be created using this function
* There is nothing in the END {} block so we will move on to the next function

* * *

## Get-Worklog

### Function Header
```
function Get-WorkLog {

	[cmdletbinding()]
	param (
		[parameter(Mandatory = $false)]
		[System.String]$Path = "$ENV:USERPROFILE\Documents\GitHub\WorkLog"
	)
```

This isn't any different than the start of the New-WorkLog function, so you can reference that if you have any questions


### BEGIN Block
This is also no different than the New-WorkLog function; you can reference more detail above

### PROCESS Block
```
PROCESS {

  if (Test-Path -LiteralPath $filePath -PathType Leaf) {

    Get-Content -LiteralPath $filePath -ReadCount 0

  } else {

    Write-Warning -Message 'Work Log file has not been created'

  } # end if/else

} # end PROCESS block
```

* This is the most straight forward function, of all the rest. We are just using the Get-Content cmdlet to read in and display the contents of the current work log file.
* I use the '-ReadCount 0' parameter so that is reads the entire file contents in at one time, instead of line-by-line.
* If the Work Log file has not been created, display a message to the console
* There is no END {} block


* * *

## Output

Below are some example commands and the file/file format that they produce

```
Add-WorkLog 'This is the first item of the day'
Add-WorkLog 'This is the first sub-item of the day' -Indent 1
Add-WorkLog 'This is the first sub-sub-item of the day' -Indent 2
Add-WorkLog 'This is the second item of the day'
Add-WorkLog "This is how using back-tick marks through the PowerShell console can be used to ````show code```` in the GFM"
```

#### Output file name

`20150206_Friday_WL.md`

#### Output file content

```
## Work Log
### Friday, February 06, 2015

* This is the first item of the day
  * This is the first sub-item of the day
    * This is the first sub-sub-item of the day
* This is the second item of the day
* This is what using back-tick marks through the PowerShell console can be used to ``show code`` in the GFM
```

#### End result