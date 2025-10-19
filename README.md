
---

## 6 — Endpoint Fuzzer (safe request generator) (`src/endpoint_fuzzer.py`)

**Code**
```python
#!/usr/bin/env python3
"""
Endpoint Fuzzer (safe mode)
- Generates harmless HTTP requests with varying params for testing endpoints
- Rate-limited and can be pointed at local test servers only
Run: python3 src/endpoint_fuzzer.py
"""
import requests, random, time, sys, urllib.parse

def fuzz_params(base_params, n=10):
    for _ in range(n):
        params = {k: (v + str(random.randint(0,100))) for k,v in base_params.items()}
        yield params

def safe_post(url, params):
    parsed = urllib.parse.urlparse(url)
    if parsed.netloc not in ("localhost","127.0.0.1"):
        raise RuntimeError("Fuzzer restricted to localhost for safety")
    r = requests.get(url, params=params, timeout=5)
    print("GET", r.url, "=>", r.status_code)
    time.sleep(0.1)

def demo():
    url = "http://localhost:8000/test"
    base = {"q":"test"}
    for p in fuzz_params(base, 20):
        try:
            safe_post(url, p)
        except Exception as e:
            print("err", e)

if __name__ == "__main__":
    demo()
