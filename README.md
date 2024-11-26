# High Level Programming Language and its Compiler

## Machine Language Interpreter

This is a machine language interpreter written in C, its purpose is to translate a designed machine language and run it on a virtual machine.

#### Abbreviation Table:
| Abbreviation  	| Description  									|
| ------------- 	| ----------------------------------------------|
|     **ACC**   	|Accumulator (register)							|
|     **IP**    	|Instruction Pointer (register)					|
|     **\<OP>**   	|Operation: from **0** to **9**							|
|     **OpCode**   	|Operation code: **sign_bit** + operation **\<op>**	|
|     **OpType**   	|Operands type: specifies the types of the operands	|
|     **\<OPD1>**   |Operand1: first 4 digits after **opCode**			|
|     **\<OPD2>**   |Operand2: first 4 digits after **\<OPD1>**		|
|     **\<SRC>**   	|Source address or literal similar to **\<OPD1>**			|
|     **\<DEST>**   |Destination address or literal, similar to **\<OPD2>**		|


### 1. Numeric Machine Language (ML) Design:
#### Syntax:
The syntax of this ML is the following:
` 
+/- <OP> <OPD1> <OPD2> <OpType>
`
- **Sign bit**: + or -
- **Operation**: **1** digit  - from **0** to **9**
- **OpCode**: combination of ***sign bit*** and ***operation***
- **Operand 1**: **4** digits - ranges from **0000** to **9999** (10,000 values)
- **Operand 2**: **4** digits - ranges from **0000** to **9999** (10,000 values)
- **Operands type**:**1** digits - describes whether the operands are ***Literals*** or ***Addresses*** (described in detail in **ML Specifications**)

- The instruction: ***+1 2345 6789 0*** could be described as follows:

| sign_bit  | operation |operand1  |operand2  | operands_type |
| --------- | ----------| ---------|----------|---------------|
| +         | 1         |     2345 |  6789    |    0          |

#### ML Specifications:

- This language considers ***\<OPD1>*** and ***\<OPD2>*** as a ***source*** and ***destination*** respectively.
- This language uses **2** registers: **IP** & **ACC**
- This language allows to store both **literals** and **addresses** in an instruction (using ***operands_type*** digit).

- The ***operands_type*** digit can be one of the following: **{ 0 , 1 , 2 , 3 , 7 , 8 , 9 }**
    | operands_type | operand1 |operand2  |
    | ------------  | ---------| ---------|
    | **0**         | Address  |  Address |
    | **1**         | Address  |  Literal |
    | **2**         | Literal  |  Address |
    | **3**         | Literal  |  Literal |
    
    - **7**  : special "operands_type" value. Signifies that we should assign the value in **ACC** to **operand2** (address).
    - **8**  : special "operands_type" value. Signifies that we should assign the value in **operand1** (address) to **ACC**.
    - **9**  : special "operands_type" value. Signifies that we should assign the value in **operand1** (literal) to **ACC**.
    
- Since we have a memory of **20,000** words we divided it into two parts:
    - ***Program section***: consists of **10,000** words (0 - 9999)
    - ***Data section***: consists of **10,000** words (0 - 9999)
    
- With this design, we can access all the memory addresses since we have **4 digits** in each operand.

**NOTES:** 
- The user should set the accumulator before using an instruction that requires a third operand.
- The result of all operations that use **two operands** store the result in the **ACC**.
- In order to store a value in an array, the user should:
    - Compute the destination address (**Base address** + **index**)
    - Store that address from **ACC** to a memory address
    - Use the corresponding function (**ATV** or **VTA** / **+6** or **-6**) to store a variable in that address, or store the content of that address in a variable.

#### Full OpCode table:
| OpCode | Description      | OpCode | Description    |
|--------|------------------|--------|----------------|
| +0     | assign           | -0     | (not used)     |
| +1     | +                | -1     | -              |
| +2     | x                | -2     | /              |
| +3     | square           | -3     | square root    |
| +4     | equals           | -4     | does not equal |
| +5     | greater or equal | -5     | less than      |
| +6     | z <- A[x]        | -6     | A[x] <- z      |
| +7     |       test, jump | -7     | label          |
| +8     | read             | -8     | print          |
| +9     | stop             | -9     | (not used)     |

