## Project 5: Code Generation
**Overview**:
- Implement code-generation phase
	- Generate C++ code from an abstract syntax tree (AST)
	- Source-to-source compiler specifically, so generating C++ code from AST instead of raw machine code

- ie. translate:
```
void main(string[] args) {
	println("Hello world!");
}
```
- Into:
```
#include "defs.hpp"
#include "ref.hpp"
#include "array.hpp"
#include "library.hpp"
#include "expr.hpp"

namespace uc {
	// Forward type declarations

	// Forward function declarations

	UC_PRIMITIVE(void)
		UC_FUNCTION(main)(UC_ARRAY(UC_PRIMITIVE(string)) UC_VAR(args));
	// Full type definitions
	// Full function definitions
	UC_PRIMITIVE(void)
		UC_FUNCTION(main)(UC_ARRAY(UC_PRIMITIVE(string)) UC_VAR(args)) {
		UC_FUNCTION(println)("Hello world!"s);
	}
	
} // namespace uc
```