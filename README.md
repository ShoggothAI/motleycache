# motleycache

A universal caching engine for LLM apps and beyond.

Motleycache caches LLM and tool calls, as well as other web requests out of the box.
It works with all requests made using most popular Python HTTP clients: [requests](https://github.com/psf/requests), [HTTPX](https://github.com/projectdiscovery/httpx), and [Curl CFFI](https://github.com/yifeikong/curl_cffi).


## Installation
```pip install motleycache```

## Usage
```python
from motleycache import enable_cache, disable_cache

enable_cache()
# Now all requests made inside your code or any libraries you use will use cache if available

# To turn caching off, call this:
disable_cache()
```


### Whitelist and blacklist

If you want to only cache certain URLs, you can use a whitelist:
```python
from motleycache import set_cache_whitelist
set_cache_whitelist(["*://api.openai.com/*"])
# Now only requests to OpenAI API will be cached
```

Alternatively, you can specify a blacklist. Note that these two options are mutually exclusive.
```python
from motleycache import set_cache_blacklist
set_cache_blacklist(["*://api.openai.com/*"])
# All requests except those made to OpenAI API will be cached
```


### Cache location
By default, the cache is stored in the default user cache location, which depends on the platform.  
It's possible to set the cache location manually:
```python
from motleycache import set_cache_location
set_cache_location("/path/to/cache/dir")  # relative or absolute
```


### Advanced options
There is an option to block all non-cached requests. It is particularly useful for testing your applications.  
Motleycache will raise a `StrongCacheException` whenever there's no cache for a request:
```python
from motleycache import set_strong_cache
set_strong_cache(True)  # turn on
set_strong_cache(False)  # turn off
```

If you want to only use existing cache, this option will disable updating the cache with new requests:
```python
from motleycache import set_update_cache_if_exists
set_update_cache_if_exists(True)  # turn on
set_update_cache_if_exists(False)  # turn off
```


## Motleycrew and Lunary integration
Motleycache was first designed to be a part of our multi-agent framework [motleycrew](https://github.com/ShoggothAI/motleycrew) (check it out!), but we soon decided to make it a separate lightweight library instead. Because of that, motleycache also provides integration with [Lunary](https://github.com/lunary-ai/lunary) observability platform, so that cached calls are indicated in your traces.  
To find out more, see our [Caching and observability](https://motleycrew.readthedocs.io/en/latest/caching_observability.html) docs page.


## How it works
When `enable_cache` is called, we decorate the request functions of supported clients with our caching wrapper. When a request is made to a cacheable URL, a hash of all its parameters is computed, and the corresponding cache file (containing pickled response) is retrieved if it exists, and its contents are unpickled and returned to the caller.  
In case of a cache miss, the request is passed on to the original client function (unless strong caching is enabled), and the response is then added to the cache (unless cache updating is turned off).

If you use some other HTTP client and want us to support it, please raise an issue in this repo. Contributions are also welcome!
