# Python Basics
> A reference guide to Python scripting for IT automation, monitoring agents, and system administration.

**Category:** Automation  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **Python** | Python | A high-level general-purpose programming language |
| **pip** | Package Installer for Python | The tool for installing Python packages |
| **venv** | Virtual Environment | An isolated Python environment with its own packages |
| **Module** | Module | A Python file containing reusable code |
| **Package** | Package | A collection of modules distributed together |
| **Import** | Import | Loading a module or package into your script |
| **Function** | Function | A reusable block of code defined with `def` |
| **Class** | Class | A blueprint for creating objects |
| **Object** | Object | An instance of a class |
| **List** | List | An ordered, mutable collection `[1, 2, 3]` |
| **Dictionary** | Dictionary | A key-value collection `{"key": "value"}` |
| **Tuple** | Tuple | An ordered, immutable collection `(1, 2, 3)` |
| **Set** | Set | An unordered collection of unique values `{1, 2, 3}` |
| **Exception** | Exception | An error that can be caught and handled |
| **Decorator** | Decorator | A function that modifies another function |
| **Generator** | Generator | A function that yields values one at a time |
| **API** | Application Programming Interface | A way for programs to communicate |
| **JSON** | JavaScript Object Notation | A common data format for APIs |
| **requests** | requests | A popular Python library for making HTTP requests |
| **subprocess** | subprocess | A Python module for running system commands |

---

## Overview
Python is the go-to language for IT automation because of its readability, extensive library ecosystem, and versatility. It powers tools like Ansible, Fabric, and most modern monitoring agents.

**Why Python for IT automation:**
- Cleaner than Bash for complex logic
- Excellent library support (requests, paramiko, psutil, pysnmp)
- Cross-platform — same script runs on Linux and Windows
- Used in Argus Ops, Morpheus, and Iris agents

**Python 3 is the current standard — never use Python 2.**

---

## 1. Python Basics

```python
# Print output
print("Hello World")
print(f"My name is Brady")    # f-string (formatted string literal)
print("Sum:", 5 + 3)

# Comments
# This is a single line comment

"""
This is a
multi-line string
often used as a docstring
"""

# Running Python
# python3 script.py
# python3 -c "print('hello')"    # One-liner
# python3                         # Interactive shell
```

---

## 2. Variables and Data Types

```python
# Basic types
name = "Brady"          # str
age = 19                # int
height = 5.11           # float
is_admin = True         # bool
nothing = None          # NoneType

# Check type
print(type(name))       # <class 'str'>
print(isinstance(name, str))   # True

# Type conversion
str(19)                 # "19"
int("42")               # 42
float("3.14")           # 3.14
bool(0)                 # False
bool(1)                 # True
list("abc")             # ['a', 'b', 'c']

# Multiple assignment
a, b, c = 1, 2, 3
x = y = z = 0

# Constants (convention — UPPERCASE)
MAX_RETRIES = 3
DEFAULT_TIMEOUT = 30
```

---

## 3. Strings

```python
# String creation
single = 'Single quotes'
double = "Double quotes"
multi = """
Multi-line
string
"""

# f-strings (Python 3.6+) — preferred
name = "Brady"
age = 19
print(f"Name: {name}, Age: {age}")
print(f"2 + 2 = {2 + 2}")
print(f"Uppercase: {name.upper()}")

# String methods
s = "Hello World"
s.upper()           # HELLO WORLD
s.lower()           # hello world
s.strip()           # Remove whitespace
s.lstrip()          # Remove left whitespace
s.rstrip()          # Remove right whitespace
s.replace("World", "Brady")    # Hello Brady
s.split(" ")        # ["Hello", "World"]
s.split(",")        # Split by comma
" ".join(["a", "b", "c"])      # "a b c"
s.startswith("Hello")          # True
s.endswith("World")            # True
s.contains("World")            # AttributeError — use "World" in s
"World" in s                   # True
s.find("World")    # 6 (index) or -1 if not found
s.count("l")       # 3
s.strip()          # "Hello World"
len(s)             # 11

# String formatting options
"{} is {}".format(name, age)    # "Brady is 19"
"%-10s %d" % (name, age)        # Old style (avoid)

# Raw strings (no escape processing)
path = r"C:\Users\Brady\Documents"

# Multiline f-string
message = (
    f"Name: {name}\n"
    f"Age: {age}\n"
    f"Admin: {is_admin}"
)
```

