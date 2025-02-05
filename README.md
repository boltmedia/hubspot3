[![image](https://travis-ci.org/jpetrucciani/hubspot3.svg?branch=master)](https://travis-ci.org/jpetrucciani/hubspot3)
[![PyPI
version](https://badge.fury.io/py/hubspot3.svg)](https://badge.fury.io/py/hubspot3)
[![Code style:
black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/ambv/black)
[![Documentation
Status](https://readthedocs.org/projects/hubspot3/badge/?version=latest)](https://hubspot3.readthedocs.io/en/latest/?badge=latest)
[![Python 3.5+
supported](https://img.shields.io/badge/python-3.5+-blue.svg)](https://www.python.org/downloads/release/python-350/)

A python wrapper around HubSpot's APIs, _for python 3.5+_.

Built initially around hapipy, but heavily modified.

Check out the [documentation
here](https://hubspot3.readthedocs.io/en/latest/)\! (thanks readthedocs)

# Quick start

## Installation

```bash
# install hubspot3
pip install hubspot3
```

## Basic Usage

```python
from hubspot3 import Hubspot3

API_KEY = "your-api-key"

client = Hubspot3(api_key=API_KEY)

# all of the clients are accessible as attributes of the main Hubspot3 Client
contact = client.contacts.get_contact_by_email('testingapis@hubspot.com')
contact_id = contact['vid']

all_companies = client.companies.get_all()

# new usage limit functionality - keep track of your API calls
client.usage_limits
# <Hubspot3UsageLimits: 28937/1000000 (0.028937%) [reset in 22157s, cached for 299s]>

client.usage_limits.calls_remaining
# 971063
```

## Individual Clients

```python
from hubspot3.companies import CompaniesClient

API_KEY = "your-api-key"

client = CompaniesClient(api_key=API_KEY)

for company in client.get_all():
    print(company)
```

## Passing Params

```python
import json
from hubspot3.deals import DealsClient

deal_id = "12345"
API_KEY = "your_api_key"

deals_client = DealsClient(api_key=API_KEY)

params = {
    "includePropertyVersions": "true"
}  # Note values are camelCase as they appear in the Hubspot Documentation!

deal_data = deals_client.get(deal_id, params=params)
print(json.dumps(deal_data))
```

## Command-line interface

There is also a command-line tool available. Install the extra
requirement for that tool via:

```bash
pip install hubspot3[cli]
```

and you can use it as a command:

```bash
hubspot3 --help
```

See the Sphinx documentation for more details and explanations.

# Rate Limiting

Be aware that this uses the HubSpot API directly, so you are subject to
all of the [guidelines that HubSpot has in
place](https://developers.hubspot.com/apps/api_guidelines).

at the time of writing, HubSpot has the following limits in place for
API requests:

Free & Starter:

-   10 requests per second
-   250,000 requests per day.

Professional & Enterprise:

-   10 requests per second
-   500,000 requests per day.

This daily limit resets at midnight based on the time zone setting of
the HubSpot account. There is also an additional addon you can purchase
for more requests.

# Retrying API Calls

By default, hubspot3 will attempt to retry all API calls up to 2 times
upon failure.

If you'd like to override this behavior, you can add a `number_retries`
keyword argument to any Client constructor, or to individual API calls.

# Extending the BaseClient - thanks [@Guysoft](https://github.com/guysoft)\!

Some of the APIs are not yet complete\! If you'd like to use an API that
isn't yet in this repo, you can extend the BaseClient class\!

```python
import json
from hubspot3.base import BaseClient


PIPELINES_API_VERSION = "1"


class PipelineClient(BaseClient):
    """
    Lets you extend to non-existing clients, this example extends pipelines
    """

    def __init__(self, *args, **kwargs):
        super(PipelineClient, self).__init__(*args, **kwargs)

    def get_pipelines(self, **options):
        params = {}

        return self._call("pipelines", method="GET", params=params)

    def _get_path(self, subpath):
        return "deals/v{}/{}".format(
            self.options.get("version") or PIPELINES_API_VERSION, subpath
        )


if __name__ == "__main__":
    API_KEY = "your_api_key"
    a = PipelineClient(api_key=API_KEY)
    print(json.dumps(a.get_pipelines()))
```

# Testing

I'm currently working on rewriting many of the tests with
[pytest](https://docs.pytest.org/en/latest/) to work against the public
API and ensure that we get the correct type of mock data back. These
tests are currently in a **very** early state - I'll be working soon to
get them all built out.

```bash
# run tests
make
# or
make test_all
```
