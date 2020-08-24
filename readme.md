browsermob-proxy-py
===================

Python client for the BrowserMob Proxy 2.0 REST API.



How to use with selenium-webdriver
----------------------------------

Manually:

``` python 
from browsermobproxy import Server
server = Server("path/to/browsermob-proxy")
server.start()
proxy = server.create_proxy()

from selenium import webdriver
profile  = webdriver.FirefoxProfile()
profile.set_proxy(proxy.selenium_proxy())
driver = webdriver.Firefox(firefox_profile=profile)


proxy.new_har("google")
driver.get("http://www.google.co.uk")
proxy.har # returns a HAR JSON blob

server.stop()
driver.quit()

```

for Chrome use

```
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument("--proxy-server={0}".format(proxy.proxy))
browser = webdriver.Chrome(chrome_options = chrome_options)
```

a complete example (using the splunktransactions.py file

```python
from selenium.common.exceptions import NoSuchElementException
from selenium.common.exceptions import TimeoutException
from selenium.common.exceptions import WebDriverException
from msedge.selenium_tools import Edge, EdgeOptions
from webdriver_manager.microsoft import EdgeChromiumDriverManager as ECDM
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import Select
import unittest
from splunktransactions import Transaction
from urllib.parse import urlparse
from browsermobproxy import Server
import subprocess

URI = "https://httpstat.us/500"
applicationName = (urlparse(URI)).netloc.replace(":", "-")
responseTimeout = 5


class WebsiteTest(unittest.TestCase):

    def setUp(self):
        self.pserver = Server(path='C:/Users/bdcoope2/Downloads/browsermob-proxy-2.1.4/bin/browsermob-proxy')
        self.pserver.start()
        self.proxy = self.pserver.create_proxy()
        options = EdgeOptions()
        options.use_chromium = True
        options.add_argument('--ignore-ssl-errors=yes')
        options.add_argument('--ignore-certificate-errors')
        options.add_argument("--proxy-server={}".format(self.proxy.proxy))
        # options.add_argument("headless")
        # options.add_argument("InPrivate")
        options.add_experimental_option("excludeSwitches", ["enable-logging"])
        self.driver = Edge(ECDM().install(), options=options)
        self.driver.implicitly_wait(30)
        self.base_url = URI
        self.driver.set_page_load_timeout(responseTimeout)
        self.verificationErrors = []
        self.accept_next_alert = True

    def test(self):
        driver = self.driver
        a = Transaction(driver, applicationName)
        self.proxy.new_har("main_request", options={})  # Optional:  options={'captureContent': True}
        try:
            print(a.TransactionStart(driver, 'initial'))
            driver.get(self.base_url + "/")
            WebDriverWait(driver, responseTimeout).until(EC.title_is("httpstat.us"))
            print(a.TransactionEnd(driver, 'initial'))
        except TimeoutException:
            print(a.TransactionError(driver, 'initial', self.proxy))
        except NoSuchElementException as ex:
            print(ex)
        except WebDriverException as ex:
            print(ex)

    def tearDown(self):
        self.driver.close()
        self.driver.quit()
        self.pserver.stop()
        self.assertEqual([], self.verificationErrors)
        del self.driver


if __name__ == "__main__":
    unittest.main(warnings='ignore')
```


Running Tests
-------------

Install pytest if you don't have it already.

```bash
$ pip install pytest
```

Start a browsermob instance.

```bash
$ java -jar browsermob.jar --port 9090
```

In a separate window:

```bash
$ py.test
```

If you are going to watch the test, the 'human' ones should display an english
muffin instead of the american flag on the 'pick your version' page. Or at
least it does from Canada.

To run the tests in a CI environment, disable the ones that require human
judgement by adding "-m "not human" test" to the py.test command.

```bash
$ py.test -m "not human" test
```

See also
--------

* http://proxy.browsermob.com/
* https://github.com/webmetrics/browsermob-proxy

Note on Patches/Pull Requests
-----------------------------

* Fork the project.
* Make your feature addition or bug fix.
* Add tests for it. This is important so I don't break it in a
  future version unintentionally.
* Send me a pull request. Bonus points for topic branches.

Copyright
---------

Copyright 2011 David Burns 

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.


