### Notes on everything I didn't know in chapter 2

- Nested functions are not supported
- A value-returning function that does not return a value will produce undefined behavior
- An **Unnamed Parameter** is a parameter that doesn't have a identifier
  - It is for when a function might need to take a parameter
    - Why? For example if we have a function that takes a parameter, ship that code and later find an optimization that doesn't need that parameter, we can just make the parameter unnamed so internally we know the parameter isn't necesarry but externally it keeps backwards compatibility.
    - Other cases too operator++ & operator-- (discussed in chapter 21), when you only need to know the type of the input while overloading a function, etc...

- Variables are destroyed in reverse order of creation at the end of the set of curly braces they are defined in

- Scope is a compile time property

-**Temporary Objects** aka **Anonymous Objects** are objects that have no scope and are destroyded before the next statement begins e.g.

  `std::cout << getValueFromUser() <<'\n'`
  - Since getValueFromUser isn't stored in any variable it is held in a temporary unnamed object till the statment is over

## 2.7
- A **Function Signature** is a description of the function's interface which include function name, and parameter types e.g add(int, int)

- A **Function Declaration** or **Function Prototype** is a complete declaration of the function which includes: function name, parameter types, return type, and (optionally) parameter names with a semicolon as it is a statement e.g int add(int a, int b);

- cpp compiles sequentially
  - functions must be defined before use or you must include a **Forward Declaration**, this is done by putting the function declaration with a seimicolon at the end above wherever it's called.
    - We often put these in header files !
    - important to tell the compiler about the existence of functions in other files`
    - Will get a linker error if you forward declare and call a func but don't actually provide a function body

- A **Declaration** tells the compiler of the existence of an indentifie like int add(int x, int y)

- A **Definition** is a declaration that actually implements (for functions and types) or instantiates for (for variables) the identifier
  - int x; would be a definition as it is declared and instantiated
q
- Declarations that aren't definitions are called **Pure Declarations**

- The **One Definition Rule** states:
  1. Within a file, each functions, variable, type, or template in a given scope can only have on definition
  2. Within a program, each function or variable in a given scope can only have one Definition. This rule exists because we can have more than 1 file in a program. Functions and vars not visible to linker are excluded from this rule.
  3. Types, templates, inline functions, and inline variables are allowed to have duplicate definitions in different files. Explained more later

- Overloaded functions count as distinct functions and thus different definitions.

## 2.8
- compiler compiles each file independentaly, one file doesn't know contents of others unless specified
  - This limited visibility is intentional since: it allows source files to be compiled in any order, when we change a source file only that file needs to be recompiled, and it reduces the possibility of naming conflicts

- if you have a main.cpp and a add.cpp you can use add in main as long as you have a forward declaration in main and compile both files such as `g++ main.cpp add.cpp -o main`

- This is how hpp files work! Notice how hpp files don't include their corresponding cpp file, but the cpp file has to include the hpp file since it's treated like a forward declaration

## 2.9
- A **Scope Region** is an area of the source code where all the declared identifiers are considered distinct from names declared in other scopes.
  - The body of a function is an example of this, two identically named identifiers can exists in two different function bodies no problem

- A **Namescape** provides another type of scope region named **Namescape Scope** that allows you to declare and define names inside purely for disambiguation, primarly to get around rule 2 in the one definition rule.
  - A namespace may only contain declarations and definitions, executable statements are only allowed as part of a definitions
  - used to group related identifiers in large projects to avoid naming collisions

- In cpp, any name not defined in a class, function, or namespace is automatically part of the implicitly-defined **Global Namespace** aka **Global Scope**
  - identifiers declared in global scope are in scope from the point of declaration to the end of files
  - We should avoid defining variabes in global scope (explained more in chapter 7.8)

- the std namespace was originally designed to just be part of the global namespace but would've caused confusion with naming conflicts from user defined identifiers in the global namespace

- When using code in a namespace we must tell the compiler with the **Scope Resolution Operator (::)** by puting namespace:: before the identifier, e.g std::cout. This is best practice

- We can use the **Using Directive** to acces names in a namepsace without using the namespace prefix
  - however is poor practice because for example if we are `using namespace std` this means that ANY identifier that we defined may conflict with any identically identifier in std, killing the whole purpose of namespaces

- Curly braces are often used to delineate a scope region nested within another scope region. For example a function defined in the global scope uses curly braces to seperate the scope region of the function to the global scope

```
void foo(int x) //foo is defined in the global score, x is defined within scope of foo()
{ //braces used to delineate nested scoped region for foo()
  std::cout << x;
} //x goes out of scope here
```

##  2.10
- Prior to compilation cpp files go through a **Preprocessing** phase. A programmed called the **Preprocessor** makes various changes to the text of the code file.
  - It doesn't actually modify the original code files but rather temporarily in memory
  - Historically preprocessors used to be seperate from compilers
  - It mostly just does uninteresting things like stripping out comments but it has one important role which is processing directives such as #include

- When a preprocessor is done processing the result is a **Translation Unit** which is what is compiled by the compiler.

- The entire process of preprocessing, compiling, and linking is called **Translation**

- **Preprocessor Directives** aka **Directives** are instructions that start with # symbol and end with a new line (NOT a semicolon). They tell the preprocessor to perform certain text manipulation tasks.
  - proprocessor doesn't understand c++ syntax, directives have their own syntax

- Note: the using-directive statement is NOT a preprocessor directive. So while the term directive usually means a preprocessor directive, this is not always the case.

- When you #include a file, the preprocessor replaces the #include directive with the contents of the included files
  - Include is almost exclusively used with header files

- The #define directive can be used to create a macros. A **Macro** is a rule that defines how input text is converted to output text
  - There are two basic types of macros: object-like macros and function like macros
  - Function like macros act like functions and are useless in modern C++

- Object-like macros can be defined in one of two ways, by convention macro names are all uppercase, seperated by underscores.
```
#define INDENTIFIER
or
#define INDENTIFIER substitution_text
```

- Object-like macros with substitution text is kinda just like a global variable, not recommended in modern C++ since it is not type safe and is just text
  - They used to be used in C to assign names to literals
```
#include <iostream>
#define MY_NAME "Ojas"