#### Challenges:
- We found it a bit hard to design an instruction that gives us the possiblity to access all the memory cells (0 - 9999). However, we decided to make our operands 4 digits long and **put the 3rd operand**, if needed, in the **ACC**.

### 2. Interpreter:
#### Usage:
The executable interpreter should be compiled from the C source interpreter file "interpreter.c".  
The executable takes one or two command line arguments:
1. The ML source file to read instructions from.
2. The options. *(i.e: -v for verbose)*

***Compile source code:***
```
    >> gcc interpreter.c -lm -o interpreter
```
***Run executable:***
-  *For Linux/Unix Based OS:*
```
    >> ./interpreter <source_ML_file_name>
    OR
    >> ./interpreter <source_ML_file_name> -v

    i.e:  ./interpreter source.nml -v
```
- *For Windows OS:*
```
    >> interpreter <source_ML_file_name>
    OR
    >> interpreter <source_ML_file_name> -v

    i.e:  interpreter source.nml -v
```
***Options:***
```
    -v : VERBOSE, logs all executed instructions sequentially.
```

#### Explanation of ML operations:
- **+0**: Assign **operand1** to **operand2**
- **+1**: Add **operand1** to **operand2** and  store result in **ACC**
- **-1**: Subtract **operand1** from **operand2** and store result in **ACC**
- **+2**: Multiply **operand1** by **operand2** and store result in **ACC**
- **-2**: Divide **operand1** by **operand2** and store result in **ACC**
- **+3**: Square of **operand1** and store result in **operand2**
- **-3**: Square root of **operand1** and store result in **operand2**
- **+4**: If **operand1** equals **operand2**, put 1 in **ACC**
- **-4**: If **operand1** does not equal **operand2**, put 0 in **ACC**
- **+5**: If **operand1** is greater or equals to **operand2** then put 1 in **ACC**
- **-5**: If **operand1** is less than **operand2** then put 0 in **ACC**
- **+6**: **Operand1** is an address. We access the value inside of it and then put in the address of **operand2**.
- **-6**: **Operand1** is assigned to the address that is pointed at by the address of **operand2**.
- **+7**: Jump to the address in operand1 if the adress is **operand2** contains 1
- **-7**: Store instruction in the address pointed at by **operand2**.
- **+8**: Read input from user and put in the address pointed at by **operand2**.
- **-8**: Print the content of the address pointed at by **operand1**.
- **+9**: Stops the program from executing.


#### Interpreter implementation timeline:

- [x] Read, line by line, file containing Numeric Machine Language (ML).
- [x] Decode and tokenize each line as an ***instruction*** structure.
- [x] Load all ***instructions*** to RAM (***CODE_MEMORY***):
    - TODO: explanation
- [x] Execute the ***instructions*** in RAM one by one:
    - [x] Loop through ***CODE_MEMORY*** array.
    - [x] Fetch each decoded instruction.
    - [x] Execute instruction:
        - [x] Implement a function for each opCode.
        - [x] Implement a log function for each function. 
- [x] Add command line input arguments:
    - [x] Input source ML file as a command line argument.
    - [x] Input options as command line arguments (-v for verbose).


### 3. Symbolic Machine Language Design (AL):
#### Syntax
The syntax of this assembly language is the following:
` 
<OP> <type><OPD1> <type><OPD2>
`
- **Operation** could be one of the following: {"ASN",  "ASN", "ADD",  "SUB",  "MUL", "DIV", "SQR",
                            "SQRT", "EQL", "NEQL", "GOE",  "LT", "ATV", "VTA",
                            "JMP",  "LBL", "READ", "PRNT", "STOP"}
- **Type** describes the type of the next operand. "L" means the next operand is a literal. "A" means the next operand is an address.
- **Operand1**: 4 digits - ranges from 0000 to 9999 (10000 values)
- **Operand2**: 4 digits - ranges from 0000 to 9999 (10000 values)
- The instruction: ASN L2341 A1001 could be described as follows:

| operation  | type     |operand1  |type      | operand2      |
| ---------  | ---------| ---------|----------|---------------|
| ASN        | L        |     2341 |  A       |    1001       |

#### AL Specifications:
- This language is similar to assembly.
- It implements an easy and direct mapping between its syntax and ML syntax.
- This language will be translated into ML.