---

## 4. Collections

```python
# List — ordered, mutable
fruits = ["Apple", "Banana", "Cherry"]
numbers = [1, 2, 3, 4, 5]
mixed = [1, "two", True, 3.14, None]

# Access
fruits[0]       # "Apple"
fruits[-1]      # "Cherry"
fruits[1:3]     # ["Banana", "Cherry"] (slice)
fruits[:2]      # ["Apple", "Banana"]
fruits[::2]     # Every second item

# Modify
fruits.append("Date")           # Add to end
fruits.insert(1, "Avocado")     # Insert at index
fruits.extend(["Elderberry"])   # Add multiple
fruits.remove("Banana")         # Remove by value
del fruits[0]                   # Remove by index
popped = fruits.pop()           # Remove and return last
fruits.sort()                   # Sort in place
sorted_fruits = sorted(fruits)  # Return sorted copy
fruits.reverse()                # Reverse in place
fruits.index("Cherry")          # Find index
"Apple" in fruits               # True/False
len(fruits)                     # Count

# List comprehension
squares = [x**2 for x in range(10)]
evens = [x for x in range(20) if x % 2 == 0]
upper_fruits = [f.upper() for f in fruits]

# Dictionary — key-value pairs
person = {
    "name": "Brady",
    "age": 19,
    "city": "Winnipeg"
}

# Access
person["name"]          # "Brady"
person.get("name")      # "Brady" (no KeyError if missing)
person.get("email", "N/A")  # Default if missing

# Modify
person["email"] = "brady@company.com"   # Add/update
del person["city"]                       # Delete key
person.pop("age")                        # Remove and return

# Iterate
for key in person:
    print(key)
for key, value in person.items():
    print(f"{key}: {value}")
person.keys()     # dict_keys
person.values()   # dict_values
person.items()    # dict_items

# Dict comprehension
squared = {x: x**2 for x in range(5)}

# Tuple — ordered, immutable
coords = (10.5, 20.3)
rgb = (255, 128, 0)
single_item = (42,)    # Note the comma
x, y = coords          # Unpack

# Set — unordered, unique
tags = {"python", "linux", "automation"}
tags.add("networking")
tags.remove("linux")
"python" in tags        # True
set1 = {1, 2, 3}
set2 = {2, 3, 4}
set1 | set2             # Union: {1, 2, 3, 4}
set1 & set2             # Intersection: {2, 3}
set1 - set2             # Difference: {1}
```

---

## 5. Control Flow

```python
# If/elif/else
age = 19
if age >= 18:
    print("Adult")
elif age >= 13:
    print("Teenager")
else:
    print("Child")

# One-liner (ternary)
status = "Adult" if age >= 18 else "Minor"

# For loop
for i in range(5):          # 0, 1, 2, 3, 4
    print(i)

for i in range(1, 11):      # 1 to 10
    print(i)

for i in range(0, 20, 5):   # 0, 5, 10, 15
    print(i)

for fruit in ["Apple", "Banana", "Cherry"]:
    print(fruit)

for i, fruit in enumerate(["Apple", "Banana"]):
    print(f"{i}: {fruit}")   # 0: Apple, 1: Banana

for key, value in person.items():
    print(f"{key}: {value}")

# While loop
count = 0
while count < 5:
    print(count)
    count += 1

# Break and continue
for i in range(10):
    if i == 5:
        break       # Exit loop
    if i == 3:
        continue    # Skip to next iteration
    print(i)

# Loop else (runs if loop completed without break)
for i in range(5):
    if i == 10:
        break
else:
    print("Loop completed normally")

# Match statement (Python 3.10+)
status_code = 404
match status_code:
    case 200:
        print("OK")
    case 404:
        print("Not Found")
    case 500:
        print("Server Error")
    case _:
        print("Unknown")
```

