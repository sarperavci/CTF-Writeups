# So Many 64s

This is actually pretty simple. We are given a multiple times base64 encoded string. We need to decode it to get the flag.

The challenge file is [here](./files/flag.txt) and to reach the challenge page click [here](https://ctflearn.com/challenge/121).

## Solution

Here's a recursive Python function to decode the base64 encoded string multiple times until we get the flag.

```python
import base64

def decode(data):
    try:
        return decode(base64.b64decode(data))
    except:
        return data

with open("flag.txt", "r") as f:
    data = f.read().strip()
    print(decode(data).decode())
```