#### Full Operations table:
| AL Operation | ML Operation | Description   | AL Operation | ML Operation | Description    |
| ------ |--------|------------------| ------ |--------|----------------|
| ASN    | +0     | assign           |        | -0     | (not used)     |
| ADD    | +1     | +                | SUB    | -1     | -              |
| MUL    | +2     | x                | DIV    | -2     | /              |
| SQR    | +3     | square           | SQRT   | -3     | square root    |
| EQL    | +4     | equals           | NEQL   | -4     | does not equal |
| GOE    | +5     | greater or equal | LT     | -5     | less than      |
| ATV    | +6     | z <- A[x]        | VTA    | -6     | A[x] <- z      |
| JMP    | +7     | test, jump       | LBL    | -7     | label          |
| READ   | +8     | read             | PRNT   | -8     | print          |
| STOP   | +9     | stop             |        | -9     | (not used)     |

### 4.Assembler
#### Usage:
The assembler executable outputs a file named ***source.nml*** that contains the corresponding ML code to be interpreted.

***Compile source code:***
```
    >> gcc assembler.c -o assembler
```
***Run executable:***
-  *For Linux/Unix Based OS:*
```
    >> ./assembler <source_AL_file_name>

    i.e:  ./assembler source.al
```
- *For Windows OS:*
```
    >> assembler <source_AL_file_name>

    i.e:  assembler source.al
```
#### Explanation of AL operations:
- **ASN**: Assign **operand1** to **operand2**
- **ADD**: Add **operand1** to **operand2** and store result in **ACC**
- **SUB**: Subtract **operand1** from **operand2** and store result in **ACC**
- **MUL**: Multiply **operand1** by **operand2** and store result in **ACC**
- **DIV**: Divide **operand1** by **operand2** and store result in **ACC**
- **SQR**: Square of **operand1** and store result in **operand2**
- **SQRT**: Square root of **operand1** and store result in **operand2**
- **EQL**: If **operand1** equals **operand2**, put 1 in **ACC**
- **NEQL**: If **operand1** does not equal **operand2**, put 0 in **ACC**.
- **GOE**: If **operand1** is greater or equals to operand2 then put 1 in **ACC**.
- **LT**: If **operand1** is less than **operand2** then put 0 in **ACC**.
- **ATV**: **Operand1** is an address. We access the value inside of it and then put in the address of **operand2**.
- **VTA**: **Operand1** is assigned to the address that is pointed at by the address of **operand2**.
- **JMP**: Jump to the address in **operand1** if the adress is **operand2** contains 1.
- **LBL**: Store instruction in the address pointed at by **operand2**.
- **READ**: Read input from user and put in the address pointed at by **operand2**.
- **PRNT**: Print the content of the address pointed at by **operand1**.
- **STOP** Stops the program from executing.

**NOTES:** 
- The **#** character describes the start of a comment in assembly. The assembler will ignore lines that start with **#**
- If the operand2 in print instruction is 9999 regardless of its type, it means it is \n.
#### Assembler implementation timeline:
- [x] Add a simple complexity algorithm for testing:
    - Perform assignments and arithmetic operations.
    - Pseudo-code:
    ```python
        a = 10
        b = 20
        c = 4
        s = a + b
        sq = sqrt(c)
        res = s / sq
        print(res)
        print('\n')
    ```
- [x] Add a medium complexity algorithm for testing:
    - Find max in an array of 5 numbers (loop and if statement).
    - Pseudo-code:
    ```python
        arr = [342, 96, 5935, 436, 1220]
        maximum = 0
        counter = 0
        while(counter < 5):
            current = arr[counter]
            if(current > maximum):
                maximum = current
            counter = counter + 1
        print(maximum)
        print('\n')
    ```
- [x] Add a high complexity algorithm for testing:
    - Draw a triangle shape (nested loops), then use nested if statements to print a value.
    - Pseudo-code:
    ```python
        # Nested loops
        i = 0
        j = 0
        n = 10
        while (i < n):
            j = i
            while (j < n):
                print(0)
                j = j + 1
            print('\n')
            i = i+1

        # Nested if statements
        a = 15
        b = 5
        c = a + b
        if (c >= 15):
            d = c / 2
            if(d >= 5):
                print(1337)
        print('\n')
    ```

### Screenshots

### Simple problem:

