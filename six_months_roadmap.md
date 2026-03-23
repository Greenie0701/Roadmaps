
# MONTH 1 — Foundations and Lexical Analysis
 
**Goal:** Understand what a compiler does, build a working tokenizer,
implement a DFA from scratch.
 
---
 
## Week 1 — What Is a Compiler?
 
**Theme:** Build intuition before theory. Discover the problems before learning
the solutions.
 
### Watch
- M1: Compilation is Partial Evaluation of the Interpreter
- M2: Evolution of Compilers
- M3: Introduction to Optimizations (just watch, don't stress the details yet)
- M4: Outline of a Compiler
 
### Read
- [`chibicc/main.c`](https://github.com/rui314/chibicc/blob/main/main.c) — 
  the entry point. How does a compiler start? What does it call first?
- [`chibicc/chibicc.h`](https://github.com/rui314/chibicc/blob/main/chibicc.h)
  lines 1–80 — just the Token struct and TokenKind enum. Don't read further yet.
 
### Build — The Calculator (your first compiler)
 
Write a program in C that:
1. Takes a string like `"12 + 34 - 5"` as input
2. Breaks it into numbers and operators (your first tokenizer)
3. Evaluates the result and prints it
 
Rules:
- No libraries except `stdio.h` and `stdlib.h`
- Handle spaces between tokens
- Handle multi-digit numbers
- Handle errors: `"12 + abc"` should print an error, not crash
 
```c
// Expected behavior
$ ./calc "12 + 34 - 5"
41
$ ./calc "100 - 50 + 25"
75
$ ./calc "abc + 1"
error: unexpected character 'a'
```
 
Don't worry about multiplication or precedence yet. Just `+` and `-`.
 
### Checkpoint ✓
Your calculator correctly evaluates all of these:
- `"1 + 2"` → `3`
- `"100 - 50 + 25"` → `75`
- `"  10  +  20  "` → `30` (with spaces)
- `"abc"` → prints an error without crashing
 
---
 
## Week 2 — Lexical Analysis Theory
 
**Theme:** Give your Week 1 tokenizer a formal foundation.
 
### Watch
- M5: Introduction to Lexical Analysis
- M6: Regular Expressions
- M7: Lexical Analysis High-Level Algorithm
 
### Read
- [`chibicc/tokenize.c`](https://github.com/rui314/chibicc/blob/main/tokenize.c)
  — read it completely, top to bottom. It is ~300 lines. Take notes.
  - How does `tokenize()` work overall?
  - How does it handle whitespace?
  - How does it handle multi-character operators like `==` and `!=`?
  - How does `convert_keywords()` work and why is it a separate pass?
 
### Build — Extend your calculator's tokenizer
 
Add these token types to your Week 1 tokenizer:
- Multi-digit numbers (already done)
- Identifiers: `abc`, `x`, `myVar`
- Keywords: `if`, `while`, `return` (these are identifiers that get special treatment)
- Two-character operators: `==`, `!=`, `<=`, `>=`
- Single-character operators: `+`, `-`, `*`, `/`, `<`, `>`, `=`, `;`, `(`, `)`, `{`, `}`
 
Print each token with its type:
```
$ ./tokenizer "int x = 1 + 2;"
TK_KEYWORD  "int"
TK_IDENT    "x"
TK_PUNCT    "="
TK_NUM      1
TK_PUNCT    "+"
TK_NUM      2
TK_PUNCT    ";"
TK_EOF
```
 
Model your `Token` struct on chibicc's definition in `chibicc.h`.
 
### Checkpoint ✓
Your tokenizer correctly tokenizes:
- `"int x = 42;"` → 5 tokens + EOF
- `"if (x == 0) return 1;"` → 9 tokens + EOF
- `"while (x != 10) x = x + 1;"` → 12 tokens + EOF
 
---
 
## Week 3 — DFA and NFA Theory
 
**Theme:** Understand the automata that underlie every tokenizer.
 
### Watch
- M8: DFA and NFA
- M9: Software Implementation of DFA and NFA
- M10: Regular Expression to NFA
 
### Read
- Re-read [`chibicc/tokenize.c`](https://github.com/rui314/chibicc/blob/main/tokenize.c)
  now with DFA in mind
  - The main dispatch `if/else` chain in `tokenize()` is a DFA state machine
  - Each `if` branch is a state transition
  - Draw the DFA on paper as you read
 
### Build — Implement a DFA for number tokenizing
 
Write a standalone DFA that recognizes C numeric literals:
- Decimal integers: `42`, `100`
- Hexadecimal: `0xFF`, `0x1A`
- Octal: `0755`
- Reject invalid: `09` (invalid octal), `0xGG` (invalid hex)
 
Implement it as an explicit state machine with a `state` variable and a
`switch` statement — not as ad-hoc if/else chains.
 
```c
typedef enum { S_START, S_ZERO, S_DEC, S_HEX, S_OCT, S_ERR, S_DONE } State;
 
// Your DFA transition function
State transition(State s, char c);
```
 
### Checkpoint ✓
Your DFA correctly classifies:
- `"42"` → decimal, value 42
- `"0xFF"` → hex, value 255
- `"0755"` → octal, value 493
- `"09"` → error (invalid octal)
- `"0xGG"` → error (invalid hex digit)
 
---
 
## Week 4 — Build a Complete Lexer
 
**Theme:** Put the theory into a production-quality tokenizer.
 
### Watch
- Review M5–M7 if anything is unclear
- M15: Error Handling (watch just the first half — the lexer error part)
 
### Read
- [`chibicc/tokenize.c`](https://github.com/rui314/chibicc/blob/main/tokenize.c)
  one more time — this time focus on error reporting
  - `error_at()` — how does it point to the exact character in source?
  - `error_tok()` — how does it point to the exact token?
  - How does chibicc store source location in each Token?
 
### Build — Complete lexer with error reporting
 
Rewrite your tokenizer from scratch (don't copy — rebuild from memory using
chibicc as reference only when stuck). It must:
 
1. Tokenize all C tokens from Week 2
2. Store source location (line number + column) in each token
3. Report errors pointing to the exact character in source:
 
```
$ ./lexer "int x = @;"
error: ./test.c:1:9: invalid character '@'
int x = @;
        ^
```
 
Model your error reporting on chibicc's `error_at()` function.
 
### Checkpoint ✓
Your lexer handles all of these without crashing:
- Valid: `"int main() { return 42; }"`
- Error with good message: `"int x = @;"` — points to `@`
- Error with good message: `"int x = 1 $ 2;"` — points to `$`
- Stores correct line and column in every token
 
---
 
# MONTH 2 — Parsing
 
**Goal:** Build a recursive descent parser that produces a real AST.
Understand how C's grammar works.
 
---
 
## Week 5 — Context Free Grammars
 
**Theme:** The formal model behind every parser.
 
### Watch
- M11: Context Free Grammar
- M12: CFG Examples
- M13: Derivation Tree and Order
- M14: Ambiguity in CFG
 
### Read
- [`chibicc/chibicc.h`](https://github.com/rui314/chibicc/blob/main/chibicc.h)
  — the `Node` struct and `NodeKind` enum. These are the AST node types.
  Count how many node kinds exist. What does each one represent?
- [`chibicc/parse.c`](https://github.com/rui314/chibicc/blob/main/parse.c)
  lines 1–100 — just the helper functions at the top (`new_node`, `new_binary`,
  `new_num`, `expect`, `consume`)
 
### Build — Write a CFG for arithmetic
 
Write the grammar rules (on paper, not code) for arithmetic expressions that:
- Handles `+`, `-`, `*`, `/`
- Has correct operator precedence (multiplication before addition)
- Has no left recursion
- Has no ambiguity
 
Then verify: does your calculator from Week 1 implement this grammar? Where does
it fail (hint: it probably doesn't handle `*` and `/` with correct precedence)?
 
Fix your Week 1 calculator to correctly evaluate `2 + 3 * 4` as `14`, not `20`.
 
### Checkpoint ✓
Your calculator correctly handles:
- `"2 + 3 * 4"` → `14` (not 20)
- `"10 - 2 * 3"` → `4` (not 24)
- `"(2 + 3) * 4"` → `20`
- `"10 / 2 + 1"` → `6`
 
---
 
## Week 6 — Recursive Descent Parsing
 
**Theme:** Turn grammar rules directly into code.
 
### Watch
- M16: Abstract Syntax Tree
- M17: Recursive Descent Parsing Introduction
- M18: Recursive Descent Parsing Algorithm
 
### Read
- [`chibicc/parse.c`](https://github.com/rui314/chibicc/blob/main/parse.c)
  — the expression parsing functions only:
  - `expr()`, `assign()`, `equality()`, `relational()`, `add()`, `mul()`,
    `unary()`, `primary()`
  - For each function, write the grammar rule it implements
  - Trace `2 + 3 * 4` through all these function calls on paper
 
### Build — Recursive descent expression parser
 
Write a recursive descent parser for arithmetic + comparison expressions.
It must produce an AST (not evaluate directly — build a tree first).
 
```c
// Your AST node
typedef struct Node Node;
struct Node {
    NodeKind kind;   // ND_ADD, ND_SUB, ND_MUL, ND_DIV, ND_NUM, ND_EQ, ND_NEQ ...
    Node *lhs;       // left child
    Node *rhs;       // right child
    int val;         // only for ND_NUM
};
```
 
Then write an `eval(Node *node)` function that walks the tree and computes the result.
 
Grammar to implement:
```
expr       = equality
equality   = relational ("==" relational | "!=" relational)*
relational = add ("<" add | "<=" add | ">" add | ">=" add)*
add        = mul ("+" mul | "-" mul)*
mul        = unary ("*" unary | "/" unary)*
unary      = ("+" | "-")? primary
primary    = num | "(" expr ")"
```
 
### Checkpoint ✓
Your parser produces correct ASTs for:
- `"1 + 2 * 3"` → ADD(1, MUL(2, 3)) — multiplication is deeper in the tree
- `"(1 + 2) * 3"` → MUL(ADD(1, 2), 3)
- `"1 == 1"` → EQ(1, 1) → evaluates to 1 (true)
- `"1 != 1"` → NEQ(1, 1) → evaluates to 0 (false)
 
---
 
## Week 7 — Parsing Statements and Declarations
 
**Theme:** Extend from expressions to a full program structure.
 
### Watch
- M15: Error Handling (full module now)
- M19: Left Recursion
- M20: Predictive Parsing
 
### Read
- [`chibicc/parse.c`](https://github.com/rui314/chibicc/blob/main/parse.c)
  — the statement parsing functions:
  - `stmt()`, `compound_stmt()`, `declaration()`
  - How does `compound_stmt()` handle the scope of variables?
  - How does `stmt()` handle `if`, `while`, `for`, `return`?
 
### Build — Add statements to your parser
 
Extend your parser to handle:
- Variable declarations: `int x;` and `int x = 5;`
- Assignment: `x = 10;`
- `return` statement: `return x + 1;`
- `if/else`: `if (x == 0) return 1; else return 2;`
- `while`: `while (x < 10) x = x + 1;`
- Blocks: `{ stmt; stmt; stmt; }`
 
Your AST should have nodes for all of these. Add them to your `NodeKind` enum.
 
### Checkpoint ✓
Your parser produces a valid AST for:
```c
int x;
x = 5;
while (x < 10) {
    x = x + 1;
}
return x;
```
Print the AST as indented text and verify the structure makes sense.
 
---
 
## Week 8 — LL(1) Parsing Theory + Full Parser
 
**Theme:** Understand the theory behind your parser, build a complete one.
 
### Watch
- M21: LL1 Parsing
- M22: Constructing First Sets
- M23: Constructing Follow Sets
- M24: Constructing LL1 Parsing Table
 
### Read
- [`chibicc/parse.c`](https://github.com/rui314/chibicc/blob/main/parse.c)
  — the full file now, including `function()` and `program()`
  - How does chibicc parse a function definition?
  - How does it track local variables (the `locals` linked list)?
 
### Build — Add function support to your parser
 
Extend your parser to handle a complete C function:
```c
int add(int a, int b) {
    int c;
    c = a + b;
    return c;
}
```
 
Your AST should represent:
- Function name and return type
- Parameter list with names and types
- Function body as a compound statement
 
Also: compute First and Follow sets for your expression grammar on paper.
Verify your recursive descent parser makes the same decisions as the LL(1) table.
 
### Checkpoint ✓
Your parser handles a complete program with 2 functions:
```c
int double(int x) {
    return x * 2;
}
 
int main() {
    int a;
    a = 21;
    return double(a);
}
```
Prints the full AST without errors.
 
---
 
# MONTH 3 — Semantic Analysis and Code Generation
 
**Goal:** Add a type checker and symbol table. Then make your compiler emit
real x86-64 assembly that can run.
 
---
 
## Week 9 — Semantic Analysis Theory
 
**Theme:** Understanding meaning, not just structure.
 
### Watch
- M34: Semantic Analysis Introduction to Scope
- M35: Semantic Analysis as Recursive Descent over AST
- M36: Types
- M37: Type Formalism
 
### Read
- [`chibicc/chibicc.h`](https://github.com/rui314/chibicc/blob/main/chibicc.h)
  — the `Type` struct and `TypeKind` enum
  - What fields does `Type` have? (`kind`, `size`, `align`, `base`, `params`...)
  - How does chibicc represent a pointer type? An array type? A function type?
- [`chibicc/type.c`](https://github.com/rui314/chibicc/blob/main/type.c)
  — read it completely (~200 lines)
  - `add_type()` — how does it annotate every AST node?
  - `pointer_to()` — how does it build a pointer type?
 
### Build — Symbol table
 
Add a symbol table to your parser. Every variable declaration must:
1. Create a new `Symbol` entry with name, type, and stack offset
2. Be findable by name in the current scope
3. Go out of scope at the end of a `{}` block
 
```c
typedef struct Symbol Symbol;
struct Symbol {
    char *name;
    Type *type;
    int offset;       // position on the stack frame (negative from rbp)
    Symbol *next;     // linked list
};
```
 
Make `find_var(name)` walk the linked list and return the right symbol.
If not found in the current scope, walk to the parent scope.
 
### Checkpoint ✓
Your symbol table correctly handles:
```c
int x = 1;
{
    int x = 2;    // inner x shadows outer x
    // here: x == 2
}
// here: x == 1
```
Using a function, resolve `x` at each point and return the right value.
 
---
 
## Week 10 — Type Checking
 
**Theme:** Catching errors before running code.
 
### Watch
- M38: Soundness of a Type System
- M39: More Type Rules
- M39.1: Type Rules Revised for Reference Types
- M40: Type Environments
- M41: Type Environment and Symbol Table
 
### Read
- [`chibicc/parse.c`](https://github.com/rui314/chibicc/blob/main/parse.c)
  — search for `new_add()` and `new_sub()`
  - How does chibicc handle pointer + integer arithmetic?
  - How does it handle pointer - pointer?
  - What error does it emit for invalid combinations?
- [`chibicc/type.c`](https://github.com/rui314/chibicc/blob/main/type.c)
  — `is_integer()`, `is_pointer()` predicates
 
### Build — Type checker
 
Walk your AST after parsing and annotate every node with its type.
Enforce these rules:
- `+`, `-`, `*`, `/` require both operands to be integers
- `==`, `!=`, `<`, `>` work on integers and produce integers (0 or 1)
- Assignment requires compatible types
- `return` type must match function's declared return type
- Emit useful error messages: `"error: line 3: cannot add int and void*"`
 
### Checkpoint ✓
Your type checker:
- Accepts valid programs silently
- Rejects `int x = "hello";` with a type error
- Rejects `int x = 1 + 2; int y = x + "str";` with a type error
- Reports the correct line number in every error message
 
---
 
## Week 11 — Stack Machine and Code Generation Theory
 
**Theme:** From tree to instructions.
 
### Watch
- M46: Runtime Organization
- M47: Activations
- M48: Activation Records
- M49: Variables
- M50: Alignment
- M51: Stack Machine
- M52: Simulating a Stack Machine
 
### Read
- [`chibicc/codegen.c`](https://github.com/rui314/chibicc/blob/main/codegen.c)
  — just these functions to start:
  - `push()` and `pop()` — the entire stack machine is here
  - `assign_lvar_offsets()` — stack frame layout
  - `emit_text()` — the outer driver
- Run chibicc on a simple file and read the assembly:
  ```bash
  echo 'int main() { return 42; }' > t.c
  ./chibicc -S t.c && cat t.s
  ```
  Annotate every line: what concept from M47–52 does it implement?
 
### Build — Stack frame layout
 
Without emitting assembly yet, implement `assign_lvar_offsets()` for your compiler:
- Walk all local variables in a function
- Assign each a negative offset from `rbp` (e.g. first int → `-8`, second → `-16`)
- Round the total up to 16-byte alignment (required by x86-64 ABI)
- Store the total frame size so the prologue can do `sub rsp, N`
 
```c
// After this pass, every Obj should have:
obj->offset = -8;   // for the first int local variable
obj->offset = -16;  // for the second
// ... and so on
```
 
### Checkpoint ✓
For this function:
```c
int main() {
    int a;
    int b;
    char c;
    return 0;
}
```
Your `assign_lvar_offsets()` assigns:
- `a` → offset -8
- `b` → offset -16
- `c` → offset -17 (but total frame rounded to -32 for alignment)
 
---
 
## Week 12 — First Real Code Generation
 
**Theme:** Your compiler produces assembly that actually runs.
 
### Watch
- M53: Code Generation
- M54: Code Generation for Function Calls
- M55: Code Generation for Variables
- M56: Code Generation Example
 
### Read
- [`chibicc/codegen.c`](https://github.com/rui314/chibicc/blob/main/codegen.c)
  — `gen_expr()` fully. Trace through the `ND_ADD` case on paper:
  1. gen_expr(lhs) → result in rax
  2. push rax
  3. gen_expr(rhs) → result in rax
  4. pop rdi
  5. add rax, rdi
  → result in rax
 
### Build — Code generator for expressions and return
 
Write `gen_expr()` and `gen_stmt()` for your compiler. Start minimal — just
enough to compile this:
 
```c
int main() {
    return 21 + 21;
}
```
 
Your output `.s` file should be valid x86-64 AT&T syntax assembly. Test it:
 
```bash
./mycompiler test.c > test.s
gcc -o test test.s
./test; echo $?    # should print 42
```
 
Then extend to handle: variables, assignment, `if`, `while`, function calls.
 
### Checkpoint ✓ — The big one
 
Your compiler successfully compiles and runs this program:
 
```c
int add(int a, int b) {
    return a + b;
}
 
int main() {
    int x;
    int y;
    x = 20;
    y = 22;
    return add(x, y);
}
```
 
```bash
./mycompiler program.c > program.s
gcc -o program program.s
./program; echo $?
# must print: 42
```
 
**This is the first major milestone. You have built a complete compiler.**
 
---
 
# MONTH 4 — Optimizations and Dataflow Analysis
 
**Goal:** Understand how compilers improve code. Implement liveness analysis
and a register allocator.
 
---
 
## Week 13 — Why Optimization Matters
 
**Theme:** See the problem before learning the solution.
 
### Watch
- M57: Temporaries
- M72: Optimization Overview
- M73: Local Optimizations
 
### Read + Experiment
Run chibicc vs gcc -O2 on the same file and study every difference:
 
```bash
cat > opt_test.c << 'EOF'
int main() {
    int x = 2 + 3;        // constant folding candidate
    int y = x * 1;        // strength reduction candidate
    int z = 0;
    z = z;                 // dead store candidate
    return x + y + z;
}
EOF
 
./chibicc -S opt_test.c -o chibicc.s
gcc -O2 -S opt_test.c -o gcc.s
diff chibicc.s gcc.s
```
 
For every difference, write down which optimization GCC applied and which
course module teaches it. This is the best way to motivate Months 4–5.
 
### Build — Constant folding
 
Add a constant folding pass to your compiler. Walk the AST after parsing.
Any `ND_ADD`, `ND_SUB`, `ND_MUL`, `ND_DIV` node where both children are
`ND_NUM` → replace the whole subtree with a single `ND_NUM`:
 
```
ADD(NUM(2), NUM(3))  →  NUM(5)
MUL(NUM(4), NUM(1))  →  NUM(4)   (multiply by 1)
ADD(NUM(0), x)       →  x        (add 0)
```
 
### Checkpoint ✓
After constant folding, `int x = 2 + 3 * 4;` compiles to the same assembly
as `int x = 14;` — a single `mov` instruction, no arithmetic at runtime.
 
---
 
## Week 14 — Intermediate Representation
 
**Theme:** The language that optimizations operate on.
 
### Watch
- M68: Intermediate Representation
- M69: Three Address Code
- M70: Static Single Assignment IR
- M71: Phi Nodes
 
### Read
```bash
# See LLVM's IR for a function you know
cat > ssa_test.c << 'EOF'
int max(int a, int b) {
    if (a > b)
        return a;
    return b;
}
EOF
clang -O0 -emit-llvm -S ssa_test.c -o ssa_test.ll
cat ssa_test.ll
```
 
Find the phi node in the output. Understand why it's needed: `max` returns
either `a` or `b` depending on the branch taken. In SSA form, you can't just
say "return x" when x has two definitions. The phi node merges them.
 
### Build — Three-Address Code IR
 
Add an IR generation pass to your compiler. After parsing and type checking,
lower your AST to three-address code:
 
```
# Source: int c = a + b * 2;
t1 = b * 2
t2 = a + t1
c = t2
```
 
Each TAC instruction has: `result`, `op`, `arg1`, `arg2`.
Print the TAC for any input program.
 
### Checkpoint ✓
Your TAC generator produces correct three-address code for:
```c
int result = (a + b) * (c - d);
```
Output should be exactly 3 instructions (2 binary ops + 1 assignment).
 
---
 
## Week 15 — Dataflow Analysis Theory
 
**Theme:** Reasoning about the entire program, not just one statement.
 
### Watch
- M75: Global Optimization
- M76: DFA Values
- M77: Dataflow Analysis Algorithm
- M78: DFA Example
- M79: DFA Loop Example
 
### Read
- M80: DFA Value Orderings — watch this too
- Re-read your TAC output from Week 14 and manually apply the reaching
  definitions analysis (M90) on paper for a simple 10-instruction program
 
### Build — Basic blocks
 
Split your TAC program into basic blocks:
- A new block starts at: the first instruction, any label target, any
  instruction after a jump
- A block ends at: any jump instruction, any `return`
 
```c
typedef struct BasicBlock BasicBlock;
struct BasicBlock {
    Instruction **instrs;   // array of instructions
    int instr_count;
    BasicBlock **succs;     // successor blocks (for jumps)
    int succ_count;
    BasicBlock **preds;     // predecessor blocks
    int pred_count;
};
```
 
Build the control flow graph (CFG): connect blocks with edges based on jumps
and fall-through.
 
### Checkpoint ✓
For this function:
```c
int abs(int x) {
    if (x < 0)
        x = -x;
    return x;
}
```
Your CFG has exactly 3 basic blocks:
1. `if (x < 0)` — one instruction, two successors
2. `x = -x` — one instruction, one successor
3. `return x` — one instruction, no successors
 
---
 
## Week 16 — Liveness Analysis
 
**Theme:** Which variables are still needed?
 
### Watch
- M81: Liveness Property
- M82: Liveness DFA
- M83: Liveness DFA Example
- M84: More DFA Examples
- M89: Gen and Kill Sets
 
### Read
```
llvm/lib/CodeGen/LiveIntervals.cpp   (skim — see how LLVM implements liveness)
```
 
Even if you can't read all of it, look for how it computes live ranges and
how it stores them. Compare with the algorithm you just learned.
 
### Build — Liveness analysis
 
Implement the liveness dataflow equations over your CFG from Week 15:
 
```
USE[B]      = variables read before any write in block B
DEF[B]      = variables written in block B
 
LIVE_IN[B]  = USE[B] ∪ (LIVE_OUT[B] − DEF[B])
LIVE_OUT[B] = ∪ LIVE_IN[S]   for each successor S of B
```
 
Iterate until fixed point. At each program point, record the set of live variables.
 
Print the live-in and live-out sets for each basic block.
 
### Checkpoint ✓
For `int abs(int x)` from Week 15, your liveness analysis computes:
 
| Block | LIVE_IN | LIVE_OUT |
|---|---|---|
| Block 1 (if x < 0) | {x} | {x} |
| Block 2 (x = -x) | {x} | {x} |
| Block 3 (return x) | {x} | {} |
 
---
 
# MONTH 5 — Register Allocation and chibicc Deep Dive
 
**Goal:** Build a graph-coloring register allocator. Read and fully
understand all of chibicc.
 
---
 
## Week 17 — Register Allocation Theory
 
**Theme:** Assigning variables to physical registers.
 
### Watch
- M85: Register Allocation
- M86: Register Spilling
- M87: Limitations of Graph Colouring
 
### Read
- [`chibicc/codegen.c`](https://github.com/rui314/chibicc/blob/main/codegen.c)
  — understand why chibicc uses `push`/`pop` for everything instead of registers.
  It's simple and correct, but it's also slow. Every `push` is a memory write.
  Every `pop` is a memory read. Register allocation eliminates most of these.
 
### Build — Interference graph
 
Using your liveness analysis from Week 16:
 
1. **Build the interference graph:** Two variables `a` and `b` interfere if
   they are simultaneously live at any program point. Add an edge between them.
 
2. **Represent it as an adjacency list:**
   ```c
   typedef struct {
       char *var;
       char **neighbors;
       int neighbor_count;
       int color;    // assigned register, -1 = uncolored
   } IGNode;
   ```
 
3. **Print the interference graph** for the programs from Week 15–16.
 
### Checkpoint ✓
For a function with variables `a`, `b`, `c`, `d` where `a` and `b` are live
at the same time, and `c` and `d` are live at the same time but not with `a`/`b`:
- Graph has edges: a—b, c—d
- No edge between a and c (they don't interfere)
 
---
 
## Week 18 — Graph Coloring
 
**Theme:** Assign registers using graph coloring.
 
### Watch
- M85 again — focus on the Kempe's algorithm section
- M86 again — focus on spilling heuristics
 
### Build — Graph coloring register allocator
 
Implement graph coloring with k = 4 colors (4 available registers for now):
 
```
Algorithm (Kempe's):
1. If any node has fewer than k neighbors, push it on a stack and remove it
2. Repeat until graph is empty or only high-degree nodes remain
3. If a node remains (potential spill), select one and mark it for spilling
4. Pop nodes from the stack, assigning colors avoiding neighbors' colors
5. For spilled variables, insert load/store around each use
```
 
Map colors to physical registers:
- Color 0 → `rax`
- Color 1 → `rcx`
- Color 2 → `rdx`
- Color 3 → `rsi`
 
After coloring, update your code generator to use physical register names
instead of `push`/`pop`.
 
### Checkpoint ✓
For a function where you have proven (via liveness) that two variables are
never simultaneously live, your allocator assigns them the same register —
eliminating at least 2 push/pop pairs compared to chibicc's naive strategy.
 
---
 
## Week 19 — chibicc Complete Read-through
 
**Theme:** Read a real compiler from first line to last.
 
### No new course modules this week — focus entirely on chibicc.
 
### Read — Every file, in order
 
Work through the complete chibicc source. For each file, answer the questions:
 
**`main.c`**
- What options does chibicc accept?
- What is the call order from `main()` to assembly output?
 
**`tokenize.c`**
- You've read this before. This time: how does it handle string literals with
  escape sequences? How does it handle multi-line strings?
 
**`preprocess.c`**
- How does `#define` work?
- How does `#include` work?
- How does chibicc handle recursive macros?
 
**`parse.c`**
- How does chibicc parse struct definitions?
- How does it parse `typedef`?
- How does it handle function pointers?
 
**`type.c`**
- How does `add_type()` handle the case where a node already has a type?
- How does chibicc compute struct member offsets?
 
**`codegen.c`**
- How does chibicc emit code for a struct member access (`obj.field`)?
- How does it handle `a[i]` (array indexing)?
- How does it handle `*p` (pointer dereference)?
 
### Build — Write a summary
 
For each file, write a 5-line summary:
- What it does
- Its main entry point function
- Its 3 most important functions
- One thing that surprised you
- One question you still have
 
### Checkpoint ✓
You can explain, without looking at the code, what any of these functions
does: `tokenize()`, `find_var()`, `add_type()`, `gen_expr()`,
`assign_lvar_offsets()`.
 
---
 
## Week 20 — Peephole Optimizations + chibicc Extension
 
**Theme:** Improving the final output with local pattern matching.
 
### Watch
- M74: Peephole Optimizations
- M74.1: Examples of Peephole Optimizations
 
### Build — Two projects this week
 
**Project A: Peephole optimizer**
 
Write a post-pass over chibicc's assembly output that applies these rules:
 
| Pattern | Replacement | Saving |
|---|---|---|
| `push rax` then `pop rax` | delete both | 2 instructions |
| `push rax` then `pop rdi` | `mov rdi, rax` | 1 instruction |
| `mov rax, $0` | `xor rax, rax` | smaller encoding |
| `add rax, $0` | delete | 1 instruction |
| `imul rax, rax, $1` | delete | 1 instruction |
| `cmp rax, $0` then `je` | `test rax, rax` then `jz` | smaller, faster |
 
Measure: how many instructions does your optimizer remove on chibicc's own
`tcctest.c` equivalent?
 
**Project B: Add a new feature to chibicc**
 
Pick one feature chibicc doesn't have and add it:
- `do { ... } while (cond);` loop
- Comma operator: `a = (x = 1, x + 2);`
- `break` and `continue` in loops
- Ternary operator: `a ? b : c`
 
Follow the pattern: add NodeKind → handle in `parse.c` → handle in `codegen.c`.
Use `git diff` on chibicc commits as a guide.
 
### Checkpoint ✓
- Your peephole optimizer removes at least 15% of instructions on a test program
- Your new chibicc feature compiles and runs correctly on 3 test cases
 
---
 
# MONTH 6 — LLVM Introduction
 
**Goal:** Understand LLVM IR, write your first LLVM pass, navigate the
AArch64 backend (if on ARM), read real compiler infrastructure code.
 
---
 
## Week 21 — LLVM IR
 
**Theme:** The universal intermediate language.
 
### Watch
- M70: Static Single Assignment IR (re-watch with LLVM in mind)
- M71: Phi Nodes (re-watch)
 
### Setup: Build LLVM
 
```bash
git clone https://github.com/llvm/llvm-project.git
cd llvm-project && mkdir build && cd build
 
# Minimal build — only what you need
cmake -G Ninja ../llvm \
  -DCMAKE_BUILD_TYPE=Release \
  -DLLVM_TARGETS_TO_BUILD="X86;AArch64" \
  -DLLVM_ENABLE_PROJECTS="clang" \
  -DLLVM_ENABLE_ASSERTIONS=ON
 
ninja -j$(nproc)    # takes 30-60 min first time
```
 
### Read — LLVM IR by example
 
```bash
# See IR for programs you already know how to compile
cat > sum.c << 'EOF'
int sum(int a, int b) { return a + b; }
EOF
 
clang -O0 -emit-llvm -S sum.c -o sum_O0.ll && cat sum_O0.ll
clang -O2 -emit-llvm -S sum.c -o sum_O2.ll && cat sum_O2.ll
 
# See the phi node in action
cat > max.c << 'EOF'
int max(int a, int b) { return a > b ? a : b; }
EOF
clang -O1 -emit-llvm -S max.c -o max.ll && cat max.ll
# Find the phi node — which value does it merge?
```
 
For every construct in the IR, map it to:
- The chibicc AST node it came from
- The course module that explains it
 
**Reference:** https://llvm.org/docs/LangRef.html — bookmark this, you will use it constantly.
 
### Build — Hand-write LLVM IR
 
Without using clang, write LLVM IR by hand for this function:
 
```c
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```
 
Then run it:
```bash
lli factorial.ll    # interpret it
llc factorial.ll -o factorial.s && gcc factorial.s -o factorial && ./factorial
```
 
### Checkpoint ✓
Your hand-written `factorial.ll` computes `factorial(5) = 120` correctly
when run with `lli`.
 
---
 
## Week 22 — Writing Your First LLVM Pass
 
**Theme:** Modifying the compiler from inside.
 
### Read
- https://llvm.org/docs/WritingAnLLVMPass.html — the official guide
- `llvm/lib/Transforms/Utils/` — simple utility passes, good examples
  - `llvm/lib/Transforms/Utils/SimplifyCFG.cpp` — simplify control flow
  - Look for small, self-contained passes to read first
 
### Build — Three passes in increasing difficulty
 
**Pass 1: Instruction counter**
Count and print instructions per function. This is purely mechanical — get
the pass infrastructure working.
 
**Pass 2: Constant folder**
Find `add i32 <const1>, <const2>` instructions and replace them with the
computed constant. This is M73 in LLVM form.
 
```cpp
// In your pass:
for (auto &BB : F) {
    for (auto &I : BB) {
        if (auto *binOp = dyn_cast<BinaryOperator>(&I)) {
            if (binOp->getOpcode() == Instruction::Add) {
                auto *lhs = dyn_cast<ConstantInt>(binOp->getOperand(0));
                auto *rhs = dyn_cast<ConstantInt>(binOp->getOperand(1));
                if (lhs && rhs) {
                    // replace with constant
                }
            }
        }
    }
}
```
 
**Pass 3: Dead instruction eliminator**
Remove instructions whose result is never used. An instruction is dead if
`I.use_empty()` and it has no side effects (`!I.mayHaveSideEffects()`).
 
```bash
# Test your pass
opt -load-pass-plugin=./MyPass.so -passes="my-pass" input.ll -S -o output.ll
diff input.ll output.ll
```
 
### Checkpoint ✓
Your constant folding pass correctly simplifies:
```llvm
%1 = add i32 2, 3
%2 = mul i32 %1, 4
ret i32 %2
```
Into:
```llvm
ret i32 20
```
 
---
 
## Week 23 — LLVM Backend Overview
 
**Theme:** How IR becomes machine code.
 
### Watch
- M68: Intermediate Representation (re-watch — now you know what LLVM IR is)
 
### Read — The code generation pipeline
 
```
llvm/lib/CodeGen/SelectionDAG/     → instruction selection
llvm/lib/CodeGen/RegAllocGreedy.cpp → register allocation (you built this!)
llvm/lib/CodeGen/PrologEpilogInserter.cpp → stack frame (you built this too!)
```
 
Then pick your target architecture:
 
**If on x86-64:**
```
llvm/lib/Target/X86/X86ISelLowering.cpp    → how IR ops map to x86 instructions
llvm/lib/Target/X86/X86InstrSSE.td         → SSE instruction definitions
llvm/lib/Target/X86/X86CallingConv.td      → calling convention
```
 
**If on ARM / developing for ARM:**
```
llvm/lib/Target/AArch64/AArch64ISelLowering.cpp    → IR → AArch64 instruction mapping
llvm/lib/Target/AArch64/AArch64InstrNEON.td        → NEON instruction definitions
llvm/lib/Target/AArch64/AArch64CallingConvention.td → AAPCS64 ABI encoding
```
 
### Build — Trace a function through the full pipeline
 
```bash
# See every pass that runs on a function
llc -march=x86-64 -O2 -debug-pass=Structure sum.ll -o /dev/null 2>&1 | head -60
 
# See the SelectionDAG (requires graphviz)
llc -march=x86-64 -view-isel-dags sum.ll -o /dev/null
 
# See machine IR before and after register allocation
llc -march=x86-64 -O2 -print-after=greedy sum.ll -o /dev/null 2>&1 | head -80
```
 
Document the pipeline: what are the 10 most important passes between IR input
and assembly output? For each one, which course module does it correspond to?
 
### Checkpoint ✓
You can name 10 LLVM passes in order, explain what each one does, and map
each to a concept from the course.
 
---
 
## Week 24 — Real Optimization Passes in LLVM
 
**Theme:** Your course theory implemented at production scale.
 
### Watch
- M77: Dataflow Analysis Algorithm (re-watch with LLVM in mind)
- M85: Register Allocation (re-watch — now you know LLVM's version)
 
### Read — Production implementations of what you built
 
| What you built | LLVM's version | Location |
|---|---|---|
| Liveness analysis | `LiveVariables` | `llvm/lib/CodeGen/LiveVariables.cpp` |
| Interference graph | `LiveIntervals` | `llvm/lib/CodeGen/LiveIntervals.cpp` |
| Graph coloring allocator | `RegAllocGreedy` | `llvm/lib/CodeGen/RegAllocGreedy.cpp` |
| Constant folding | `ConstantFolding` | `llvm/lib/Analysis/ConstantFolding.cpp` |
| Dead code elimination | `DCE` | `llvm/lib/Transforms/Scalar/DCE.cpp` |
| Peephole optimizer | `PeepholeOptimizer` | `llvm/lib/CodeGen/PeepholeOptimizer.cpp` |
 
For each file: read the first 100 lines and the main entry function. You are
not reading to understand every detail — you are reading to confirm that you
recognize the algorithm from your own implementation.
 
### Build — Missed optimization finder
 
Write a script that:
1. Compiles a C file with `clang -O0 -emit-llvm -S`
2. Runs your constant folding pass
3. Runs `clang -O2 -emit-llvm -S` on the same file
4. Diffs the results and reports which optimizations your pass found vs missed
 
### Checkpoint ✓
You can open `RegAllocGreedy.cpp` and point to exactly where Kempe's algorithm
is implemented — and it matches the algorithm you built in Week 18.
 
---
 
## Week 25 — Finding Your First Contribution
 
**Theme:** Transition from student to contributor.
 
### Build — Your contribution pipeline
 
**Step 1: Subscribe to LLVM discourse**
- https://discourse.llvm.org — read "new issues" daily for 1 week
- Focus on: `backend:X86` or `backend:AArch64` + `missed-optimization` labels
 
**Step 2: Find a good first issue**
- https://github.com/llvm/llvm-project/contribute
- Filter by `good first issue`
- Look for: test additions, documentation fixes, small missed optimizations
 
**Step 3: Study a merged beginner contribution**
Find 3 recently merged first-time contributor PRs:
```
https://github.com/llvm/llvm-project/pulls?q=is%3Amerged+label%3A%22good+first+issue%22
```
For each: what was the change? How was the test written? How many review rounds?
 
**Step 4: Write your first test**
Before fixing anything, add a `FileCheck` test that demonstrates a missed
optimization. A test without a fix is called an "improvement opportunity" —
it's a valid, welcome contribution:
 
```bash
# LLVM test format
# RUN: llc -march=x86-64 -O2 < %s | FileCheck %s
# CHECK: imulq   (make sure this instruction appears)
# CHECK-NOT: pushq  (make sure this instruction does NOT appear)
```
 
### Checkpoint ✓
You have:
- Subscribed to LLVM discourse
- Identified 1 specific issue you could fix
- Written a `FileCheck` test that demonstrates the issue
- Opened a GitHub issue or commented on an existing one
 
---
 
## Week 26 — Final Week: Portfolio and Next Steps
 
**Theme:** Consolidate everything, make it visible, plan the next 6 months.
 
### Build — Your portfolio
 
**Project 1: Your compiler (the most important)**
 
Clean up and publish your compiler from Months 1–3. It should:
- Have a clear `README.md` explaining what it compiles and how to run it
- Have a `tests/` directory with at least 20 test programs
- Compile and run cleanly on a fresh machine
- Have a design doc explaining your architecture decisions
 
Name it something memorable. This is your primary portfolio piece.
 
**Project 2: Your optimization passes**
 
Package your peephole optimizer and liveness/register allocation work as a
standalone project. Show before/after assembly comparisons in the README.
 
**Project 3: Your LLVM pass**
 
Put your constant folding and DCE passes in a GitHub repo with build instructions
and test cases. Even tiny LLVM contributions signal to employers that you can
navigate the codebase.
 
### Final Checkpoint ✓ — The 6-month test
 
You can answer all of these from memory:
 
**Lexical Analysis**
- What is the difference between NFA and DFA?
- Why does chibicc run `convert_keywords()` as a separate pass?
 
**Parsing**
- What is left recursion and why does it break recursive descent parsers?
- How does operator precedence get encoded in a recursive descent grammar?
 
**Semantic Analysis**
- What is the difference between a type environment and a symbol table?
- Why does `pointer + integer` need special treatment in the type checker?
 
**Code Generation**
- What is an activation record and what does it contain?
- How does the stack machine strategy in chibicc avoid needing a register allocator?
 
**Optimizations**
- What is the liveness equation and what does it compute?
- Why can't you color the interference graph if it has a clique of size k+1?
 
**LLVM**
- What is SSA form and why does LLVM use it?
- What does a phi node do?
- What is the difference between IR-level and machine-level optimization passes?
 
---
 
## What Comes After 6 Months
 
You have now:
- Built a complete compiler from scratch
- Read and extended a real open source compiler (chibicc)
- Written LLVM passes
- Identified your first contribution opportunity
 
The next 6 months should focus on:
 
| Goal | Action |
|---|---|
| Get an LLVM contribution merged | Fix the issue from Week 25 |
| Go deeper on one phase | Pick: frontends (Clang/MLIR), backends (AArch64), or optimizations (loop opts) |
| Study the polyhedral framework | M125–M165 + Polly (https://polly.llvm.org) |
| Formal semantics | M166–M190 + Alive2 (https://github.com/AliveToolkit/alive2) |
| Apply for compiler roles | Your portfolio now supports it |
 
---
 
## Daily Schedule (Suggested)
 
| Time | Activity |
|---|---|
| 09:00 – 11:00 | Watch course modules (take notes, pause and re-watch) |
| 11:00 – 13:00 | Read source code (chibicc or LLVM — with a specific question) |
| 14:00 – 17:00 | Build — write code for the week's milestone |
| 17:00 – 18:00 | Godbolt.org — compile something simple, read the assembly |
| 18:00 – 18:30 | Review: what did you learn? what is still unclear? |
 
One day per week: **no new material**. Only fix bugs in what you've already built,
write tests, clean up code, and document your design decisions.
 
---
 
## Resources
 
| Resource | URL | When to use |
|---|---|---|
| chibicc | https://github.com/rui314/chibicc | Months 1–5 reference |
| chibicc companion book | https://www.sigbus.info/compilerbook | Read alongside chibicc |
| Godbolt Compiler Explorer | https://godbolt.org | Daily — read assembly output |
| LLVM project | https://github.com/llvm/llvm-project | Month 6+ |
| LLVM Language Reference | https://llvm.org/docs/LangRef.html | Month 6 IR reading |
| Writing an LLVM Pass | https://llvm.org/docs/WritingAnLLVMPass.html | Week 22 |
| LLVM Good First Issues | https://github.com/llvm/llvm-project/contribute | Week 25 |
| Engineering a Compiler (Cooper & Torczon) | Buy or borrow | Best textbook companion |
| Crafting Interpreters (Nystrom) | https://craftinginterpreters.com | Free — great supplement for Months 1–2 |
