# Compiler Design Course × chibicc Roadmap

A milestone-driven roadmap that maps all 190 course modules to the
[chibicc](https://github.com/rui314/chibicc) open-source C compiler by Rui Inoue.

> **Why chibicc?**
> chibicc was written specifically as a teaching compiler. Every git commit adds
> exactly one feature. The code is cleanly separated into phases that map 1-to-1
> with what the course teaches. There are no tricks, no global state hacks, and
> almost every function is short enough to read in one sitting.

---

## Repository Structure

```bash
git clone https://github.com/rui314/chibicc.git
cd chibicc
make
echo 'int main() { return 42; }' > test.c && ./chibicc test.c && ./a.out; echo $?
```

**Every source file has a single, clear responsibility:**

| File | Course Phase | Role |
|---|---|---|
| [`chibicc.h`](https://github.com/rui314/chibicc/blob/main/chibicc.h) | All | All structs, enums, and function declarations |
| [`tokenize.c`](https://github.com/rui314/chibicc/blob/main/tokenize.c) | Lexical Analysis | Tokenizer / lexer |
| [`preprocess.c`](https://github.com/rui314/chibicc/blob/main/preprocess.c) | Lexical Analysis | C preprocessor (#include, #define, macros) |
| [`parse.c`](https://github.com/rui314/chibicc/blob/main/parse.c) | Parsing + Semantic Analysis | Recursive descent parser, AST builder, type checker |
| [`type.c`](https://github.com/rui314/chibicc/blob/main/type.c) | Semantic Analysis | Type system: size, alignment, pointer arithmetic rules |
| [`codegen.c`](https://github.com/rui314/chibicc/blob/main/codegen.c) | Code Generation | x86-64 assembly emitter, walks the AST |
| [`hashmap.c`](https://github.com/rui314/chibicc/blob/main/hashmap.c) | Semantic Analysis | Hash map used by the symbol table |
| [`strings.c`](https://github.com/rui314/chibicc/blob/main/strings.c) | All | String utilities (intern, format) |
| [`main.c`](https://github.com/rui314/chibicc/blob/main/main.c) | All | Entry point, driver, argument parsing |

---

## The chibicc Learning Strategy

chibicc's git history is its best feature. Each commit is a self-contained,
compilable, testable step. The recommended workflow for every phase is:

```bash
# Browse the commit history to find the commit that adds the feature you just studied
git log --oneline

# Check out that specific commit and read it
git checkout <commit-hash>
git diff HEAD~1  # see exactly what was added

# Then return to main
git checkout main
```

This means you never stare at a finished, complex file. You read the compiler
growing one feature at a time — exactly how the course teaches each concept.

---

## Phase 1 — Lexical Analysis

**Course modules:** M1 (Compilation is Partial Evaluation), M2 (Evolution of Compilers),
M5 (Introduction to Lexical Analysis), M6 (Regular Expressions), M7 (Lexical Analysis
High-level Algorithm), M8 (DFA and NFA), M9 (Software Implementation of DFA and NFA),
M10 (Regular Expression to NFA)

**Duration:** ~4 weeks

### Why chibicc makes this easy

`tokenize.c` is a standalone file with zero dependencies on the rest of the compiler.
You can read it completely before writing a single line of code yourself.

### Key source locations

```
chibicc.h    → struct Token          (token type, value, source location)
chibicc.h    → enum TokenKind        (TK_IDENT, TK_NUM, TK_PUNCT, TK_EOF, ...)
tokenize.c   → tokenize()            (main entry: takes source string, returns Token list)
tokenize.c   → read_number()         (handles int, float, hex, octal literals)
tokenize.c   → read_string_literal() (string and char literal tokenizing)
tokenize.c   → convert_keywords()    (post-pass: turns identifier tokens into keyword tokens)
tokenize.c   → is_keyword()          (keyword lookup table)
```

**Direct course mapping:**
- `tokenize()` is the high-level DFA simulation from M7
- `convert_keywords()` demonstrates why keywords are handled as a post-pass (M5)
- `read_number()` is a hand-coded NFA for numeric literal patterns (M8–M10)

### Milestone 1 — Read `tokenize.c` completely

Open [`tokenize.c`](https://github.com/rui314/chibicc/blob/main/tokenize.c) and read it
top to bottom. It is ~300 lines. For each branch in the main dispatch loop, draw the
corresponding DFA state from M8. Answer these questions:

- Which states are accepting states?
- Where does the DFA handle ambiguity (e.g. `<` vs `<=`)?
- How are keywords distinguished from identifiers (M5)?

### Milestone 2 — Implement your own tokenizer

Write a tokenizer for a C subset: `int`, `if`, `while`, `return`, identifiers, integer
literals, `+`, `-`, `*`, `/`, `==`, `!=`, `<`, `>`, `=`, `(`, `)`, `{`, `}`, `;`.

Use NFA→DFA construction (M10) as your design method. Compare your output token stream
with chibicc's on the same input:

```bash
# Add a debug print to tokenize.c and recompile
# Then run both tokenizers on the same input file
```

---

## Phase 2 — Parsing

**Course modules:** M11 (Context Free Grammar), M12 (CFG Examples), M13 (Derivation Tree
and Order), M14 (Ambiguity in CFG), M15 (Error Handling), M16 (Abstract Syntax Tree),
M17 (Recursive Descent Parsing Introduction), M18 (Recursive Descent Parsing Algorithm),
M19 (Left Recursion), M20 (Predictive Parsing), M21 (LL1 Parsing), M22 (Constructing
First Sets), M23 (Constructing Follow Sets), M24 (Constructing LL1 Parsing Table),
M25 (Introduction to Bottom up Parsing), M26 (Shift Reduce Parsing), M27 (Handles),
M28 (Viable Prefixes), M29 (Structure of Viable Prefixes), M30 (NFA for Viable Prefixes),
M31 (LR0 Algorithm), M32 (SLR Implementation), M33 (SLR Examples)

**Duration:** ~6 weeks

### Why chibicc makes this easy

`parse.c` is a textbook recursive descent parser. Every grammar production rule is a
C function. The AST node types are all defined in `chibicc.h` and are easy to
visualize. There is no table, no automaton — just recursive function calls, exactly
as M17–18 teach.

### Key source locations

```
chibicc.h    → struct Node           (AST node: kind, type, left/right children, token)
chibicc.h    → enum NodeKind         (ND_ADD, ND_SUB, ND_IF, ND_WHILE, ND_CALL, ...)
chibicc.h    → struct Obj            (variable/function: name, type, offset, is_local)
parse.c      → program()             (top-level: parses a full translation unit)
parse.c      → function()            (parses a function definition)
parse.c      → compound_stmt()       (parses a { } block, handles scope)
parse.c      → stmt()                (dispatches on: if, while, for, return, expr)
parse.c      → expr()                (assignment expression)
parse.c      → equality()            (== and !=)
parse.c      → relational()          (< > <= >=)
parse.c      → add()                 (+ and -)
parse.c      → mul()                 (* and /)
parse.c      → unary()               (unary -, !, dereference, address-of)
parse.c      → primary()             (literals, identifiers, grouped expressions)
parse.c      → new_node()            (AST node constructor)
parse.c      → new_binary()          (binary AST node constructor)
```

**Direct course mapping:**
- Each `parse.c` function is one grammar production rule (M11, M13)
- The call hierarchy encodes operator precedence without ambiguity (M14)
- `primary()` → `unary()` → `mul()` → `add()` → `relational()` → `equality()` is the
  classic precedence climbing grammar from M18
- There is no left recursion anywhere — chibicc rewrites all left-recursive rules
  iteratively with `while` loops (M19)
- `compound_stmt()` demonstrates scope as a linked list of `Obj` (M34 preview)

### Milestone 3 — Draw the grammar from `parse.c`

Read [`parse.c`](https://github.com/rui314/chibicc/blob/main/parse.c) and write the
full CFG production rules for chibicc's expression grammar. Verify:

1. No left recursion (M19)
2. No ambiguity — each production uniquely predicts based on lookahead (M14, M20)
3. Operator precedence is correctly encoded in the nesting depth (M18)

### Milestone 4 — Implement First and Follow sets (theory exercise)

Take the grammar you extracted in Milestone 3 and manually compute First and Follow
sets (M22–23). Build the LL(1) parsing table (M24). Verify that every cell has at most
one entry — proving the grammar is LL(1).

chibicc doesn't use this table; it uses recursive descent instead. The exercise teaches
you *why* — recursive descent and LL(1) table-driven parsing are equivalent for LL(1)
grammars, but recursive descent is far simpler to implement.

### Milestone 5 — Extend chibicc's parser with a new statement

Pick a C construct chibicc doesn't yet support (e.g. `do-while`, `switch/case`, or
comma operator). Add it by:

1. Adding a new `NodeKind` variant in `chibicc.h`
2. Adding a new case in `stmt()` or `expr()` in `parse.c`
3. Adding code generation in `codegen.c` (Phase 4 preview)

Use git diff on chibicc's commits as a guide for how the author adds new constructs.

---

## Phase 3 — Semantic Analysis

**Course modules:** M34 (Semantic Analysis Introduction to Scope), M35 (Semantic
Analysis as Recursive Descent over AST), M36 (Types), M37 (Type Formalism), M37.1
(Subtyping Relation), M38 (Soundness of a Type System), M39 (More Type Rules), M39.1
(Type Rules for Reference Types), M40 (Type Environments), M41 (Type Environment and
Symbol Table), M42 (Typing Methods), M43 (Typechecker Implementation), M44 (Static vs
Dynamic Types), M45 (Recovering from Type Errors)

**Duration:** ~4 weeks

### Why chibicc makes this easy

chibicc separates type-checking into its own file, `type.c`, and interleaves it with
parsing in `parse.c`. This is exactly the design M35 teaches: semantic analysis as a
recursive descent over the AST. The symbol table is a simple linked list of `Obj`
structs — you can trace a variable from declaration to use to code generation in under
10 minutes.

### Key source locations

```
chibicc.h    → struct Type           (kind, size, align, pointer-to, array-of, params)
chibicc.h    → enum TypeKind         (TY_INT, TY_PTR, TY_ARRAY, TY_FUNC, TY_CHAR, ...)
chibicc.h    → struct Member         (struct field: name, type, offset)
type.c       → add_type()            (annotates every AST node with its type — M35)
type.c       → is_integer()          (type predicate)
type.c       → is_pointer()          (type predicate)
type.c       → pointer_to()          (type constructor: builds pointer type)
type.c       → func_type()           (type constructor: builds function type)
type.c       → array_of()            (type constructor: builds array type)
type.c       → copy_type()           (deep copy a Type node)
parse.c      → new_add()             (enforces ptr+int vs int+int rules — M39.1)
parse.c      → new_sub()             (enforces ptr-ptr vs ptr-int rules — M39.1)
parse.c      → find_var()            (symbol table lookup — M41)
hashmap.c    → hashmap_get()         (underlies the scope lookup chain — M40)
```

**Direct course mapping:**
- `add_type()` is M35's "semantic analysis as a recursive tree walk" made real
- `Type` structs are the type environment Γ from M40
- `find_var()` walking the `Obj` linked list is scope resolution from M34
- `new_add()` and `new_sub()` are pointer arithmetic type rules from M39.1
- `array_of()` and `pointer_to()` are type constructors from M37

### Milestone 6 — Trace a variable from declaration to use

Pick a simple C program:

```c
int main() {
  int x = 5;
  return x + 1;
}
```

Trace the following path through the source:
1. `x` declared → `parse.c` `declaration()` → new `Obj` pushed onto locals list
2. `x` used in `x + 1` → `parse.c` `primary()` → `find_var()` lookup
3. Type of `x` → `type.c` `add_type()` annotates the `ND_VAR` node with `TY_INT`
4. Type of `x + 1` → `add_type()` walks up to the `ND_ADD` node

### Milestone 7 — Add type checking to your own compiler

Implement M40–43 in your own compiler from Phase 2:

- Build a symbol table as a linked list of `{name, type, offset}` entries (same as
  chibicc's `Obj`)
- Walk your AST and annotate every node with a type (same as `add_type()`)
- Enforce type rules: arithmetic requires numeric types, assignment requires compatible
  types, function call argument count matches declaration
- Emit clear error messages on mismatches — model them on chibicc's `error_tok()`

```
chibicc.h    → error_tok()           (error reporting with source location)
tokenize.c   → error_at()            (points at the offending token in source)
```

---

## Phase 4 — Code Generation

**Course modules:** M46 (Runtime Organization), M47 (Activations), M48 (Activation
Records), M49 (Variables), M50 (Alignment), M51 (Stack Machine), M52 (Simulating a
Stack Machine), M53 (Code Generation), M54 (Code Generation for Function Calls), M55
(Code Generation for Variables), M56 (Code Generation Example), M57 (Temporaries),
M58 (Code Generation for Objects), M59 (Code Generation for Multiple Inheritance),
M60 (Code Generation for Diamond-Shaped Inheritance)

**Duration:** ~5 weeks

### Why chibicc makes this easy

`codegen.c` is a clean tree walk over the AST. Every `NodeKind` has its own `case`.
The code generator uses a simple stack-based strategy (M51–52) implemented with the
CPU's hardware stack — `push` and `pop` x86-64 instructions. This is the most direct
translation of the course's stack machine model into real assembly.

### Key source locations

```
codegen.c    → gen_expr()            (dispatches on NodeKind, emits expression code)
codegen.c    → gen_stmt()            (emits code for if, while, for, return, block)
codegen.c    → gen_addr()            (emits the *address* of an lvalue)
codegen.c    → push()                (emit: push rax — saves value to stack)
codegen.c    → pop()                 (emit: pop <reg> — restores value from stack)
codegen.c    → assign_lvar_offsets() (computes stack frame layout — M48, M50)
codegen.c    → emit_data()           (emits global variables into .data section)
codegen.c    → emit_text()           (emits all function bodies)
codegen.c    → count()               (generates unique label numbers for jumps)
```

**Direct course mapping:**
- `push()` / `pop()` in `codegen.c` are literally the stack machine operations from M51–52
- `assign_lvar_offsets()` is activation record layout from M47–48
- `gen_addr()` + `gen_expr()` together implement lvalue/rvalue distinction from M55
- Label generation with `count()` implements conditional and loop code from M53
- The `ND_CALL` case in `gen_expr()` is function call code generation from M54

### Milestone 8 — Read `codegen.c` with the stack machine model in mind

Open [`codegen.c`](https://github.com/rui314/chibicc/blob/main/codegen.c). Before
reading a single function, internalize this model from M51:

> Every expression leaves its result in `rax`. If a binary operation needs both
> operands simultaneously, the left operand is `push`ed to the stack while the right
> is evaluated, then `pop`ped into `rdi`. The operation then computes `rax OP rdi → rax`.

Now trace `gen_expr()` for `ND_ADD`:
1. Recursively emit left child → result in `rax`
2. `push rax` → left operand saved on stack
3. Recursively emit right child → result in `rax`
4. `pop rdi` → left operand restored to `rdi`
5. `add rax, rdi` → result in `rax`

This is M52's stack machine simulation made concrete.

### Milestone 9 — Trace a function's stack frame

For this C function:

```c
int add(int a, int b) {
  int c = a + b;
  return c;
}
```

Compile it with chibicc and inspect the output:

```bash
./chibicc -S test.c && cat test.s
```

Map each line of assembly to:
- `assign_lvar_offsets()` → where `a`, `b`, `c` live on the stack (M48, M49)
- Alignment padding → M50 (Alignment)
- Function prologue → `push rbp; mov rbp, rsp; sub rsp, N` (M47)
- Function epilogue → `mov rsp, rbp; pop rbp; ret` (M47)

### Milestone 10 — Implement a stack-based code generator

Extend your compiler from Phase 3 to emit x86-64 assembly using chibicc's exact
strategy: every expression pushes its result, binary operations pop two values,
compute, and push the result.

Target AT&T syntax (same as chibicc). Assemble and run:

```bash
gcc -o myout myout.s && ./myout; echo $?
```

Compare your output assembly with chibicc's on the same input.

---

## Phase 5 — Optimizations and Dataflow Analysis

**Course modules:** M68 (Intermediate Representation), M69 (Three Address Code), M70
(Static Single Assignment IR), M71 (Phi Nodes), M72 (Optimization Overview), M73
(Local Optimizations), M74 (Peephole Optimizations), M74.1 (Examples of Peephole
Optimizations), M75 (Global Optimization), M76 (DFA Values), M77 (Dataflow Analysis
Algorithm), M78 (DFA Example), M79 (DFA Loop Example), M80 (DFA Value Orderings),
M81 (Liveness Property), M82 (Liveness DFA), M83 (Liveness DFA Example), M84 (More
DFA Examples), M85 (Register Allocation), M86 (Register Spilling), M87 (Limitations
of Graph Colouring), M88 (Alternate DFA Representation), M89 (Gen and Kill Sets), M90
(Reaching Definitions), M91 (Introduction to DFA Framework), M92 (Live Variable
Analysis), M93 (Must Reach Definitions), M94 (Worklist Algorithm), M95 (Introduction
to Semilattices), M96 (Semilattice Representations), M97 (Product of Semilattices),
M98 (Semilattice Height), M98.1 (Range Analysis of Integers), M99–M100.1 (Transfer
Functions, Monotonicity), M101–M104 (Distributivity, Iteration, MFP/MOP/IDEAL),
M105–M108 (Partial Redundancy Elimination, Lazy Code Motion)

**Duration:** ~8 weeks

### Why chibicc is the right reference here

chibicc intentionally omits *all* optimizations. This is a feature for learning:
you can see exactly what unoptimized code looks like, then apply each optimization
the course teaches and measure the improvement yourself.

### Key source locations for study

```
codegen.c    → gen_expr() ND_ADD     (no strength reduction — a + 0 still emits add)
codegen.c    → gen_stmt() ND_IF      (no branch prediction hints, no condition folding)
codegen.c    → push() / pop()        (every subexpression uses the stack — no reg reuse)
```

### Milestone 11 — Document chibicc's missed optimizations

Write 5 C test programs, each targeting one missed optimization. Compile with chibicc
and `gcc -O2` and compare:

```bash
./chibicc -S test.c -o chibicc.s
gcc -O2 -S test.c -o gcc.s
diff chibicc.s gcc.s
```

| Test program | Expected optimization | Course module |
|---|---|---|
| `return 2 + 3;` | Constant folding | M73 |
| `int x = 5; x = x;` | Dead store elimination | M73 |
| `x * 8` | Strength reduction (shift) | M73 |
| `push rax; pop rdi` adjacent | Peephole: replace with `mov rdi, rax` | M74 |
| Loop-invariant `a + b` inside `for` | LICM — hoisted out | M110 |

### Milestone 12 — Add a peephole optimization pass to chibicc

Add a post-processing pass over chibicc's emitted assembly text. Implement these
rules from M74–74.1:

1. `push rax` followed immediately by `pop rax` → delete both
2. `push rax` followed immediately by `pop rdi` → replace with `mov rdi, rax`
3. `mov rax, $0` → replace with `xor rax, rax`
4. `add rax, $0` or `sub rax, $0` → delete
5. `imul rax, rax, $1` → delete

Measure the instruction count reduction on `test/` suite programs.

### Milestone 13 — Implement liveness analysis

Build a liveness dataflow analysis (M81–83) over a three-address code (TAC)
representation of chibicc's output:

**Step 1:** Convert chibicc's assembly into a TAC IR (M69). Each assembly instruction
maps to one TAC instruction.

**Step 2:** Build basic blocks — split at labels and jump instructions.

**Step 3:** For each basic block, compute `USE` and `DEF` sets (M89):
- `USE[B]` = variables read before any write in B
- `DEF[B]` = variables written in B

**Step 4:** Iterate the liveness equations until fixed point (M82, M94 Worklist):
```
LIVE_IN[B]  = USE[B] ∪ (LIVE_OUT[B] − DEF[B])
LIVE_OUT[B] = ∪ LIVE_IN[S]  for each successor S of B
```

**Step 5:** At each program point, record which variables are simultaneously live.

### Milestone 14 — Implement register allocation via graph colouring

Using the liveness information from Milestone 13 (M85–87):

**Step 1:** Build the interference graph. Two variables interfere if they are
simultaneously live at any point.

**Step 2:** Color the graph with k = 6 colors (the 6 caller-saved registers in
System V AMD64: `rax`, `rcx`, `rdx`, `rsi`, `rdi`, `r8`).

**Step 3:** Handle spilling (M86): variables that can't be colored get a stack slot.
Insert `mov [rbp-N], reg` and `mov reg, [rbp-N]` around their uses.

Compare your register usage against chibicc's push/pop strategy. On any non-trivial
function, your allocator should use fewer stack operations.

---

## Phase 6 — Operational Semantics

**Course modules:** M61 (Programming Language Semantics), M62 (Ways of Specifying
Semantics), M63 (Operational Semantics Introduction), M64 (Operational Semantics for
Simple Operations), M65 (Operational Semantics for Control Flow and Declarations),
M66 (Operational Semantics of the Allocation Operation), M67 (Operational Semantics
for Function Call Dispatch)

**Duration:** ~3 weeks (study alongside Phase 5)

### chibicc mapping

chibicc's `codegen.c` is an operational semantics made executable. The "rules"
from M63–67 become `case` statements in `gen_expr()` and `gen_stmt()`.

```
codegen.c    → ND_IF case            → operational semantics for if/else (M65)
codegen.c    → ND_WHILE case         → operational semantics for while loops (M65)
codegen.c    → ND_CALL case          → function call dispatch semantics (M67)
codegen.c    → ND_ADDR / ND_DEREF    → allocation and pointer semantics (M66)
codegen.c    → gen_addr()            → lvalue evaluation rule (M64)
```

### Milestone 15 — Write formal rules for chibicc's code generation

For each of these `NodeKind` cases in `codegen.c`, write the corresponding
operational semantics rule in the style taught in M63–64:

- `ND_NUM` (integer literal)
- `ND_ADD` (integer addition)
- `ND_ASSIGN` (assignment)
- `ND_IF` (conditional)
- `ND_WHILE` (loop)
- `ND_CALL` (function call)

Example format:
```
⟨e1, σ⟩ → v1    ⟨e2, σ⟩ → v2
───────────────────────────────
⟨e1 + e2, σ⟩ → v1 + v2
```

---

## Phase 7 — Advanced: UB Semantics and Static Analysis

**Course modules:** M166 (Undefined Behaviour Semantics), M167 (Undefined Behaviour
in IR), M168 (Introduction to Poison Values), M169 (Poison Value Operational
Semantics), M170 (Immediate vs Deferred UB), M171 (Undefined Values), M172 (Undef
Value Operational Semantics), M173 (WHYs of Undef and Poison), M174 (Freeze Opcode
in LLVM), M175 (Introduction to Static Analysis Approaches), M176 (Assertions),
M177 (Invariants), M178 (Verification Conditions), M179 (VC for If-Then-Else), M180
(VC for Sequence), M181 (Sequencing with If-Then-Else), M182 (Exponential Paths
Problem), M183 (While Loop Verification Condition), M184 (Proof Method), M185 (Hoare
Logic Rules), M186 (Hoare Logic Rule for While), M187 (Predicate Transformers), M188
(More Weakest Precondition Rules), M189 (Weakest Liberal Precondition), M190 (WLP
for While Loop)

**Duration:** ongoing

### chibicc mapping

chibicc does not sanitize undefined behavior — it simply emits whatever the C standard
leaves undefined. This makes it a perfect test bed for M166–170.

```
codegen.c    → ND_DIV case           (no division-by-zero check — UB in C)
parse.c      → new_add()             (pointer arithmetic — UB if out of bounds)
type.c       → (no overflow checks)  (signed overflow is UB — chibicc ignores it)
```

### Milestone 16 — Experiment with UB in chibicc

Write C programs that trigger common UB. Observe what chibicc emits vs. what
GCC/Clang with sanitizers detect:

```bash
# Signed integer overflow
echo 'int main() { int x = 2147483647; return x + 1; }' > ub.c
./chibicc ub.c && ./a.out; echo $?

# Compare with sanitizer
gcc -fsanitize=undefined -o ub_safe ub.c && ./ub_safe

# Uninitialized read
echo 'int main() { int x; return x; }' > uninit.c
./chibicc uninit.c && ./a.out
clang -fsanitize=memory -o uninit_safe uninit.c && ./uninit_safe
```

Connect each behavior to the poison value and undef models from M168–172.

### Milestone 17 — Add a simple assertion verifier

For straight-line code (no loops or branches), implement verification condition
generation (M178–M180):

1. Parse a function body with `assert(expr)` calls
2. For each `assert`, generate the weakest precondition (M187) working backwards
3. Check if the precondition is always satisfied by symbolic evaluation

This goes beyond chibicc — you are now building static analysis *on top of* your
compiler knowledge.

---

## Phase 8 — Loop Optimizations and Polyhedral Framework

**Course modules:** M110 (Loop Inversion), M111 (Introduction to Natural Loops and
Dominators), M112 (Immediate Dominators), M113 (Natural Loop Definition), M114
(Computing Natural Loops), M115 (Constructing Reducible Graphs), M116 (Symbolic
Analysis of Loops), M117–M121.1 (Region-based Analysis), M122–M124 (Induction
Variables), M125–M165 (Polyhedral Framework, Data Reuse, Affine Transforms,
Parallelization, Pipelining)

**Duration:** ~10 weeks (advanced — after completing Phases 1–6)

### chibicc has no loop optimization — that is the lesson

chibicc emits unoptimized loop code. Use this as your baseline, then study LLVM's
loop passes and Polly (LLVM's polyhedral optimizer) to see what production compilers
do.

```
codegen.c    → ND_FOR / ND_WHILE     (baseline: no LICM, no unrolling, no vectorization)
```

**Suggested resources for this phase:**
- [LLVM Loop Passes](https://llvm.org/docs/Passes.html) — see `loop-rotate`, `licm`,
  `loop-unroll`
- [Polly — LLVM's Polyhedral Optimizer](https://polly.llvm.org/) — direct implementation
  of M128–M165
- [isl library](https://repo.or.cz/w/isl.git) — the integer set library used by Polly,
  implements Fourier-Motzkin (M130)

### Milestone 18 — Identify natural loops in chibicc's CFG

Compile a loop-heavy C program with chibicc and:
1. Build the control flow graph (CFG) from the emitted assembly labels and jumps
2. Compute dominators (M112) using the dataflow algorithm
3. Find all back edges (edges where target dominates source)
4. Identify natural loops (M113–114)

This is pure analysis on top of chibicc's output — no modification needed.

---

## Commit-by-Commit Study Guide

chibicc's git history is linear and each commit is a lesson. Key milestones in the
commit history to check out and study:

| What to find in git log | Course phase |
|---|---|
| "Tokenize" initial commit | Phase 1 — Lexical Analysis |
| First arithmetic expression | Phase 2 — Parsing begins |
| "Add AST" or "Add node" commit | Phase 2 — M16 AST |
| First if/while support | Phase 2 — M17–18 Recursive Descent |
| "Add type" or type checking commit | Phase 3 — M36–43 |
| "Add pointer" type | Phase 3 — M39.1 |
| First function support | Phase 4 — M47–48 |
| "Add local variables" | Phase 4 — M49 |
| Struct support | Phase 4 — M58 |
| Variadic functions | Phase 4 — M54 |

```bash
# Browse commits and check out any one to study it in isolation
git log --oneline | head -50
git checkout <hash>
git diff HEAD~1 -- tokenize.c    # see exactly what changed
git checkout main
```

---

## Week-by-Week Schedule

| Week | Course Modules | chibicc Task |
|---|---|---|
| 1–2 | M1–M5 | Clone repo, read all of `chibicc.h`, sketch the data flow |
| 3–4 | M6–M10 | Read `tokenize.c` completely, implement mini-lexer |
| 5–7 | M11–M18 | Trace `parse.c` expression functions, draw the grammar |
| 8–9 | M19–M24 | LL(1) table exercise from extracted grammar |
| 10–11 | M25–M33 | Read SLR theory, contrast with chibicc's RD approach |
| 12–13 | M34–M35 | Trace `find_var()` and scope chain in `parse.c` |
| 14–15 | M36–M43 | Read `type.c` and `add_type()` completely |
| 16 | M44–M45 | Study chibicc's `error_tok()` and error recovery |
| 17–18 | M46–M52 | Read `assign_lvar_offsets()` and `push()`/`pop()` in `codegen.c` |
| 19–20 | M53–M57 | Trace full code gen for a function with locals |
| 21–22 | M58–M60 | Read struct member access code gen |
| 23–24 | M61–M67 | Write formal semantics rules for 6 NodeKind cases |
| 25–26 | M68–M74.1 | chibicc vs gcc -O2 diff, implement peephole pass |
| 27–29 | M75–M83 | Implement liveness DFA over TAC |
| 30–32 | M84–M94 | Worklist algorithm, implement on your liveness analysis |
| 33–35 | M95–M108 | Semilattice theory, DFA framework, MFP solution |
| 36–38 | M110–M124 | Natural loops from chibicc CFG, dominator analysis |
| 39+ | M125–M165 | Polyhedral framework — study Polly alongside course |
| ongoing | M166–M190 | UB experiments, Hoare logic, verification conditions |

---

## Useful Commands

```bash
# Build chibicc
make

# Compile to assembly and inspect
./chibicc -S yourfile.c && cat yourfile.s

# Compile and run
./chibicc yourfile.c -o out && ./out

# Run chibicc's own test suite
make test

# See what one commit added (replace HASH with any commit from git log)
git diff HASH~1 HASH

# Compare chibicc vs gcc -O2 on the same file
./chibicc -S test.c -o chibicc.s
gcc -O2 -S test.c -o gcc.s
diff chibicc.s gcc.s
```

---

## What chibicc Does Not Cover

| Course Modules | Topic | Go here next |
|---|---|---|
| M70–71 | SSA form and phi nodes | [LLVM IR reference](https://llvm.org/docs/LangRef.html) |
| M85–87 | Register allocation (graph colouring) | [LLVM RegAlloc](https://llvm.org/docs/CodeGenerator.html#register-allocation) |
| M105–108 | Lazy code motion, PRE | LLVM `GVN` and `EarlyCSE` passes |
| M110–124 | Loop analysis, natural loops | LLVM loop analysis passes |
| M125–165 | Polyhedral framework | [Polly](https://polly.llvm.org/) |
| M166–174 | Poison / undef semantics | [Alive2](https://github.com/AliveToolkit/alive2) |
| M185–190 | Hoare logic, weakest precondition | [Frama-C](https://frama-c.com/) |

---

## References

- **chibicc repository:** https://github.com/rui314/chibicc
- **Author's companion book (Japanese):** https://www.sigbus.info/compilerbook
- **TCC (for contrast — no IR, fused pass):** https://github.com/TinyCC/tinycc
- **LLVM (next step after chibicc):** https://github.com/llvm/llvm-project
- **Polly — polyhedral optimizer:** https://polly.llvm.org/
- **Alive2 — optimization verifier:** https://github.com/AliveToolkit/alive2
- **QBE — small optimizing backend:** https://c9x.me/compile/