<img width="512" alt="1" src="https://user-images.githubusercontent.com/54045588/112025849-e36c8680-8b35-11eb-91e7-d4e6d834c9c8.png">

### Medium problem:

<img width="512" alt="2" src="https://user-images.githubusercontent.com/54045588/112025971-0a2abd00-8b36-11eb-8451-37e89cbb6be4.png">

### Complex problem:

<img width="512" alt="3" src="https://user-images.githubusercontent.com/54045588/112026020-144cbb80-8b36-11eb-9d53-491260e3a85f.png">

## Language Design

The language is a bit different from common programming languages, as it is based on the Italian language (all reserved words are in Italian). It is a bit inspired from some common (C and Java), and less common (Golang and Pascal) programming languages.  
✓ The main program starts after the “inizio” reserved word, and everything before it is treated as a user-defined function.  
✓ “:=” is the assignment operator since the common “=” is often considered as bad language design since it does not mean equality. It allows us to use “=” as our equality comparison operator instead of “==”.  
✓ Any compound statement can create a scope by declaring variables (including arrays) and/or constants. Compound statements include functions, selection statements, and loops. Each nested block can access its local variables along with variables in its enclosing outer blocks up to the main program.  
✓ Global variables (including arrays) and constants would have a special place in memory as they could be accessed through every part of the program. They are declared first before function definitions before the main program.  
✓ Functions can either be defined before the main program and called within it or called within other functions, or they can be defined inside the main program and called strictly within it and within other functions in the main program. Functions can only be defined inside other subprograms (including the main program) and not inside other statement blocks.  
✓ A standard library is always linked to any program instance written in the language (like in Python) without a need to explicitly call it.  
✓ A type for aggregated data types is defined instead of doing it the C or Java way (int variable[10]). This way, we will be able to distinguish it from other words in a faster and easier way.  
➔ The language is designated as HLPL and has no name as of yet.  
➔ The language is case sensitive.  

## Language Description

