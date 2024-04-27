# RSA Noob

RSA Noob is a cryptography challenge from [CTFLearn](https://ctflearn.com/challenge/120).

It is actually pretty simple. We are given a public key and a ciphertext. We need to decrypt the ciphertext to get the flag.

### Table of contents:

- [Challenge](#challenge)
- [Solution](#solution)
- [Bonus](#bonus)

## Challenge

Here's the challenge file:

```
e: 1
c: 9327565722767258308650643213344542404592011161659991421
n: 245841236512478852752909734912575581815967630033049838269083
```

## Solution

Let's remember the RSA algorithm:

1. Choose two large prime numbers, `p` and `q`.
2. Compute `n = p * q`.
3. Compute `phi = (p - 1) * (q - 1)`.
4. Choose an integer `e` such that `1 < e < phi` and `e` is coprime to `phi`.
5. Compute `d` such that `d * e ≡ 1 (mod phi)`.
6. The public key is `(e, n)` and the private key is `(d, n)`.
7. To encrypt a message `m`, compute `c = m^e (mod n)`.
8. To decrypt a ciphertext `c`, compute `m = c^d (mod n)`.

We have the public key `(e, n)` and the ciphertext `c`. We need to decrypt the ciphertext to get the flag. Since `n` is small, we can factorize it to get `p` and `q`. Then we can compute `phi` and `d` to decrypt the ciphertext.

Here's the Python script to factorize `n`.

```python
from sage.all import *

n = 245841236512478852752909734912575581815967630033049838269083
factorized = factor(n)
p = factorized[0][0]
q = factorized[1][0]
print(f"p: {p}")
print(f"q: {q}")
```
We are able to factorize `n` into `p` and `q`. Now we can compute `phi` and `d` to decrypt the ciphertext.

```python
from Crypto.Util.number import inverse

e = 1
c = 9327565722767258308650643213344542404592011161659991421
n = 245841236512478852752909734912575581815967630033049838269083
phi = (p - 1) * (q - 1)
d = inverse(e, phi)
m = pow(c, d, n)
print(bytes.fromhex(hex(m)[2:]).decode())
```

## Bonus

Since `e` is 1, we can directly decrypt the ciphertext without factorizing `n`.

```python
c = 9327565722767258308650643213344542404592011161659991421
print(bytes.fromhex(hex(m)[2:]).decode())
```

This is because  `m = c^e (mod n)` is equal to `m = c` when `c^e` less than `n`. There's no need to compute `d` in this case.