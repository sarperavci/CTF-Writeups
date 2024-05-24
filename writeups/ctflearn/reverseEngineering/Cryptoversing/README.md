# Cryptoversing Write-up - CTFLearn

[Cryptoversing](https://ctflearn.com/challenge/667) is a medium level reverse engineering challenge on CTFLearn. Basically, we are given a binary file and we need to find the flag somehow.

### Table of Contents
- [First Look](#first-look)
- [Source Code Analysis](#source-code-analysis)
- [Solution](#solution)
- [Alternative Solution](#alternative-solution)
- [Conclusion](#conclusion)

## First Look

Let's start by running the binary file.

```bash
sarp@IdeaPad:~/Desktop$ ./xor.bin 
[*] Hello! Welcome to our Program!
Enter the password to contiune:  test12345
[-] Wrong Password
```
The program asks for a password. Eventually, we need to see the source code of the program. 


## Source Code Analysis

I tried to open it with Ghidra but it didn't create a proper decompiled code. So, I decided to use an online decompiler. I used [dogBolt](https://dogbolt.org/) to decompile the binary file. DogBolt decompiles the binary file using several decompilers and shows all of them. So, I can choose the best one. It supports  angr,BinaryNinja Boomerang, dewolf, Ghidra, Hex-Rays, RecStudio, Reko, Relyze, RetDec, Snowman decompilers. I chosed [Hex-Rays decompiler](https://dogbolt.org/?id=577c8660-6c32-49af-8197-7ec8f5048eb1#angr=122&BinaryNinja=147&dewolf=61&Ghidra=197&Hex-Rays=141) and it gave me a good decompiled code.

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  int i; // [rsp+4h] [rbp-CCh]
  int j; // [rsp+8h] [rbp-C8h]
  int k; // [rsp+Ch] [rbp-C4h]
  int v7[2]; // [rsp+18h] [rbp-B8h]
  int v8[2]; // [rsp+20h] [rbp-B0h]
  int v9[2]; // [rsp+28h] [rbp-A8h]
  char v10[64]; // [rsp+30h] [rbp-A0h] BYREF
  char s[72]; // [rsp+70h] [rbp-60h] BYREF
  unsigned __int64 v12; // [rsp+B8h] [rbp-18h]

  v12 = __readfsqword(0x28u);
  qmemcpy(v10, "h_bO}EcDOR+G)uh(jl,vL", 21);
  printf("[*] Hello! Welcome to our Program!\nEnter the password to contiune:  ");
  __isoc99_scanf("%s", s);
  v7[0] = 16;
  v7[1] = 24;
  v8[0] = strlen(s) >> 1;
  v8[1] = strlen(s);
  v9[0] = 0;
  v9[1] = strlen(s) >> 1;
  for ( i = 0; i <= 1; ++i )
  {
    for ( j = v9[i]; j < v8[i]; ++j )
      v10[j + 32] = v7[i] ^ s[j];
  }
  for ( k = 0; k < strlen(v10) - 1; ++k )
  {
    if ( v10[k + 32] != v10[k] )
    {
      puts("[-] Wrong Password");
      exit(0);
    }
  }
  puts("[+] Successful Login");
  return 0;
}
```

Now, we need to figure out how the program works. Let's analyze the code step by step.

```c
qmemcpy(v10, "h_bO}EcDOR+G)uh(jl,vL", 21);
```

The program copies the string `h_bO}EcDOR+G)uh(jl,vL` to the v10 variable. 

```c
v7[0] = 16;
v7[1] = 24;
v8[0] = strlen(s) >> 1;
v8[1] = strlen(s);
v9[0] = 0;
v9[1] = strlen(s) >> 1;
```

The program initializes some variables. Realize that the flag length is probably 21 because the length of the string `h_bO}EcDOR+G)uh(jl,vL` is 21. So here the str(len(s)) is 21. 

```c
for ( i = 0; i <= 1; ++i )
{
  for ( j = v9[i]; j < v8[i]; ++j )
    v10[j + 32] = v7[i] ^ s[j];
}
```

The program xors the password with the values in the v7 array and stores the result in the v10 array. 

```c

for ( k = 0; k < strlen(v10) - 1; ++k )
{
  if ( v10[k + 32] != v10[k] )
  {
    puts("[-] Wrong Password");
    exit(0);
  }
}
puts("[+] Successful Login");
```

Finally, the program checks if the xored password is equal to the string `h_bO}EcDOR+G)uh(jl,vL`. If it is equal, the program prints the flag.

## Solution

Here we can figure out that the program XORs the password with the values in the v7 array. The values in the v7 array are 16 and 24. Also, for the first half of the password, the program xors it with 16 and for the second half, it xors it with `strlens(s) >> 1`. So, we can write a python script to find the password.

```python
from pwn import xor

encrypted_flag = "h_bO}EcDOR+G)uh(jl,vL"
first_part_key = 0x10
second_part_key = 0x18

second_part_len = (len(encrypted_flag) >> 1)  
first_part_len = len(encrypted_flag) - second_part_len - 1

first_part = encrypted_flag[:first_part_len]
second_part = encrypted_flag[second_part_len:]
 
flag = xor(first_part, first_part_key) + xor(second_part, second_part_key)
```

When we run the script, we get the password.

```bash
sarp@IdeaPad:~/Desktop$ python3 solve.py
b'xOr_mUsT_B3_1mp0rt4nT'
```



## Alternative Solution

Also, we can perform the XOR operation on the fly by editing the binary source code as follows.

```cpp
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <string>

int main(int argc, const char **argv, const char **envp) {
    int i; 
    int j; 
    int v7[2]; 
    int v8[2]; 
    int v9[2]; 
    char v10[64];
    char s[72];

    memcpy(v10, "h_bO}EcDOR+G)uh(jl,vL", 21);
    memcpy(s,   "h_bO}EcDOR+G)uh(jl,vL", 21);
    //printf("[*] Hello! Welcome to our Program!\nEnter the password to continue:  ");
    
 //scanf("%71s", s);

    v7[0] = 16;
    v7[1] = 24;
    v8[0] = strlen(s) >> 1;
    v8[1] = strlen(s);
    v9[0] = 0;
    v9[1] = strlen(s) >> 1;
    print("Flag: ")
    for (i = 0; i <= 1; ++i) {
        for (j = v9[i]; j < v8[i]; ++j) {
            v10[j + 32] = v7[i] ^   s[j];
            printf("%c", v10[j + 32]); 
        }
    }
    printf("\n");
   
}
```

When we compile and run the program, we get the flag.

```bash
sarp@IdeaPad:~/Desktop$ g++ -o xor xor.cpp && ./xor
Flag: xOr_mUsT_B3_1mp0rt4nT
```

## Conclusion

Cryptoversing was a decent reverse engineering challenge. Though it doesn't require advanced reverse engineering skills, understanding the XOR operation was crucial to solve the challenge. I hope you enjoyed the write-up. If you have any questions, feel free to ask me on [Twitter](https://twitter.com/sarperavci). 
