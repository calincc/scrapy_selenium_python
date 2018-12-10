# Scrapy with selenium
Scrapy middleware to handle javascript pages using selenium.

## Installation
```
$ pip install scrapy-selenium-python
```
You should use **python>=3.5**. 
You will also need one of the Selenium [compatible browsers](http://www.seleniumhq.org/about/platforms.jsp).

## Configuration
1. Add the browser to use, the path to the executable, and the arguments to pass to the executable to the scrapy settings:
    ```python
    from shutil import which

    SELENIUM_DRIVER_NAME='firefox'
    SELENIUM_DRIVER_EXECUTABLE_PATH=which('geckodriver')
    SELENIUM_DRIVER_ARGUMENTS=['-headless']  # '--headless' if using chrome instead of firefox
    ```

2. Add the `SeleniumMiddleware` to the downloader middlewares:
    ```python
    DOWNLOADER_MIDDLEWARES = {
        'scrapy_selenium_legacy.SeleniumMiddleware': 800
    }
    ```
## Usage
Use the `scrapy_selenium_legacy.SeleniumRequest` instead of the scrapy built-in `Request` like below:
```python
from scrapy_selenium_legacy import SeleniumRequest

yield SeleniumRequest(url, self.parse_result)
```
The request will be handled by selenium, and the response will have an additional `meta` key, named `driver` containing the selenium driver with the request processed.
```python
def parse_result(self, response):
    print(response.meta['driver'].title)
```
For more information about the available driver methods and attributes, refer to the [selenium python documentation](http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.remote.webdriver)

The `selector` response attribute work as usual (but contains the html processed by the selenium driver).
```python
def parse_result(self, response):
    print(response.selector.xpath('//title/@text'))
```

### Additional arguments
The `scrapy_selenium_legacy.SeleniumRequest` accept 3 additional arguments:

#### `wait_time` / `wait_until`

When used, selenium will perform an [Explicit wait](http://selenium-python.readthedocs.io/waits.html#explicit-waits) before returning the response to the spider.
```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC

yield SeleniumRequest(
    url,
    self.parse_result,
    wait_time=10,
    wait_until=EC.element_to_be_clickable((By.ID, 'someid'))
)
```

#### `screenshot`
When used, selenium will take a screenshot of the page and the binary data of the .png captured will be added to the response `meta`:
```python
yield SeleniumRequest(
    url,
    self.parse_result,
    screenshot=True
)

def parse_result(self, response):
    with open('image.png', 'wb') as image_file:
        image_file.write(response.meta['screenshot])
```

