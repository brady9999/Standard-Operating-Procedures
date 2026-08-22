# Windows Event Viewer
> A guide to using Windows Event Viewer to monitor, troubleshoot, and diagnose system and application issues.

**Category:** Windows  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **Event Viewer** | Windows Event Viewer | Built-in Windows tool for viewing system, security, and application logs |
| **Event Log** | Event Log | A record of events that have occurred on the system |
| **Event ID** | Event Identifier | A unique number assigned to each type of event e.g. Event ID 4625 = failed login |
| **Source** | Event Source | The application or component that generated the event |
| **Level** | Event Level | The severity of the event — Critical, Error, Warning, Information, Verbose |
| **Channel** | Log Channel | The category of log — System, Application, Security, Setup |
| **XML** | Extensible Markup Language | The raw format of event log data, useful for advanced filtering |
| **Subscription** | Event Subscription | Collect events from remote computers into a central log |
| **Task** | Scheduled Task | An action triggered automatically when a specific event occurs |

---

## Overview
Windows Event Viewer is a built-in tool that records everything that happens on a Windows system — logins, errors, warnings, application crashes, service starts and stops, and security events. It is one of the first places an IT administrator checks when troubleshooting an issue.

Every event has:
- A **date and time**
- A **source** (what generated it)
- An **Event ID** (what type of event it is)
- A **level** (how severe it is)
- A **description** (what happened)

---

## Prerequisites
- Standard user account can view Application and System logs
- Administrator account required to view Security logs
- No additional software required — built into Windows

---

## How to Open Event Viewer

**Method 1 — Run Dialog:**
1. Press **Windows + R**
2. Type `eventvwr.msc`
3. Press **Enter**

**Method 2 — Start Menu:**
1. Click **Start**
2. Search **Event Viewer**
3. Click to open

**Method 3 — Computer Management:**
1. Right-click **Start** → **Computer Management**
2. Expand **System Tools**
3. Click **Event Viewer**

**Method 4 — PowerShell / CMD:**
```powershell
eventvwr.msc
```

---

## Understanding the Interface

```
Event Viewer
├── Custom Views
│   └── Administrative Events    ← Pre-filtered view of critical/error events
├── Windows Logs
│   ├── Application              ← App crashes, errors, warnings
│   ├── Security                 ← Login attempts, privilege use, audit events
│   ├── Setup                    ← Windows installation and update events
│   ├── System                   ← Hardware, driver, service events
│   └── Forwarded Events         ← Events collected from remote computers
├── Applications and Services Logs
│   ├── Microsoft
│   ├── Windows
│   └── ...                      ← Logs from specific applications and roles
└── Subscriptions                ← Remote event collection
```

**Left Panel** — tree of all available logs
**Middle Panel** — list of events in the selected log
**Right Panel** — actions you can perform
**Bottom Panel** — details of the selected event

---

## Event Levels Explained

| Level | Icon | What It Means |
|-------|------|---------------|
| **Critical** | 🔴 | A serious failure requiring immediate attention |
| **Error** | 🔴 | A significant problem that has occurred |
| **Warning** | ⚠️ | Something that could become a problem |
| **Information** | ℹ️ | A normal event confirming something happened |
| **Verbose** | 💬 | Detailed diagnostic information |

---

## Key Log Categories

### Application Log
Records events from applications and programs installed on the system.
- App crashes
- Application errors
- Successful/failed application starts

### Security Log
Records security-related events — requires administrator access.
- User login and logout (Event ID 4624, 4634)
- Failed login attempts (Event ID 4625)
- Account lockouts (Event ID 4740)
- Privilege use
- File access auditing

### System Log
Records events from Windows system components and drivers.
- Service starts and stops
- Driver failures
- Hardware issues
- Boot events

---

## Common Event IDs to Know

| Event ID | Log | What It Means |
|----------|-----|---------------|
| **4624** | Security | Successful user login |
| **4625** | Security | Failed login attempt |
| **4634** | Security | User logged off |
| **4648** | Security | Login using explicit credentials |
| **4720** | Security | User account created |
| **4722** | Security | User account enabled |
| **4725** | Security | User account disabled |
| **4740** | Security | User account locked out |
| **4776** | Security | Domain controller validated credentials |
| **6005** | System | Event log service started (system boot) |
| **6006** | System | Event log service stopped (system shutdown) |
| **6008** | System | Unexpected system shutdown |
| **7034** | System | A service crashed unexpectedly |
| **7036** | System | A service started or stopped |
| **1074** | System | System was shut down by a user or process |
| **41** | System | System rebooted without clean shutdown (crash) |

---

## 1. Viewing Events in a Log

### GUI — Step by Step
1. Open **Event Viewer**
2. In the left panel expand **Windows Logs**
3. Click the log you want to view — **System**, **Application**, or **Security**
4. Events appear in the middle panel sorted by date — newest first
5. Click any event to see its details in the bottom panel
6. Double-click an event to open it in a full window with complete details

> **Tip:** The **Administrative Events** view under Custom Views shows only Critical, Error, and Warning events across all logs — good starting point for troubleshooting

---

## 2. Filtering Events

### GUI — Step by Step
1. Select the log you want to filter
2. In the right panel click **Filter Current Log**
3. A dialog box appears with options:
   - **Logged** — time range (Last hour, Last 24 hours, Last 7 days, Custom)
   - **Event level** — check Critical, Error, Warning, Information as needed
   - **Event source** — filter by what generated the event
   - **Event ID** — enter specific Event IDs separated by commas e.g. `4625, 4740`