---

## 6. Functions

```python
# Basic function
def greet():
    print("Hello World")

greet()

# Function with parameters
def get_user_info(username, domain="company.com"):
    return f"{username}@{domain}"

print(get_user_info("jsmith"))
print(get_user_info("brady", "argusops.ca"))

# *args — variable positional arguments
def sum_all(*numbers):
    return sum(numbers)

print(sum_all(1, 2, 3, 4, 5))   # 15

# **kwargs — variable keyword arguments
def create_user(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

create_user(name="Brady", age=19, city="Winnipeg")

# Type hints (Python 3.5+)
def get_disk_usage(drive: str = "/") -> float:
    """
    Returns disk usage percentage for the given drive.
    
    Args:
        drive: The drive path to check
        
    Returns:
        Float representing usage percentage
    """
    import shutil
    total, used, free = shutil.disk_usage(drive)
    return round((used / total) * 100, 2)

# Lambda (anonymous function)
square = lambda x: x ** 2
double = lambda x: x * 2
add = lambda x, y: x + y

# Use with sorted/map/filter
numbers = [3, 1, 4, 1, 5, 9, 2, 6]
sorted_nums = sorted(numbers, key=lambda x: -x)    # Descending
squares = list(map(lambda x: x**2, numbers))
evens = list(filter(lambda x: x % 2 == 0, numbers))
```

---

## 7. Error Handling

```python
# Try/except/else/finally
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Math error: {e}")
except (ValueError, TypeError) as e:
    print(f"Value/Type error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
    raise   # Re-raise the exception
else:
    print("No error occurred")
finally:
    print("This always runs")

# Common exceptions
# FileNotFoundError — file doesn't exist
# PermissionError   — no access to file/dir
# ConnectionError   — network connection failed
# TimeoutError      — request timed out
# KeyError          — dict key doesn't exist
# IndexError        — list index out of range
# ValueError        — wrong value type
# TypeError         — wrong type

# Raise your own exceptions
def set_age(age):
    if not isinstance(age, int):
        raise TypeError(f"Age must be int, got {type(age)}")
    if age < 0 or age > 150:
        raise ValueError(f"Age {age} is not realistic")
    return age

# Custom exception class
class ConfigError(Exception):
    """Raised when configuration is invalid"""
    pass

def load_config(path):
    if not path.endswith(".json"):
        raise ConfigError(f"Config must be JSON, got: {path}")
```

---

## 8. File Operations

```python
import os
import json
import csv
import shutil
from pathlib import Path

# Read a file
with open("/etc/hostname", "r") as f:
    content = f.read()

# Read line by line
with open("/var/log/syslog", "r") as f:
    for line in f:
        print(line.strip())

# Read all lines into list
with open("data.txt", "r") as f:
    lines = f.readlines()

# Write to file
with open("/tmp/output.txt", "w") as f:
    f.write("Hello World\n")

# Append to file
with open("/tmp/output.txt", "a") as f:
    f.write("Another line\n")

# JSON
data = {"name": "Brady", "age": 19}

# Write JSON
with open("data.json", "w") as f:
    json.dump(data, f, indent=2)

# Read JSON
with open("data.json", "r") as f:
    loaded = json.load(f)

# CSV
rows = [["Name", "Age"], ["Brady", 19], ["Alice", 25]]

# Write CSV
with open("data.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerows(rows)

# Read CSV
with open("data.csv", "r") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row["Name"], row["Age"])

# Path operations using pathlib
path = Path("/opt/myapp")
path.exists()           # True/False
path.is_file()          # True/False
path.is_dir()           # True/False
path.mkdir(parents=True, exist_ok=True)
path.stat().st_size     # File size in bytes
list(path.glob("*.py")) # Find all .py files
path.read_text()        # Read file content
path.write_text("text") # Write content

# OS operations
os.getcwd()                     # Current directory
os.chdir("/tmp")                # Change directory
os.listdir("/etc")              # List directory
os.makedirs("/tmp/a/b/c", exist_ok=True)  # Create dirs
os.path.exists("/etc/nginx")    # Check exists
os.path.isfile("/etc/nginx/nginx.conf")
os.path.isdir("/etc/nginx")
os.path.join("/etc", "nginx", "nginx.conf")  # Join paths
os.path.basename("/etc/nginx/nginx.conf")    # "nginx.conf"
os.path.dirname("/etc/nginx/nginx.conf")     # "/etc/nginx"
shutil.copy("src.txt", "dst.txt")
shutil.copytree("src/", "dst/")
shutil.rmtree("/tmp/olddir")
```