### Lexical Description
1. Handling Reserved Words:  
RegEx = costante | ind | int | car | vettore | func | := | non | e | o | se | altro | mentre | inizio | vuoto  
2. User-defined identifiers:  
RegEx = [a-zA-Z] ( _ | [0-9a-zA-Z])*  
3. Numeric literal syntax:  
RegEx = 0 | -? [1-9][0-9]*  
4. Character literal syntax:  
RegEx = ‘ [a-zA-Z0-9-!$%^&*()_+|~=`{}\[\]:";'<>?,.\/ ] ’ ⇒ Any printable character.  
5. String literal syntax:  
RegEx = “[a-zA-Z0-9-!$%^&*()_+|~=`{}\[\]:";'<>?,.\/ ]*”  
⇒ Considering empty string and spaces (the string includes all printable characters)  
6. Operators:  
RegEx = ( = | < | > | + | non | e | o | <= | >= | - | * | / )  
7. Punctuation:  
RegEx = (. | , | : | { | } | ( | ))  
8. Comments:  
RegEx = #[a-zA-Z0-9-!$%^&*()_+|~=`{}\[\]:";'<>?,.\/ ]* ⇒ ⇒ After # all printable characters are expected.  
9. Whitespace: RegEx = \s  
⇒ \s is the whitespace character. Space, tab, and new line are all considered whitespace.  

![Syntactic Description (CFG in EBNF)](https://github.com/benseddikismail/hlpl-and-its-compiler/blob/main/img/cfg.jpg)

➔  Since comments can be included anywhere in the program, as long as they are not followed by any statement in the same line, they are not expressed in each rule for the grammar to remain readable.  
➔  It is certainly implied that variable names and function names ought not be similar to reserved words nomenclature.

## Lexer
The lexer is separated into different classes in order to make it more readable and easier to scale. The Lexer class implements the most important functions; `getChar()` and `make_tokens()` among other helpful functions. `getChar` function gets the next character from the input stream and places it into the current_char class variable. Then, depending on the current char, the appropriate function that takes care of constructing the token is called. The next important class is the Position class which takes care of keeping track of the current line and current column. Generally, the lexer reads character by character and constructs a token based on the current stream of characters. Once the token is constructed, it is appended to an array of tokens that stores all the tokens from the input stream.

<img width="512" alt="lexer" src="https://github.com/benseddikismail/hlpl-and-its-compiler/blob/main/img/lexer.jpg">

## Parser and Static Semantics

### LL(1) Recursive Descent Program Acceptor

The CFG passes the pairwise disjointness test for every rule:  
FIRST(<program>) = {ID, CONST_TYPE, VEC_TYPE, VEC_CHAR, INIT}  
FIRST(<function>) = {FUNC_TYPE}  
FIRST(<function_call>) = {ID}  
FIRST(<function_body>) = {ID, CONST_TYPE, VEC_TYPE, FUNC_TYPE, IF, WHILE}  
FIRST(<body>) = {ID, CONST_TYPE, VEC_TYPE, IF, WHILE}  
FIRST(<statement>) = {IF, WHILE, ID}  
FIRST(<assignment>) = {ID}  
FIRST(<arithmetic_expression>) = {INT, ID}  
FIRST(<selection_statement>) = {IF}  
FIRST(<loop>) = {WHILE}  
FIRST(<condition>) = {INT, ID, CHAR, VEC_CHAR}  
FIRST(<argument_declaration>) = {VOID_TYPE, ID}  
FIRST(<var_declaration>) = {ID, CONST_TYPE, VEC_TYPE}  
FIRST(<constant>) = {CONST_TYPE}  
FIRST(<array_declaration>) = {VEC_TYPE}  
FIRST(<array_use>) = {ID}  
FIRST(<comment>) = {COMMENT}  
ID ➔ variable name/function name/array name:  
• The distinction between a variable declaration and a function when called can be done through the left parenthesis ‘(’  
• The distinction between variable declaration and an assignment can be done through ‘:=’  
• The distinction between variable declaration and an assignment can be done through ‘:=’ - arrays can be distinguished through ‘[’  
• Variables/functions/arrays cannot take the name of a reserved word  
The grammar is also not left recursive as it’s defined using EBNF.  

### Parsing and Statics Semantics
After the lexer outputs tokens (to tokens.txt) along with the actual source values to which they are associated, the parser, implemented in C, reads the tokens and mirrors the grammar to check whether the syntax is correct. Using subprograms for every non-terminal, the parser assesses whether a token sequence is acceptable or not based on the grammar.
It also registers declared variables into the symbol table (symbol_table.txt) so to evaluate static semantics. The symbol table records and identifier’s name, type, and value (in case it is assigned to a value).  
For static semantics evaluation, the symbol table would be used to make sure that any identifier used through the program is defined and would also be used for type checking.

<img width="512" alt="syntax" src="https://github.com/benseddikismail/hlpl-and-its-compiler/blob/main/img/syntax.jpg">

### Static Scoping
Static scoping, or lexical scoping is implemented in the following way: each time a function or nameless block is defined, we keep track of its environment, meaning whether that block was declared inside another block (such as a function). Each function will keep track of a “parent” which will be the block it is defined within. Once a reference to a variable is made inside a block, we search within the activation record of that block, if the variable is not found, we jump to the activation record of the parent block (which we are keeping track of) and search there. This recursive process is carried on until the variable is found, or an error occurs. Therefore, each time a block is declared in our program, an activation record is created for it, including its local variables, as well as a reference to its parent block. Identifying a parent block in our case can be done by setting every function within a block’s curly brackets as children to that block. Once done, the variable lookup is an easy task, we only need to recursively visit parents until we find the variable or until no parent is left (in which case we raise an error). In the AL code, once a reference is made to a local variable, it is used normally, however, once there is a non-local reference we visit the parents based on their references from each block.
### Dynamic Scoping
Dynamic Scoping is implemented as follows: a call stack is created and tracked, which will be a basic stack data structure that will store the activation record of each and every function called. Therefore, the parameters passed, the local variables and the return address of the caller of each specific function will be stored in LIFO manner. The AL code therefore needs to create a stack (implemented using an array) and keep track of its top. The environment of each block is stored in there in a LIFO way, meaning that the callee is pushed right after the caller on the stack. Once a local reference is made, we check the top of the stack and use that variable.
On the other hand, if a non-local reference is made, we traverse the stack from top to bottom until reaching the variable we are looking for or until reaching the bottom and raising an error. The AL code will need a variable to keep track of the top of the stack, and another variable to traverse the stack (the array) from top to bottom.
