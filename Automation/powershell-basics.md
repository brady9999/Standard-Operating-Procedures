# PowerShell Basics
> A reference guide to PowerShell scripting for IT automation and system administration.

**Category:** Automation  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **PowerShell** | PowerShell | Microsoft's task automation and configuration management framework |
| **Cmdlet** | Cmdlet | A PowerShell command in Verb-Noun format e.g. `Get-Process` |
| **Pipeline** | Pipeline | Passing output of one cmdlet as input to another using `\|` |
| **Object** | Object | PowerShell returns structured objects, not plain text |
| **Property** | Property | An attribute of an object e.g. Name, Status, Size |
| **Method** | Method | An action an object can perform e.g. `.Start()`, `.Stop()` |
| **Variable** | Variable | A named storage location starting with `$` e.g. `$name` |
| **Array** | Array | A collection of values e.g. `@("a", "b", "c")` |
| **Hashtable** | Hashtable | A key-value collection e.g. `@{Name="Brady"; Age=19}` |
| **Function** | Function | A reusable block of code |
| **Module** | Module | A package of related cmdlets and functions |
| **Script** | Script | A `.ps1` file containing PowerShell commands |
| **ISE** | Integrated Scripting Environment | The built-in PowerShell editor (older) |
| **VS Code** | Visual Studio Code | Recommended editor for PowerShell scripts |
| **ExecutionPolicy** | Execution Policy | Controls which scripts are allowed to run |
| **WMI** | Windows Management Instrumentation | A Windows framework for accessing system information |
| **CIM** | Common Information Model | The modern replacement for WMI |

---

## Overview
PowerShell is the primary automation tool for Windows environments. Unlike traditional shells that return text, PowerShell returns objects — making it far more powerful for automation.

**PowerShell versions:**
- **Windows PowerShell 5.1** — built into Windows, Windows only
- **PowerShell 7+** — cross-platform (Windows, Linux, macOS), actively developed

**Open PowerShell:**
- Start menu → search "PowerShell"
- Right-click Start → Windows PowerShell (Admin)
- `Win + X` → Windows PowerShell

---

## 1. Basic Syntax and Commands

```powershell
# Getting help
Get-Help Get-Process             # Help for a specific cmdlet
Get-Help Get-Process -Examples   # Show examples
Get-Help Get-Process -Full       # Full documentation
Update-Help                      # Update help files

# Finding commands
Get-Command                      # List all commands
Get-Command -Noun Process        # Find commands about processes
Get-Command -Verb Get            # Find all Get-* commands
Get-Command *service*            # Find commands with "service" in the name

# Basic output
Write-Host "Hello World"         # Print to screen
Write-Output "Hello World"       # Send to pipeline (preferred)
Write-Verbose "Debug message"    # Verbose output
Write-Error "Something failed"   # Error output
Write-Warning "Be careful"       # Warning output

# Clear screen
Clear-Host
cls

# Exit PowerShell
Exit
```

---

## 2. Variables

```powershell
# Assign a variable
$name = "Brady"
$age = 19
$isAdmin = $true
$pi = 3.14159

# Display a variable
$name
Write-Host $name
Write-Host "My name is $name"        # String interpolation
Write-Host "My name is ${name}Genik" # Variable in string

# Variable types
[string]$name = "Brady"
[int]$age = 19
[bool]$active = $true
[double]$price = 19.99
[datetime]$today = Get-Date

# Special variables
$null         # Represents nothing/empty
$true         # Boolean true
$false        # Boolean false
$_            # Current object in pipeline
$Error        # Array of recent errors
$PSVersionTable  # PowerShell version info
$env:USERNAME    # Environment variables

# Multiple assignment
$a, $b, $c = 1, 2, 3

# Increment
$count = 0
$count++          # Increment by 1
$count += 5       # Increment by 5
```

---

## 3. Strings

