---
title: "A password strength checker (PowerShell)"
date: 2020-05-18
tags: [programming, powershell]
header:
  image: "/images/banner2.png"
  excerpt: "Programming, PowerShell"
---

What led to writing this script, was the realization that my shell knowledge and experience was somewhat limited to Unix systems. Whenever I needed to quickly navigate through a Windows system through a shell, I noticed that the process was a lot more sluggish than I was used to on different operating systems. The Windows GUI is an exception to this as I've owned and used Windows for the majority of my life, but somehow I never really used the CLI as much as I do on Unix systems. Getting comfortable with PowerShell seemed like the perfect solution for this, so I wrote a script that checks the strength of either an existing or a potential password.

## Create password variables

I start off by checking if the user's device has a functioning connection to the internet. I do this by testing a connection to Google's nameservers over a small period of time. If this checks out, the user is prompted for the string that either represents his/her current password, or a potential (but not current) password. When the user enters the string, his/her input is censored and thus not shown on the screen. This is to help prevent credential theft. The string that the user enters is then manipulated in various ways, to make sure that the script actually knows what was entered and can analyze its strength as a password. Note that these variables are only stored inside this script - the script does not save these variables anywhere outside of its own local scope.

## Check if password is commonly used by others

I then `Invoke-WebRequest` to gather a list of the most commonly used passwords, in this case the top 10000 from 2017. I then *locally* check that list for a match with the string the user entered. Note that this way of string matching is done offline and locally *after* the contents of the online list are retrieved and parsed. This does not tell the website the list is hosted on to search for a particular string.

The part of the script where the URL to the online list of passwords is used, is probably one of the things that I'll tweak/update the most. For example: whenever I find a bigger, equally reliable but more recent list I'll probably replace the old list with that one. I'm also considering letting the script check multiple password lists as opposed to just one, but I'll have to see what exactly that does to the performance of the script.

If a match is found between the user's input and the list, the program informs the user of this and exits. The reasoning behind this was that a common password isn't really worth performing further checks on.

## Check if password already (partly) exists in plaintext on local device

In this part, I first make sure that the user doesn't see any of the errors that potentially occur. I chose to do this because in all of the test runs I've done, there's always atleast one file that can't be accessed by the script due to permissions. In my opinion, the script doesn't need to check every single file, just the ones it has permission to. I make sure the way errors are displayed to the user is reset later on, so if any other (unintended) errors occur, those will show.

What this part of the script does, is check if the string entered by the user already exists in a set of files on the user's device. Depending on the user's device this can take an extremely long time, so I made sure to only check files ending in `.txt`, `.odt`, `.docx` and `.doc`!!!. If a match is found, the script once again informs the user and exits. The reasoning from earlier behind exiting the script applies here as well.

I'd like to note that this particular check not only finds out if passwords can be found extremely easily by searching the user's device, but also if the password is an extremely weak, easy-to-guess one. For example, if my password is "arno", this script will find out. If my password is the city I live in, it will also find out. Anything that's stored in the text files mentioned earlier gets searched, making sure the password string can't easily be guessed because of its relation to the user.

## Check password length

This is where the script checks the length of the string entered by the user. I used `select -expandproperty` for this, which is something I use all the time and is extremely useful. The script then defines a number of criteria and allocates points to the password strength variable based on those criteria.

## Check special characters

In here, I check which and how many special characters the potential password has. I compare that amount to the amount of total characters in the string so I can work with a percentage of special characters. If that percentage between a minimum and a maximum (0.1 and 0.9), the script allocates points to the password strength variable.

In my opinion, checking for relative numbers is better than checking for absolute ones. A strong password should seem completely random, and a password that's 99% normal characters and 1% special characters doesn't seem random to me.

## Check for upper- and lowercase combination

This part checks for the combination of upper- and lowercase characters. It does the same thing the special character check does, and once again bases its point allocation on a percentage rather than an absolute number.

## Check for numbers

As the header says, this is where the script checks for numbers in the string entered by the user. Once again, it more or less does the same thing the previous couple of parts do. This is also the final part of the script, where the end result of the point allocations throughout the script are presented to the user. This is where the user find outs how his/her potential password scores on a scale of 0 to 100. As far as I know clearing the variables created throughout the script isn't necessary, but "for good measure" I manually clear them anyway.

## Final words

Creating this script was a lot of fun and similar to my experience with the first Bash program I posted on this website. PowerShell is definitely something I'll keep coming back to, so I'm pretty sure I'll be posting more PS scripts on this website in the future. As always, my main goal here was to learn so if you have any suggestions, questions or comments in general, please let me know. They're more than welcome.

```powershell
#Create password variables

clear
Write-Host -NoNewLine "Checking if a stable connection to the internet can be made..."

if ((Test-Connection 8.8.8.8 -Quiet) -eq $True)
  {
    Write-Host " Done.`n" -ForegroundColor green
  }