---

## 9. System Interaction

```python
import subprocess
import psutil
import os

# Run system commands
result = subprocess.run(["ls", "-la", "/etc"], capture_output=True, text=True)
print(result.stdout)
print(result.returncode)   # 0 = success

# Run shell command (use shell=True carefully — security risk)
result = subprocess.run("df -h | grep /dev/sda", shell=True, capture_output=True, text=True)

# Check if command succeeded
try:
    subprocess.run(["systemctl", "restart", "nginx"], check=True)
    print("nginx restarted")
except subprocess.CalledProcessError as e:
    print(f"Failed to restart nginx: {e}")

# System info with psutil
# pip install psutil
import psutil

# CPU
psutil.cpu_percent(interval=1)         # CPU usage %
psutil.cpu_count()                     # Number of CPUs

# Memory
mem = psutil.virtual_memory()
mem.total     # Total RAM in bytes
mem.available # Available RAM
mem.percent   # Usage percentage

# Disk
disk = psutil.disk_usage("/")
disk.total    # Total size
disk.used     # Used space
disk.free     # Free space
disk.percent  # Usage percentage

# Network
net = psutil.net_io_counters()
net.bytes_sent      # Bytes sent
net.bytes_recv      # Bytes received

# Processes
for proc in psutil.process_iter(['pid', 'name', 'cpu_percent']):
    print(proc.info)

# Environment variables
import os
os.environ.get("HOME")          # /home/brady
os.environ.get("PATH")
os.environ.get("MYVAR", "default")  # With default
```

---

## 10. Practical IT Scripts

### SNMP Poller (Similar to Morpheus Agent)
```python
#!/usr/bin/env python3
"""Simple SNMP poller — polls a switch for interface status"""

from pysnmp.hlapi import *
import json

def get_snmp_value(host, community, oid):
    """Get a single SNMP value"""
    errorIndication, errorStatus, errorIndex, varBinds = next(
        getCmd(SnmpEngine(),
               CommunityData(community),
               UdpTransportTarget((host, 161)),
               ContextData(),
               ObjectType(ObjectIdentity(oid)))
    )
    if errorIndication:
        raise Exception(f"SNMP error: {errorIndication}")
    for varBind in varBinds:
        return str(varBind[1])

def poll_device(host, community="public"):
    """Poll basic device info"""
    results = {
        "host": host,
        "sysDescr": get_snmp_value(host, community, "1.3.6.1.2.1.1.1.0"),
        "sysName": get_snmp_value(host, community, "1.3.6.1.2.1.1.5.0"),
        "sysUpTime": get_snmp_value(host, community, "1.3.6.1.2.1.1.3.0"),
    }
    return results

if __name__ == "__main__":
    data = poll_device("192.168.1.1")
    print(json.dumps(data, indent=2))
```