```powershell
# String creation
$single = 'Single quotes — no interpolation: $name'
$double = "Double quotes — interpolation works: $name"

# String methods
$str = "Hello World"
$str.Length                    # 11
$str.ToUpper()                 # HELLO WORLD
$str.ToLower()                 # hello world
$str.Replace("World", "Brady") # Hello Brady
$str.Contains("World")         # True
$str.StartsWith("Hello")       # True
$str.EndsWith("World")         # True
$str.Trim()                    # Remove leading/trailing spaces
$str.Split(" ")                # Split into array: @("Hello", "World")
$str.Substring(0, 5)           # Hello (start at 0, length 5)
$str.IndexOf("World")          # 6

# String formatting
"Name: {0}, Age: {1}" -f "Brady", 19

# Here-string (multi-line)
$text = @"
Line 1
Line 2
Line 3
"@

# String comparison
"hello" -eq "hello"      # True
"hello" -eq "HELLO"      # False
"hello" -ieq "HELLO"     # True (case insensitive)
"hello" -like "hel*"     # True (wildcard)
"hello" -match "^hel"    # True (regex)
```

---

## 4. Arrays and Collections

```powershell
# Create an array
$fruits = @("Apple", "Banana", "Cherry")
$numbers = 1..10                    # Range: 1 to 10
$mixed = @(1, "two", $true, 3.14)

# Access elements
$fruits[0]          # Apple (zero-indexed)
$fruits[-1]         # Cherry (last element)
$fruits[0..1]       # Apple, Banana (slice)

# Array properties
$fruits.Count       # 3
$fruits.Length      # 3

# Add to array
$fruits += "Date"   # Creates new array with Date added

# Array methods
$fruits -contains "Apple"     # True
$fruits -notcontains "Grape"  # True
$fruits | Sort-Object         # Sort alphabetically
$fruits | Where-Object {$_ -like "C*"}  # Filter: Cherry

# Generic List (better for adding/removing)
$list = [System.Collections.Generic.List[string]]::new()
$list.Add("Apple")
$list.Add("Banana")
$list.Remove("Apple")

# Hashtable (dictionary)
$person = @{
    Name = "Brady"
    Age = 19
    City = "Winnipeg"
}

# Access hashtable
$person["Name"]      # Brady
$person.Name         # Brady
$person.Keys         # Name, Age, City
$person.Values       # Brady, 19, Winnipeg

# Add/modify hashtable
$person["Country"] = "Canada"
$person.Remove("City")
```

---

## 5. Operators

```powershell
# Arithmetic
5 + 3     # 8
10 - 4    # 6
3 * 4     # 12
10 / 3    # 3.333...
10 % 3    # 1 (modulus)

# Comparison
5 -eq 5       # Equal: True
5 -ne 4       # Not equal: True
5 -gt 3       # Greater than: True
5 -lt 10      # Less than: True
5 -ge 5       # Greater than or equal: True
5 -le 5       # Less than or equal: True

# Logical
$true -and $false    # False
$true -or $false     # True
-not $true           # False
!$true               # False

# String operators
"abc" -like "a*"     # True (wildcard)
"abc" -notlike "x*"  # True
"abc" -match "^a"    # True (regex)
"abc" -replace "b","B"  # aBc

# Null check
$var -eq $null
$null -eq $var       # Safer way to check null
```

---

## 6. Control Flow

```powershell
# If/ElseIf/Else
$age = 19
if ($age -ge 18) {
    Write-Host "Adult"
} elseif ($age -ge 13) {
    Write-Host "Teenager"
} else {
    Write-Host "Child"
}

# Switch
$day = "Monday"
switch ($day) {
    "Monday"    { Write-Host "Start of week" }
    "Friday"    { Write-Host "End of week" }
    "Saturday"  { Write-Host "Weekend" }
    "Sunday"    { Write-Host "Weekend" }
    Default     { Write-Host "Midweek" }
}

# For loop
for ($i = 0; $i -lt 5; $i++) {
    Write-Host "Count: $i"
}

# ForEach loop
$servers = @("server1", "server2", "server3")
foreach ($server in $servers) {
    Write-Host "Checking $server"
    Test-NetConnection -ComputerName $server
}

# ForEach-Object (pipeline)
$servers | ForEach-Object {
    Write-Host "Checking $_"
}

# While loop
$count = 0
while ($count -lt 5) {
    Write-Host "Count: $count"
    $count++
}

# Do-While (runs at least once)
$count = 0
do {
    Write-Host "Count: $count"
    $count++
} while ($count -lt 5)

# Break and Continue
foreach ($i in 1..10) {
    if ($i -eq 5) { break }     # Exit loop
    if ($i -eq 3) { continue }  # Skip to next iteration
    Write-Host $i
}
```

