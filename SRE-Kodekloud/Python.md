

## Set up python env


```sh
python3 -m venv venv
```

activate and installing module requests:

```sh
source venv/bin/activate && pip install requests
```


### Running python application

```sh
./venv/bin/python health-check.py
```



## Health-check.py


```python
import requests

def check_status():
    # This endpoint is simulated to be very slow
    url = 'http://127.0.0.1:5000/slow-api'
    print(f'Checking {url}...')

    # Added a 2-second timeout inside the parentheses
    response = requests.get(url,timeout=2)
    print(f'Done! Status: {response.status_code}')

if __name__ == '__main__':
    try:
        check_status()
    except requests.exceptions.Timeout:
        print('Error: The request timed out!')
    except Exception as e:
        print(f'An error occurred: {e}')
```



### psutil


Basically, Python uses the **psutil** library to talk to your operating system and pull real-time hardware data.

```python
import psutil

# Get CPU usage as a percentage
cpu_usage = psutil.cpu_percent(interval=1)
print(f"CPU is at {cpu_usage}%")

# Get memory details
mem = psutil.virtual_memory()
print(f"Available memory: {mem.available / (1024**2):.2f} MB")

# Check if disk usage is over 90%
disk = psutil.disk_usage('/')
if disk.percent > 90:
    print("Alert: Disk space is low!")
```


### psutil health_check.py


```python
import psutil

def check_disk_usage(path):
    # Get disk usage for the given path
    usage = psutil.disk_usage(path)
    return usage.percent

if __name__ == '__main__':
    # Set the threshold to 90%
    threshold = 90
    current_usage = check_disk_usage('/')
    
    print(f"Current usage is: {current_usage}%")
    
    # Add logic here to print 'ALERT: Disk usage high!' if usage > threshold
    if current_usage > threshold:
        print(f"ALERT: Disk usage high!")
    else:
        print(f"Disk are ok!")
```


This is spot on. We use the virtual_memory function to get RAM stats, and since the result is in bytes, dividing by 1024 squared gives us the value in Megabytes.
```python
mem = psutil.virtual_memory()
print(mem.available / (1024**2))
```



### Monitor cpu load

```python
import os
import psutil

def get_load_status():
    # Get the 1-minute load average
    # os.getloadavg() returns (1min, 5min, 15min)
    load1, _, _ = os.getloadavg()
    
    # Get the number of CPU cores
    cores = psutil.cpu_count()
    
    # Calculate load per core
    load_per_core = load1 / cores
    
    if load_per_core > 0.7:
        return f"HIGH LOAD: {load1} on {cores} cores"
    else:
        return f"NORMAL LOAD: {load1} on {cores} cores"

if __name__ == "__main__":
    print(get_load_status())
```



## Logging wiht JSON

Basically, **Structured Logging** turns your log messages into organized data objects, usually in JSON format. Instead of a long string, every log entry becomes a set of key-value pairs that machines can read instantly.



```python
import logging
from pythonjsonlogger import jsonlogger

logger = logging.getLogger()
logHandler = logging.StreamHandler()

# We tell the formatter which fields to include
formatter = jsonlogger.JsonFormatter('%(timestamp)s %(levelname)s %(message)s')
logHandler.setFormatter(formatter)
logger.addHandler(logHandler)

# This output is now a searchable JSON object
logger.error("Database connection failed", extra={'db_host': '10.0.0.5', 'retry': 3})
```

Structured logs turn messy text into searchable data that your monitoring tools can actually understand and act upon.