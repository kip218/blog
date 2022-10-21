---
layout: post
title: "Cryptanalysis of a Class of Ciphers"
excerpt: "Applied Cryptography: Project 1"
---

# Applied Cryptography: Project 1

For my Applied Cryptography class for the Fall 2022 semester, we were given a cipher algorithm that we had to analyze and attempt to break given the ciphertext.

PDF of our original report can be found [here](https://github.com/kip218/applied-crypto-project-1/blob/main/HuangParkSon-report.pdf){:target="_blank"}

Code used in this writeup can be found [here](https://github.com/kip218/applied-crypto-project-1){:target="_blank"}



&nbsp;
## 1. Introduction

### 1.1 Team Members

Team members for this project are David Son, Elaina Huang, and Kang In Park. All team members contributed to brainstorming potential solutions for breaking the cipher and assisted to write the report. Each person explored several known cipher attacks and as a group we discussed whether it could be applied to this particular cipher. 

In our final algorithm, Kang In implemented the Index of Coincidence calculation for deriving most likely key length as well as the convenience functions in `utils.py`. David Son filled in the frequency analysis using the chi-squared test.  Elaina Huang developed tools to assist the decryption process and attempted solutions using methods like N-gram frequency.

&nbsp;
### 1.2 The Encryption Algorithm

Here is our understanding of the cipher described by the project.

The text space is restricted to be in the set of {space, ‘a’, ‘b’, ‘c’, …, ‘z’} a total of 27 different elements, with no symbol (e.g. comma, period, etc.) presented. The key space is restricted to be in the set of {0, 1, 2, …, 26} a total of 27 different elements with the key length restriction in the range of [1, 24]. 

With the above conditions set, now we can further elaborate on the algorithm scheme.

The encryption algorithm takes in 3 parameters: a string of plaintext, a key, and lastly, a preset probability value. The encryption process will be using the text from the text space as defined above, to substitute any characters in the plain text. For example, an alphabet can be replaced with a space, and a space char can be replaced with an alphabet.

The method to encrypt is as follows:

Preset a probability value with a real number from the range of (0, 1). This value will later be used to compare with another randomly generated value (note, this value will be generated as a real number in the set of [0, 1].) to diverge the branched conditions.

Then, we will start encrypting every letter individually by using a loop that only terminates when it reaches the last element in the plain text. Inside the loop, there are two branched conditions to strengthen the security of ciphertext.

If the conditions “preset probability < randomly generated value” and “randomly generated value <= 1” are met, we will randomly choose a number from the list of keys to shift the plain text char into a cipher text char.

If the conditions “randomly generated value >= 0” and “randomly generated vale <= preset probability” are met, we will randomly choose a character from the text space as the noise to add to ciphertext; this procedure will eventually result in a longer length of output than its original length plaintext, thus satisfies the perfect secrecy.

The way to shift plaintext char is also based on the text space as defined above. Here is an indexed example of text space, ciphertxt_space = { (0, space), (1, ‘a’), (2, ‘b’), (3, ‘c’), ….., (26, ‘z’) }. The encryption algorithm shifts character based on this order.

Encryption scheme coded in python:

```python
#encrypt using project description
def encrypt(plaintext, key):
	prob_random = 0.00
	i = 0
	ciphertext = ''
	while i < len(plaintext):
		coin = random.random()
		if coin < prob_random:
			ciphertext += random.choice(text_space)
		else:
			k = random.choice(key)
			ciphertext += shift(plaintext[i], k)
			i += 1
	return ciphertext
```


&nbsp;
## 2. Cryptanalysis Approach

Our initial intuition was to use frequency analysis as the cipher seemed like a polyalphabetic cipher. However, the project description states that the key scheduling algorithm is unknown, and it does not necessarily rotate through the key in a consistent manner. This makes the project cipher more akin to a running key cipher as the final key length is as long as the message length.

Here it is important to note that the “final” key length is different from the key length ‘t’ as mentioned in the project description. Because we are operating without knowledge of the key scheduling algorithm, even if t=3 and the key is [1, 2, 3], the key does not repeat in an orderly fashion like so:

<p align="center"> 1 → 2 → 3 → 1 → 2 → 3… </p>

Instead, the key may end up being:

<p align="center"> 3 → 3 → 2 → 3 → 1 → 3… </p>

Since the key scheduling algorithm is unknown to the attacker, the algorithm might as well just be random. Therefore, the “final” key length in this project denotes the actual length of the key values used to encrypt the individual plaintext characters, and thus the “final” key length is the same as the plaintext length L.

This constraint created a lot of problems for us when trying to brainstorm ideas to crack the cipher. We explored many well known cipher attacks as well as individual ideas that at first seemed promising. These attempts will be outlined in the sections below.

These constraints on top of the random ciphertext factor made it impossible to find a reliable way to crack the cipher with the information available to the attacker. After exploring a number of potential solutions, we decided to build up from less constraints and work our way to the top.

Assuming that the key scheduling algorithm rotates through the key in a repeating fashion (thus making it a polyalphabetic cipher), and starting out with no randomness factor, we went with our initial intuition of using frequency analysis with index of coincidence to find the key length and chi squared test to find the key. Details will be provided in sections below.

&nbsp;
### 2.1 Attempted Solutions

In our research and discussion, we looked at many different known cipher attacks to attempt to tackle the cipher described by the project. While we came up with many ideas and potential solutions, when we examined the project constraints and adhered to it strictly, we could not find a solution that would reliably crack the cipher. Our attempts revolved around trying to identify the class of cipher that this particular project cipher belongs to, and whether there are known cipher attacks against these types of ciphers. The biggest challenge was the fact that the “final” key length was as long as the plaintext length. This made the project cipher more akin to a running key cipher or somewhat of a one time pad for shift ciphers (although the possibility of key values is limited to maximum 24 and not the full message space of 27 symbols). These ciphers are not crackable unless part of the key is exposed or the same key is used to encrypt several plaintext messages. None of these conditions existed for our project.

&nbsp;
#### Hill Climbing

One method we considered was the hill climbing algorithm, which can be used to break substitution ciphers. The way it works is the algorithm first generates a random key, also known as the “parent” key. The algorithm then attempts to decipher the ciphertext using this random key, which produces a sample plaintext. The algorithm then uses a fitness test, which scores the text based on how similar it is to english text. The algorithm continues generating random keys and comparing the fitness score to the highest scoring plaintext, keeping the plaintext that has a higher score. It repeats this pattern unless a higher score hasn’t been found for the past 1000 iterations. The reason this method caught our attention is because our dictionary texts contain english words and are fairly long at 600 characters, which would make it easier to utilize the fitness test. However, the number of possible keys in a substitution cipher is 26! (each alphabet in plaintext is mapped to another alphabet in ciphertext). The number of possible key variations in our project is t^L, which is at maximum 24^600. This makes it impossible to yield any meaningful results just by generating random variations of the key.

&nbsp;
#### Differential Cryptanalysis

Another method we considered was differential cryptanalysis. However, since we have no access to an encryption oracle to test different inputs, we cannot do this. There is no way for us to produce a ciphertext with the same algorithm, as the key scheduling algorithm is unknown to us. The only information we are given is the ciphertext and 5 candidate plaintexts, which we have no way of knowing whether the ciphertext given was actually from those 5 candidate plaintexts. The project description states that our decryption algorithm does not know whether it is looking at a ciphertext from test1 or test2. As mentioned above, there is no way for us the produce another ciphertext using the same algorithm, as the cipher described in the project is more akin to a running key cipher or a one time pad cipher for shift ciphers where the key is unique for every encryption.

&nbsp;
#### N-gram Frequency

Other methods in the frequency analysis category were the first that came to our minds. Methods like N-gram frequency and other analysis strategies that we eventually opted for such as Index of Coincidence and Chi squared test would not work if we were strictly adhering to the cipher constraints as outlined in the project. This is because the initialized key is not repeated, and so any kind of pattern matching or frequency analysis ends up having more noise than meaningful data. When we analyzed the sample ciphertexts created from our encryption algorithm, the frequency stats were distributed very evenly in all the characters. In the end we modified some specifications in the project description and ended up using frequency analysis with Index of Coincidence and Chi Squared test.

&nbsp;
#### Frequency Amplification & Fingerprinting

Another fun attempt that we tried was “frequency amplification”. This was not a method we found anywhere online, but just started as an idea for analyzing the frequency of characters and potentially “fingerprinting” the candidate plaintexts with their unique letter frequency distribution. The idea came from the fact that the randomly sprinkled ciphertext characters do not largely affect the frequency of the letters (at least when probability is 0.05). The most problematic effect it has is that it messes up the matching for plaintext[i] to ciphertext[i]. This makes it so that any kind of stream-based attack that attempts to run through both the plaintext and ciphertext is futile. However, the frequency distribution was relatively intact. With 0.05 probability for random ciphertext, the frequency distribution for text as long as 600 characters would only have about 30 anomalies. If we take into account that these anomalies are also random and should be relatively evenly distributed among the 27 potential characters, it is almost negligible when it comes to the whole “frequency fingerprint” idea.

The idea was to “amplify” the frequency distribution of our candidate plaintexts by encrypting them over multiple times, in hopes that the unique composition of those plaintexts would eventually lead to a consistent and distinguishable distribution of letter frequencies in their ciphertexts. While it was a fun idea, and we had somewhat of a success fingerprinting the plaintexts if the key was the same, this could not work for two reasons. First, the amplified frequency distribution for plaintexts with different keys was not consistent. It was only consistent if the key was the same, meaning that the randomness of the key scheduling algorithm or the random ciphertext characters could be overcome if amplified, but the sheer size of potential key values (24^600) could not be overcome. Furthermore, in reality, the decryption algorithm will only be given a ciphertext of length 600 + r. This sample is not big enough to reliably match its frequency distribution to the candidate fingerprints. While it was a fun attempt, this approach could not work for these reasons. The code for this can be seen in `amplify.py`.


&nbsp;
## 3. Explanation of Solution Algorithm

The approach we chose is frequency analysis. Assuming that the key is repeated, it is possible to derive the most likely key length using the Index of Coincidence. The Index of Coincidence is a way to calculate overlap of identical characters appearing in text. The IOC is typically greater in normal english text than in random ciphertext. Calculating the IOC is an easy way to detect monoalphabetic ciphers, since the IOC of the encrypted ciphertext will be the same as the english plaintext, indicating a single byte key used to encrypt the plaintext. 

&nbsp;
### Step 1: Guessing the key length using Index of Coincidence

In our polyalphabetic cipher, we first assume different key lengths from 1 to 24. The basis for using the Index of Coincidence is that the character in ciphertext[i] will be offset by the same amount as the character in ciphertext[i+t] since the same key value was used to encrypt it.

If we assumed that the key length was 5, we break the ciphertext into 5 groups:

<p align="center"> ciphertext[0], ciphertext[5], ciphertext[10] … </p>

go in the same group and

<p align="center"> ciphertext[1], ciphertext[6], ciphertext[11] … </p>

go in the same group. In the end we will have 5 groups, which we can treat as separate monoalphabetic ciphers. If these groups give a high IOC value, meaning they are close to a monoalphabetic cipher, which means that we have found the right key length. It should be noted that the most likely key length may come out to be a multiple of the actual key length.

&nbsp;\\
Code to break the ciphertext into n groups:
```python
#group into n groups, assuming key length is n
def group_into_n(ciphertext, n):
	groups = {i:'' for i in range(n)}
	for i in range(len(ciphertext)):
		group = i % n
		groups[group] += ciphertext[i]
	return groups
```

&nbsp;\\
Code to calculate Index of Coincidence:
```python
#get index of coincidence
def get_IOC(text, freq):
	total = 0
	for n in freq.values():
		total += n * (n - 1)
	N = len(text)
	if N <= 2:
		return 0
	total = float(total) / ((N * (N - 1)))
	return total
```

&nbsp;\\
Combining the two functions above, we can guess the most likely key length:
```python
#guess key length (try up to n length keys)
#returns list of tuples, ordered from most likely length to least
def guess_key_length(ciphertext, n):
	guess = defaultdict(int)
	for i in range(1, n+1):
		groups = group_into_n(ciphertext, i)
		IOCs = []
		for group in groups.values():
			freq = get_freq(group)
			IOCs.append(get_IOC(group, freq))
		IOC_avg = sum(IOCs) / i
		guess[i] = IOC_avg
	return order_freq(guess)
```

&nbsp;
### Step 2: Guessing the key using Chi Squared Test

From this point on, we can use the chi-squared test to find the key. A chi-squared test compares the frequency of a given dataset and compares it to its expected dataset, giving a smaller number the closer it is to its expected frequency. This test assumes the key length n and splits the ciphertext into groups of length n. Then, across the whole ciphertext, each group has their first letter substituted with every letter of the alphabet and has a chi-squared test performed for each substitution. The lowest score is assumed to be the first letter of the key. The same is tested on the second letter of each group, then the third, and so on, up to the nth letter of each group. The result of this operation is the proposed key.

```python
#guess the key in ciphertext based on proposed key length n
frequencies = {' ': 0.10445682451253482, 'a': 0.06963788300835655, 'b': 0.01894150417827298, 'c': 0.036211699164345405, 'd': 0.03147632311977716, 'e': 0.10891364902506964, 'f': 0.011142061281337047, 'g': 0.023676880222841225, 'h': 0.015598885793871866, 'i': 0.07855153203342619, 'j': 0.0025069637883008357, 'k': 0.008356545961002786, 'l': 0.0532033426183844, 'm': 0.021727019498607242, 'n': 0.0596100278551532, 'o': 0.050696378830083565, 'p': 0.02256267409470752, 'q': 0.0022284122562674096, 'r': 0.06573816155988858, 's': 0.07938718662952646, 't': 0.06100278551532033, 'u': 0.03064066852367688, 'v': 0.011977715877437326, 'w': 0.007799442896935933, 'x': 0.0036211699164345403, 'y': 0.017827298050139277, 'z': 0.0025069637883008357}
def guess_key(ciphertext, n):
	# Split into n different sections
	# sections = [(ciphertext[i:i+n]) for i in range(0, len(ciphertext), n)]
	str_freqs = [x * len(ciphertext) for x in frequencies.values()]
	key = ""

	for i in range(0, n):
		results = {}
		for char in ' abcdefghijklmnopqrstuvwxyz':
			new_str = ""
			new_str += ciphertext[0:i]
			for a in range(i, len(ciphertext), n):
				new_str += shift(ciphertext[a], -text_to_i[char])
				new_str += ciphertext[a+1:a+n]

			new_freqs = list(x[1] for x in get_alphabetical_freq(new_str))
			results[char] = chisquare(new_freqs, f_exp=str_freqs).statistic

		results = sorted(results.items(), key=lambda x: x[1])
		key += results[0][0]
	return key
```

Finally, with the ciphertext and the proposed key, the string can be decrypted using a simple polyalphabetic shift cipher decryption. For every character, subtract its value by its corresponding index in the key. The result is the original plaintext.