else
  {
    Write-Host " No stable connection could be made. Please check your internet connection and rerun this script later."
    break
  }

Write-Host "Welcome to pwstrength.ps1, a tool that checks the strength of potential passwords."
Write-Host "You will be asked to enter a (potential) password, after which a number of checks will be made to evaluate its strength as a password."
Write-Host "Its strength will then be shown to you, measured as a number out of 100 (the maximum score)."
Write-Host "Whatever you enter next will not be displayed to the screen. This will be censored completely."
Write-Host "When the script exits, it automatically erases your input making sure that no passwords will be saved. `n`n"
Write-Host -NoNewLine "Enter the password that you want to check: "
$pw_secure = Read-Host -AsSecureString
$pw_bstr = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($pw_secure)
$pw_string = [Runtime.InteropServices.Marshal]::PtrToStringAuto($pw_bstr)

$strength = 0

Write-Host "`n"

#Check if password is commonly used by others
Write-Host -NoNewLine "Checking if password is commonly used by others..."

$common = Invoke-WebRequest -UseBasicParsing https://github.com/danielmiessler/SecLists/blob/master/Passwords/darkweb2017-top10000.txt

if (($common.toString() -split "[`r`n]" | Select-String "$pw_string" -SimpleMatch) -ne $Null)
	{
		Write-Host " Password can be found in a list of most common passwords. Do not use!"
		break
	}
else
	{
		$strength+= 10
	}

Write-Host " Done.`n" -ForegroundColor green

#Check if password already (partly) exists in plaintext on local device
$ErrorActionPreference='silentlycontinue'
Write-Host -NoNewLine "Checking if password already (partly) exists in plaintext on local device... This might take a minute or two."

$local = ls c:/ -Recurse -Include *.txt, *.odt, *.docx, *.doc | select-string "$pw_string" -simplematch -verbose:$false
if ($local -ne $null)
	{
		Write-Host "Password already exists locally. Do not use!"
		break
	}
else
	{
		$strength+= 10
	}

Write-Host " Done.`n" -ForegroundColor green
$ErrorActionPreference='continue'

#Check password length
Write-Host -NoNewLine "Checking length..."

$pw_length = ($pw_string | select -expandproperty length)

if ($pw_length -in 10..14)
	{
		$strength+= 10
	}
elseif ($pw_length -in 15..19)
	{
		$strength+= 20
	}
elseif ($pw_length -ge 20)
	{
		$strength+= 35
	}

Write-Host " Done." -ForegroundColor green
Write-Host -NoNewLine "Password strength after checking for character length: "; Write-Host -ForegroundColor Yellow "$strength/100`n"

#Check special characters
Write-Host -NoNewLine "Checking special characters..."

$special_length = 0
$special = " !`"#$%&'()*+,-./:;<=>?@[\]^_``{|}~".toCharArray()
$pw_array = $pw_string.toCharArray()
$min = 0.1
$max = 0.9

$pw_array | ForEach-Object `
{
	if ($special -like $_)
		{
			$special_length+= 1
		}
}

if ($($special_length/$pw_length) -ge $min -and $($special_length/$pw_length) -le $max)
	{
		$strength+= 15
	}

Write-Host " Done." -ForegroundColor green
Write-Host -NoNewLine "Password strength after checking for special characters: "; Write-Host -ForegroundColor Yellow "$strength/100`n"

#Check for upper- and lowercase combination
Write-Host -NoNewLine "Checking uppercase characters..."

$capital_length = 0
$capitals = [char[]](65..90)

$pw_array | ForEach-Object `
{
	if ($capitals -clike $_)
		{
			$capital_length+= 1
		}
}

if ($($capital_length/$pw_length) -ge $min -and $($capital_length/$pw_length) -le $max)
	{
		$strength+= 15
	}

Write-Host " Done." -ForegroundColor green
Write-Host -NoNewLine "Password strength after checking for uppercase characters: "; Write-Host -ForegroundColor Yellow "$strength/100`n"

#Check for numbers
Write-Host -NoNewLine "Checking number characters..."

$numbers_length = 0
$numbers = (0..9)

$pw_array | ForEach-Object `
{
	if ($numbers -like $_)
		{
			$numbers_length+= 1
		}
}

if ($($numbers_length/$pw_length) -ge $min -and $($numbers_length/$pw_length) -le $max)
	{
		$strength+= 15
	}

Write-Host " Done." -ForegroundColor green
Write-Host -NoNewLine "Password strength after checking for numbers: "; Write-Host -ForegroundColor Yellow "$strength/100`n"

#Removing password variables from local scope just in case

rv pw_secure, pw_bstr, pw_string, pw_array, pw_length, local, special_length, capital_length, numbers_length
```
