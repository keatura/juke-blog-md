https://drive.google.com/drive/folders/143DtdyFFToeSkwvxlVE_16QQTQV6a1Yg

**Up to Slide 14 in Lecture**

#Lecture-20
#Macros
## Metaprogramming
#D/Metaprogramming
- **Metaprogramming:** Writing computer programs that operate on other computer programs
	- Compilers, program analyzers, code generators, etc
- **Code Generation**
	- Using language facilities to generate code on the fly, ie. the C++ macro system
	- AKA Automatic Programming

#E/Python-Scheme-Generator
**Example: Generating Scheme Functions in Python**
- Python function to generate scheme combinations for car, cdr functions
```
import itertools

for i in range(2, 5):
	for seq in itertools.product(('a','d'), repeat=i):
		print(defun(''.join(seq)))

// this will print different definitions for functions like 
// cadar = car cdr car andsoforth
```

## Macros
#D/Macro
- **Macro:** A rule that translates an input sequence/code into some replacement output sequence
	- #D/Macro-Expansion
	- **Macro Expansion:** The replacement process upon applying a macro
		- Usually implemented as a preprocessing step, something that essentially occurs before both lexical AND syntactic analysis
			- Also sometimes implemented as part of later analysis, such as during syntax analysis
			- CPP = C preprocessor is the most widely used, comes already integrated into C, C++

#E/Swap-Macro-Naive
**Swap Macro Example (Naive)**
```
#define SWAP (a, b) do { \
	auto tmp = b; \
	b = a; \
	a = tmp; \
} while (false)

// Backslash denotes that it continues
// do/while loop allows us to write the entire code body in an area that
// normally requires a single statement. The do/while essentially pretends
// to be a singular statement. Executes exactly once because of do/while shit
```
 - This will expand to:
```
do { auto tmp = y; y = x; x = tmp; } while (false);
```
