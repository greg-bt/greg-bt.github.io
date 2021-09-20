---
layout: writeup
title: "Challenge #5 - Istanbul Bazaar"
ctf: Google CTF 2021 - Beginners Quest
nextWrite: "Google2021Beginners-06"
---
For this challenge we are given a file.
Running the file command we can see the file is a Zip archive.
The archive contains three files. `RoboCaller1337.py`, `robo_numbers_list.txt` and `secret.enc`.

Inspecting `robo_numbers_list.txt` we find it contains a 624 long list of phone numbers and the `secret.enc` file appears to contains just raw bytes.

Taking a look at `RoboCaller1337.py` we can see that it was most likely used to create the two other files. The script contains a function `encodeSecret` which looks like it uses random numbers to encrypt messages.

```python
def generateRandomNumbers():
    arr = []
    for i in range(624):
        arr.append(formatNumber(random.getrandbits(32) + (1<<31)))
    return arr

def encodeSecret(s):
    key = [random.getrandbits(8) for i in range(len(s))]
    return bytes([a^b for a,b in zip(key,list(s.encode()))])
```

Looks like were gonna be doing some reversing again ...

<h2>Pseudo-Random Theory</h2>

From the top of the python script we can tell that it uses the python 'random' library. With a quick bit of googling we find that the library actually uses a pseudo-random number generator based on the Mersenne Twister. The key here is that pseudo-random numbers are not random.

A little bit more digging and we find that `Lib/random.py` uses MT19937 which has a fatal flaw. After observing only 624 iterations there is sufficient enough data to predict all future iterations that the generator will produce. Funnily enough, 624 is the exact number of phone numbers in the `robo_numbers_list.txt` file.

<h2>Generator Reconstruction</h2>

We can extract the original 624 32bit numbers from the robo_numbers_list file by reversing the `generateRandomNumbers()` function in python.

```python
# Read numbers from robo_numbers_list file
f = open("robo_numbers_list.txt", "r")
numbers = [(int(num) - (1<<31)) for num in f.read().replace('-','').split("\n")]
f.close()
```


We can then use [kmyk's mersenne-twister-predictor library](https://github.com/kmyk/mersenne-twister-predictor "kmyk's mersenne-twister-predictor library") to reverse the random number generator which was likely also used to create the `secret.enc` file. By feeding 624 generated numbers into the MT19937Predictor it will be able to reconstruct the inner state of the random generator that was being used at the time the files were created. In python that looks like:

```python
from mt19937predictor import MT19937Predictor

# Reverse generateRandomNumbers() function for robo_numbers_list.txt
predictor = MT19937Predictor()
for num in numbers:
    predictor.setrandbits(num, 32)
```

<h2>Reversing ... again</h2>

Now that we have reconstructed the number generator we can get to work decoding the message contained in `secret.enc`. We can start by read the bytes from the file into python and then reversing the `encodeSecret()` function but instead of using the random number generator we will use our MT19937Predictor.

```python
flag= ""
# Read bytes from secret file
f = open("secret.enc", "rb")
secret = list(f.read())
f.close()

# Reverse encodeSecret() function for secret.enc
key = [predictor.getrandbits(8) for i in range(len(secret))]
for a,b in zip(key,secret):
    flag += (chr(a^b))

# Output the message
print(flag)
```

Running the code untwists the original encrypted message which happens to be our flag: `CTF{n3v3r_3ver_ev3r_use_r4nd0m}`