4. Click **OK**
5. The log now shows only matching events
6. To clear the filter click **Clear Filter** in the right panel

### PowerShell
```powershell
# Get last 50 System log errors
Get-EventLog -LogName System -EntryType Error -Newest 50

# Get Security log events with specific Event ID
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} |
  Select-Object TimeCreated, Message -First 20

# Get all events from the last 24 hours
$yesterday = (Get-Date).AddHours(-24)
Get-WinEvent -FilterHashtable @{LogName='System'; StartTime=$yesterday}

# Find account lockout events
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4740} |
  Select-Object TimeCreated, Message
```

---

## 3. Searching for a Specific Event

### GUI — Step by Step
1. Select the log
2. In the right panel click **Find**
3. Type keywords from the event description
4. Click **Find Next**
5. Event Viewer highlights matching events

---

## 4. Creating a Custom View

Custom views save your filters so you can quickly access specific types of events.

### GUI — Step by Step
1. In the right panel click **Create Custom View**
2. Set your filter criteria:
   - Time range
   - Event levels
   - Logs to include
   - Event IDs
3. Click **OK**
4. Give the custom view a name and description
5. Click **OK**
6. The view appears under **Custom Views** in the left panel

---

## 5. Clearing a Log

> ⚠️ **Warning:** Clearing a log permanently deletes all events. Always save the log first.

### GUI — Step by Step
1. Right-click the log in the left panel
2. Select **Clear Log**
3. A dialog asks if you want to save the log before clearing
4. Click **Save and Clear** to keep a backup or **Clear** to delete immediately

---

## 6. Saving and Exporting Logs

### GUI — Step by Step
1. Right-click the log
2. Select **Save All Events As**
3. Choose a format:
   - **.evtx** — Windows Event Log format, reopenable in Event Viewer
   - **.xml** — XML format for parsing
   - **.txt** — Plain text
   - **.csv** — CSV for spreadsheet analysis
4. Choose a location and click **Save**

### PowerShell
```powershell
# Export System log to CSV
Get-EventLog -LogName System -Newest 1000 |
  Export-Csv "C:\Logs\system-log.csv" -NoTypeInformation

# Export specific events to XML
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} |
  Export-Clixml "C:\Logs\failed-logins.xml"
```

---

## 7. Viewing Logs on a Remote Computer

### GUI — Step by Step
1. Open **Event Viewer**
2. Right-click **Event Viewer (Local)** at the top of the left panel
3. Select **Connect to Another Computer**
4. Enter the computer name or IP address
5. Click **OK**
6. You can now browse that computer's event logs

### PowerShell
```powershell
# View events on a remote computer
Get-EventLog -LogName System -ComputerName "PC-NAME" -Newest 50

Get-WinEvent -ComputerName "PC-NAME" -FilterHashtable @{LogName='System'; Level=2}
```

---

## Troubleshooting with Event Viewer — Common Scenarios

### User Can't Log In
1. Open **Security** log
2. Filter for Event ID **4625**
3. Look at the failure reason in the event details:
   - **0xC000006A** — wrong password
   - **0xC0000234** — account locked out
   - **0xC0000072** — account disabled
   - **0xC000006D** — bad username

### System Crashed or Rebooted Unexpectedly
1. Open **System** log
2. Filter for Event ID **41** (unexpected reboot) or **6008** (unexpected shutdown)
3. Note the time and check surrounding events for the cause
4. Also check **Critical** level events around the same time

### Service Failed to Start
1. Open **System** log
2. Filter for Event ID **7034** or **7036**
3. Look at the source to identify which service failed
4. Check the error code in the description for the specific cause

### Application Crashing
1. Open **Application** log
2. Filter for **Error** level events
3. Look for the application name as the source
4. Note the exception code and faulting module for further research

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Security log is empty | Auditing not configured | Enable audit policies in Group Policy |
| Log fills up too fast | Max log size too small | Right-click log → Properties → increase max size |
| Can't view Security log | Insufficient permissions | Run Event Viewer as Administrator |
| Can't connect to remote computer | Firewall blocking or service not running | Enable Remote Event Log Management in Windows Firewall |
| Events show wrong time | Time zone mismatch | Check system clock and time zone settings |

---

## Quick Reference — PowerShell Commands

```powershell
# List all available logs
Get-EventLog -List

# Get newest 100 System errors
Get-EventLog -LogName System -EntryType Error -Newest 100

# Get events by Event ID
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625}

# Get events in a time range
Get-WinEvent -FilterHashtable @{
  LogName='System'
  StartTime='2026-08-01'
  EndTime='2026-08-18'
}

# Search event message for keyword
Get-WinEvent -LogName System | Where-Object {$_.Message -like "*disk*"}

# Get events from multiple logs at once
Get-WinEvent -FilterHashtable @{LogName='System','Application'; Level=2}

# Count events by source
Get-EventLog -LogName System -EntryType Error |
  Group-Object Source | Sort-Object Count -Descending
```

---

## Notes
- Always check Event Viewer before rebooting a server — it may reveal the cause of an issue
- Event ID 41 and 6008 together indicate a crash or power loss
- Security log requires audit policies to be configured via Group Policy to capture events
- Event logs are stored at `C:\Windows\System32\winevt\Logs\`
- Maximum log size and retention policy can be configured by right-clicking each log → Properties

---

## Related Documents
- [Active Directory User Management](active-directory-user-management.md)
- [Active Directory Password & Group Policy](active-directory-password-local--group-policy.md)
- [Windows Server Setup](#)