---

## 7. Functions

```powershell
# Basic function
function Get-Greeting {
    Write-Host "Hello World"
}
Get-Greeting    # Call the function

# Function with parameters
function Get-UserInfo {
    param (
        [string]$Username,
        [string]$Domain = "company.com"  # Default value
    )
    Write-Host "User: $Username@$Domain"
}
Get-UserInfo -Username "jsmith"
Get-UserInfo -Username "brady" -Domain "argusops.ca"

# Function with typed and mandatory parameters
function New-BackupJob {
    param (
        [Parameter(Mandatory=$true)]
        [string]$VMName,

        [Parameter(Mandatory=$false)]
        [string]$BackupPath = "D:\Backups",

        [ValidateSet("Full","Incremental","Differential")]
        [string]$Type = "Full"
    )

    Write-Host "Backing up $VMName to $BackupPath ($Type backup)"
}

# Function that returns a value
function Get-DiskUsagePercent {
    param ([string]$Drive = "C")
    $disk = Get-PSDrive $Drive
    $used = $disk.Used
    $total = $used + $disk.Free
    return [math]::Round(($used / $total) * 100, 2)
}
$usage = Get-DiskUsagePercent -Drive "C"
Write-Host "C drive is $usage% full"

# Advanced function (supports -Verbose, -WhatIf, etc.)
function Remove-OldLogs {
    [CmdletBinding(SupportsShouldProcess=$true)]
    param (
        [string]$Path = "C:\Logs",
        [int]$DaysOld = 30
    )
    $cutoff = (Get-Date).AddDays(-$DaysOld)
    $files = Get-ChildItem $Path | Where-Object {$_.LastWriteTime -lt $cutoff}
    foreach ($file in $files) {
        if ($PSCmdlet.ShouldProcess($file.FullName, "Delete")) {
            Remove-Item $file.FullName
        }
    }
}
Remove-OldLogs -WhatIf    # Preview without deleting
Remove-OldLogs             # Actually delete
```

---

## 8. Error Handling

```powershell
# Try/Catch/Finally
try {
    $result = Get-Content "C:\missing-file.txt" -ErrorAction Stop
    Write-Host "File contents: $result"
}
catch {
    Write-Host "Error: $($_.Exception.Message)"
}
finally {
    Write-Host "This always runs"
}

# ErrorAction parameter
Get-Process "nonexistent" -ErrorAction SilentlyContinue  # Suppress errors
Get-Process "nonexistent" -ErrorAction Stop              # Treat as terminating

# Check if last command succeeded
Get-Process "explorer"
if ($?) {
    Write-Host "Command succeeded"
} else {
    Write-Host "Command failed"
}

# $Error variable
$Error[0]              # Most recent error
$Error[0].Exception    # Exception object
$Error.Clear()         # Clear error history
```

---

## 9. Working with Files and the System

```powershell
# File operations
Get-ChildItem C:\Logs                    # List files
Get-ChildItem C:\Logs -Recurse           # Recursive
Get-ChildItem C:\Logs -Filter "*.log"    # Filter by extension
Get-ChildItem C:\Logs -Recurse -Filter "*.log" | Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-7)}

New-Item -Path "C:\Temp\test.txt" -ItemType File     # Create file
New-Item -Path "C:\Temp\NewFolder" -ItemType Directory # Create folder
Copy-Item -Path "C:\source.txt" -Destination "D:\dest.txt"
Move-Item -Path "C:\old.txt" -Destination "C:\new.txt"
Remove-Item -Path "C:\temp.txt"
Remove-Item -Path "C:\TempFolder" -Recurse -Force

Get-Content "C:\file.txt"                # Read file
Set-Content "C:\file.txt" -Value "text"  # Write (overwrite)
Add-Content "C:\file.txt" -Value "text"  # Append
Out-File -FilePath "C:\output.txt"       # Redirect pipeline output to file

# System information
Get-Process                              # Running processes
Get-Service                              # Windows services
Get-EventLog -LogName System -Newest 50 # Event log
Get-WmiObject Win32_ComputerSystem       # Computer info
Get-WmiObject Win32_LogicalDisk          # Disk info
Get-NetAdapter                           # Network adapters
Get-NetIPAddress                         # IP addresses

# Registry
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion"
Set-ItemProperty -Path "HKLM:\SOFTWARE\MyApp" -Name "Setting" -Value "Value"
New-Item -Path "HKLM:\SOFTWARE\MyApp"
Remove-Item -Path "HKLM:\SOFTWARE\MyApp"
```

