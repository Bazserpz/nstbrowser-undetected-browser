# nstbrowser-undetected-browser
https://nstbrowser.io/ Providing a blazing fast framework for web automation, webscraping, bots and any other creative ideas which are normally hindered by annoying anti bot systems like Captcha / CloudFlare / Imperva / hCaptcha



# NSTBrowser Selenium Integration Guide

This guide provides detailed instructions on integrating NSTBrowser with Selenium for advanced web automation tasks. It covers connecting to a launched browser, launching and connecting to a browser with a specific profile, and creating and connecting to a browser with custom configurations.

## Getting Started

To integrate NSTBrowser with Selenium, follow these steps:

1. **Launch the NSTBrowser agent client**: The client listens on port `8848`.
2. **Obtain the remote debugging address**: Get the `browserWSEndpoint` for connecting to Selenium.

### 1. Connect to a Launched Browser

To connect to a browser that has already been launched, use the following URL format:

```
http://localhost:8848/devtool/browser/{profileId}?x-api-key={x-api-key}
```

You must launch the browser corresponding to the profile in advance. Here's how you can connect:

```python
import json
from urllib.parse import urlencode
import requests
from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService

def connect_to_launched_browser(profile_id: str):
    host = '127.0.0.1'
    api_key = 'your apiKey'
    query = urlencode({
        'x-api-key': api_key,  # required
    })

    url = f'http://{host}:8848/devtool/browser/{profile_id}?{query}'
    print('devtool url: ' + url)

    port = get_debugger_port(url)
    debugger_address = f'{host}:{port}'
    print("debugger_address: " + debugger_address)

    exec_selenium(debugger_address)

connect_to_launched_browser('your profileId')
```

### 2. Launch and Connect to Browser

To launch a new browser with a specific profile and connect to it, use the following URL format:

```
http://localhost:8848/devtool/launch/{profileId}?x-api-key={x-api-key}&config={config}
```

You need to create the corresponding profile in advance and support custom configurations:

```python
import json
from urllib.parse import quote, urlencode
import requests
from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService

def launch_and_connect_to_browser(profile_id: str):
    host = '127.0.0.1'
    api_key = 'your apiKey'
    config = {
        'headless': True,  # support: True, 'new'
        'autoClose': True,
    }
    query = urlencode({
        'x-api-key': api_key,  # required
        'config': quote(json.dumps(config))
    })

    url = f'http://{host}:8848/devtool/launch/{profile_id}?{query}'
    print('devtool url: ' + url)

    port = get_debugger_port(url)
    debugger_address = f'{host}:{port}'
    print("debugger_address: " + debugger_address)

    exec_selenium(debugger_address)

launch_and_connect_to_browser('your profileId')
```

### 3. Create and Connect to Browser

Create a new Profile based on config and launch the browser. This method supports custom configurations:

```python
# Coming soon
```

### Config Parameters Notes

- `config.once`: Disposable browser that does not create a Profile and automatically closes after execution.
- `config.headless`: Enable headless mode; "new" for new headless mode.
- Additional parameters include `autoClose`, `timedCloseSec`, `remoteDebuggingPort`, and `fingerprint` configurations.

### Common Code for Selenium Execution

```python
def get_debugger_port(url: str):
    try:
        resp = requests.get(url).json()
        if resp['data'] is None:
            raise Exception(resp['msg'])
        port = resp['data']['port']
        return port
    except HTTPError:
        raise Exception(HTTPError.response)

def exec_selenium(debugger_address: str):
    options = webdriver.ChromeOptions()
    options.add_experimental_option("debuggerAddress", debugger_address)
    chrome_driver_path = r'./webdriver/chromedriver_113.exe'
    service = ChromeService(executable_path=chrome_driver_path)
    driver = webdriver.Chrome(service=service, options=options)
    driver.get("https://nstbrowser.io")
    driver.save_screenshot('screenshot.png')
    driver.close()
    driver.quit()
```

For more detailed information and updates, refer to the [NSTBrowser API documentation](https://apidocs.nstbrowser.io/).
```