int main() {
  std::cout << MY_NAME; //Just prints Ojas
}
```

- Object-like macros without substitution text replaces occurences of the identifier with nothing.
  - Mainly used for conditional compilation

- The **Conditional Compilation** preprocessor directive allows you to specify when something will and won't compile. There are a lot of condition compilation directives but the most used ones are #indef, #ifndef, #endif, and #if 0.
  - This is great for debugging, platform specific code, dev/test code, etc...

- #indef checks if an identifier has been defined via #define, if so the code between #ifdef and the matching #endif is compiled
```
#include <iostream>
#define PRINT_OJAS

int main() {
#ifdef PRINT_OJAS //If PRINT_OJAS is defined executes code till #endif
  std::cout << "OJAS\n;
#endif
  return 0;
}
```

- #if 0 means to not compile code between it and the matching #endif. Is a convient way to comment out code. You can temporarily re-enable the that block of code by changing 0 to 1
```
#include <iostream>
int main()
{
  std::cout << "Joe\n";

#if 0 // Don't compile anything starting here
  std::cout << "Bob\n";
  std::cout << "Steve\n";
#endif // until this point

  return 0;
}
```

- NOTE: Macro substitution doesn't occur when a macro identifier is used within the preprocessor command
```
#define FOO 9 // Here's a macro substitution

#ifdef FOO // This FOO does not get replaced with 9 because it’s part of another preprocessor directive
    std::cout << FOO << '\n'; // This FOO gets replaced with 9 because it's part of the normal code
#endif
```

- Directives are resolved before compilation, from top to bottom on a file-by-file basis. They are NOT part of the c++ language and are all in the global scope. This is why we don't indent as to avoid confusion

## 2.11
- **Header Files** (usually ending in .h or .hpp) are used to propogate forward declarations into a cpp .file

- When you #include a file, the content of the file is inserted at the point of inclusion
  - We use angle brackets < > when the code lives in system paths
  - We use quotes "" for own project files in current project directory

- A header file consists of two parts: a header guard and the conents of the header file, which sould be the forward declarations of all the identifiers

- [Diagram of how includes are working in compiler] (https://www.learncpp.com/images/CppTutorial/Section1/IncludeHeader.png)

- Avoid putting definitions in header files as to follow the one-defition rule
  - Since if we have two files in the same program include a header file that contains a definition, we will have 2 definitions of that code in the final compiled program
  - There are work around for this with header guards

- Source files should include their paired header

- Do not include .cpp files!
  - Can cause naming collisions between source files
  - Will make it hard to avoid one definition rule (ODR) issues
  - any change make to one cpp file will cause all the others to recompile as well which is slow
  - It isn't conventional