---

## 10. Practical IT Admin Scripts

### Check Disk Space on Multiple Servers
```powershell
$servers = @("server1", "server2", "server3")
$threshold = 20  # Alert if less than 20% free

foreach ($server in $servers) {
    $disks = Get-WmiObject Win32_LogicalDisk -ComputerName $server -Filter "DriveType=3"
    foreach ($disk in $disks) {
        $freePercent = [math]::Round(($disk.FreeSpace / $disk.Size) * 100, 1)
        if ($freePercent -lt $threshold) {
            Write-Warning "$server - Drive $($disk.DeviceID): Only $freePercent% free!"
        } else {
            Write-Host "$server - Drive $($disk.DeviceID): $freePercent% free"
        }
    }
}
```

### Find and Disable Inactive AD Users
```powershell
Import-Module ActiveDirectory

$cutoffDate = (Get-Date).AddDays(-90)
$inactiveUsers = Get-ADUser -Filter {
    LastLogonDate -lt $cutoffDate -and Enabled -eq $true
} -Properties LastLogonDate

foreach ($user in $inactiveUsers) {
    Write-Host "Disabling: $($user.Name) - Last logon: $($user.LastLogonDate)"
    Disable-ADAccount -Identity $user.SamAccountName
    Set-ADUser -Identity $user.SamAccountName `
      -Description "Disabled $(Get-Date -Format 'yyyy-MM-dd') - Inactive 90+ days"
}
```

### Bulk Create Users from CSV
```powershell
# CSV format: Name,Username,Department,Password
$users = Import-Csv "C:\new-users.csv"

foreach ($user in $users) {
    New-ADUser `
      -Name $user.Name `
      -SamAccountName $user.Username `
      -Department $user.Department `
      -AccountPassword (ConvertTo-SecureString $user.Password -AsPlainText -Force) `
      -ChangePasswordAtLogon $true `
      -Enabled $true
    Write-Host "Created user: $($user.Username)"
}
```

### Service Monitor and Restart
```powershell
$services = @("Spooler", "BITS", "WinRM")

foreach ($svcName in $services) {
    $svc = Get-Service $svcName
    if ($svc.Status -ne "Running") {
        Write-Warning "$svcName is $($svc.Status) — restarting..."
        try {
            Start-Service $svcName -ErrorAction Stop
            Write-Host "$svcName restarted successfully"
        } catch {
            Write-Error "Failed to restart $svcName`: $($_.Exception.Message)"
        }
    } else {
        Write-Host "$svcName is running"
    }
}
```

---

## Quick Reference

```powershell
# Help
Get-Help cmdlet-name -Examples

# Variables
$var = "value"
$array = @("a","b","c")
$hash = @{Key="Value"}

# Pipeline
Get-Process | Where-Object {$_.CPU -gt 10} | Select-Object Name, CPU | Sort-Object CPU -Descending

# Comparison
-eq -ne -gt -lt -ge -le -like -match -contains

# String
$str.ToUpper() / .ToLower() / .Replace() / .Split() / .Trim()

# Files
Get-ChildItem / New-Item / Copy-Item / Move-Item / Remove-Item / Get-Content / Set-Content

# Error handling
try { } catch { Write-Host $_.Exception.Message } finally { }

# Functions
function Name { param([string]$p) { code } }
```

---

## Notes
- PowerShell returns **objects** not text — use `Select-Object`, `Where-Object`, and `Sort-Object` to work with properties
- Always use `-ErrorAction Stop` in `try` blocks to catch non-terminating errors
- Use `Get-Help` and `-WhatIf` parameters constantly — they prevent mistakes
- PowerShell 7+ is cross-platform and the future — prefer it over Windows PowerShell 5.1 for new scripts
- Store scripts in version control (Git) — treat scripts like code
- Use `#Requires -RunAsAdministrator` at the top of scripts that need admin rights

---

## Related Documents
- [Bash Basics](bash-basics.md)
- [Python Basics](python-basics.md)
- [Automation Principles](automation-principles.md)
- [Active Directory User Management](../Windows/active-directory-user-management.md)