### Service Health Monitor
```python
#!/usr/bin/env python3
"""Monitor services and restart if down"""

import subprocess
import time
import logging

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")

SERVICES = ["nginx", "postgresql", "myapp"]
CHECK_INTERVAL = 60  # seconds

def is_service_running(service):
    result = subprocess.run(
        ["systemctl", "is-active", service],
        capture_output=True, text=True
    )
    return result.stdout.strip() == "active"

def restart_service(service):
    try:
        subprocess.run(["systemctl", "restart", service], check=True)
        logging.info(f"Restarted {service}")
        return True
    except subprocess.CalledProcessError as e:
        logging.error(f"Failed to restart {service}: {e}")
        return False

if __name__ == "__main__":
    logging.info("Service monitor started")
    while True:
        for service in SERVICES:
            if not is_service_running(service):
                logging.warning(f"{service} is not running — restarting")
                restart_service(service)
            else:
                logging.debug(f"{service} is running")
        time.sleep(CHECK_INTERVAL)
```

### API Request Helper
```python
#!/usr/bin/env python3
"""Simple API client — similar to Iris agent reporting to Argus"""

import requests
import json
import os

BASE_URL = os.environ.get("ARGUS_URL", "http://localhost:8000")
API_TOKEN = os.environ.get("ARGUS_TOKEN", "")

def get_headers():
    return {
        "Authorization": f"Bearer {API_TOKEN}",
        "Content-Type": "application/json"
    }

def post_metrics(metrics: dict):
    """Post metrics to the Argus API"""
    try:
        response = requests.post(
            f"{BASE_URL}/api/v1/metrics",
            headers=get_headers(),
            json=metrics,
            timeout=10
        )
        response.raise_for_status()
        return response.json()
    except requests.exceptions.ConnectionError:
        raise Exception("Cannot connect to Argus API")
    except requests.exceptions.Timeout:
        raise Exception("Request timed out")
    except requests.exceptions.HTTPError as e:
        raise Exception(f"HTTP error {response.status_code}: {e}")

if __name__ == "__main__":
    import psutil
    metrics = {
        "cpu_percent": psutil.cpu_percent(interval=1),
        "memory_percent": psutil.virtual_memory().percent,
        "disk_percent": psutil.disk_usage("/").percent
    }
    result = post_metrics(metrics)
    print(json.dumps(result, indent=2))
```

---

## Quick Reference

```python
# String
f"Hello {name}"           # f-string
s.upper() / .lower() / .strip() / .split() / .replace() / .startswith()
"value" in s              # Contains check

# List
lst = [1, 2, 3]
lst.append(4) / .insert(0, 0) / .remove(2) / .pop() / .sort()
[x**2 for x in lst]      # List comprehension

# Dict
d = {"key": "value"}
d.get("key", "default")  # Safe access
d.items() / .keys() / .values()

# File
with open("file.txt", "r") as f:
    content = f.read()

# JSON
import json
json.dumps(data, indent=2)   # Dict to JSON string
json.loads(string)            # JSON string to dict
json.dump(data, file)         # Write to file
json.load(file)               # Read from file

# Subprocess
import subprocess
result = subprocess.run(["cmd", "arg"], capture_output=True, text=True, check=True)
result.stdout / .returncode

# Error handling
try:
    risky_operation()
except SpecificError as e:
    handle(e)
finally:
    cleanup()
```

---

## Notes
- Always use `with open()` for files — it automatically closes the file even if an error occurs
- Use f-strings for string formatting — they're the most readable and efficient option
- `subprocess.run()` is the modern way to run system commands — avoid `os.system()`
- Install packages in a virtual environment — `python3 -m venv venv && source venv/bin/activate`
- Use type hints for functions that others will use — they serve as built-in documentation
- `psutil` is invaluable for IT automation — CPU, memory, disk, network, and process info

---

## Related Documents
- [PowerShell Basics](powershell-basics.md)
- [Bash Basics](bash-basics.md)
- [Automation Principles](automation-principles.md)
- [Nginx Gunicorn Configuration](../Linux/nginx-gunicorn-configuration.md)
- [Systemd File Configuration](../Linux/systemd-file-configuration.md)
