python-logging-loki (with structured metadata support)
===================

[![PyPI version](https://img.shields.io/pypi/v/python-logging-loki.svg)](https://pypi.org/project/python-logging-loki/)
[![Python version](https://img.shields.io/badge/python-3.6%20%7C%203.7%20%7C%203.8-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/pypi/l/python-logging-loki.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://travis-ci.org/GreyZmeem/python-logging-loki.svg?branch=master)](https://travis-ci.org/GreyZmeem/python-logging-loki)

A fork of the python logging handler for Loki, supporting structured metadata.

https://grafana.com/loki

Installation
============
```bash
pip install python-logging-loki
```

Usage
=====

```python
import logging
import logging_loki


handler = logging_loki.LokiHandler(
    url="https://my-loki-instance/loki/api/v1/push", 
    tags={"application": "my-app"},
    auth=("username", "password"),
    version="1",
)

logger = logging.getLogger("my-logger")
logger.addHandler(handler)
logger.error(
    "Something happened", 
    extra={"tags": {"service": "my-service"}},
)
```

Example above will send `Something happened` message along with these labels:
- Default labels from handler
- Message level as `serverity`
- Logger's name as `logger` 
- Labels from `tags` item of `extra` dict

To log structured metadata, simply add a `metadata` field to `extra`:

```python
logger.info(
    f'Epoch [{epoch+1}/{epochs}], Loss: {loss:.4f}', 
    extra={"tags": {"experiment": "example"}, "metadata": {"epoch": f"{epoch}", "loss": f"{loss:.4f}"}},
    )
```

The given examples are blocking (i.e. each call will wait for the message to be sent).  
But you can use the built-in `QueueHandler` and` QueueListener` to send messages in a separate thread.  

```python
import logging.handlers
import logging_loki
from multiprocessing import Queue


queue = Queue(-1)
handler = logging.handlers.QueueHandler(queue)
handler_loki = logging_loki.LokiHandler(
    url="https://my-loki-instance/loki/api/v1/push", 
    tags={"application": "my-app"},
    auth=("username", "password"),
    version="1",
)
logging.handlers.QueueListener(queue, handler_loki)

logger = logging.getLogger("my-logger")
logger.addHandler(handler)
logger.error(...)
```

Or you can use `LokiQueueHandler` shortcut, which will automatically create listener and handler.

```python
import logging.handlers
import logging_loki
from multiprocessing import Queue


handler = logging_loki.LokiQueueHandler(
    Queue(-1),
    url="https://my-loki-instance/loki/api/v1/push", 
    tags={"application": "my-app"},
    auth=("username", "password"),
    version="1",
)

logger = logging.getLogger("my-logger")
logger.addHandler(handler)
logger.error(...)
```
