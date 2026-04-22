# Laboratory Work No. 4: Regular expressions

### Course: Formal Languages & Finite Automata
### Author: Polina Clepicova FAF-243
### Variant 4

---

## Theory
Regular expressions (Regex) are a formal way to describe a set of strings (a formal language) over an alphabet $\Sigma$. In the context of Formal Languages and Automata Theory, a regular expression is a simplified notation for representing Regular Languages, which can also be recognized by Finite Automata (DFA/NFA).

The key operations used in this work are:
1.  **Union (OR) $|$**: If $L_1$ and $L_2$ are languages, $L_1 | L_2 = L_1 \cup L_2$.
2.  **Concatenation $\cdot$**: Combining strings from two sets (implicit in the variant).
3.  **Kleene Star $^*$**: Represents zero or more repetitions of a symbol/group.
4.  **Kleene Plus $^+$**: Represents one or more repetitions.
5.  **Finite Power $^n$**: Represents exactly $n$ repetitions.

---

## Objectives
1. Understand the mathematical foundation of regular expressions and their practical application.
2. Implement a dynamic interpreter for regular expressions that generates valid combinations of symbols without hardcoding the structure.
3. Handle repetitions ($^*$ and $^+$) by implementing a limit of 5 iterations to prevent infinite or extremely long string generation.
4. (Bonus) Provide a step-by-step sequence of how the regular expression is processed/parsed.
5. Generate a report covering implementation details and results.

---

## Implementation Description
The solution is built using a **Recursive Descent Parser** to translate a raw string regex into an **Abstract Syntax Tree (AST)**. 

### Key Components:
* **AST Nodes**: 

  `Literal`: The base unit representing a single character.

  `Or`: Handles the choice between two branches.

  `Concat`: Combines results of two nodes.

  `Repeat`: Handles $^*$, $^+$, and $^n$ logic with a `min_rep` and `max_rep` constraint.
* **The Parser**:
  
    It scans the input string and prioritizes operations according to standard formal language rules: Groups `()` first, then Repeat modifiers, then Concatenation, and finally Union `|`.
  
    It dynamically builds the tree, allowing it to handle any valid regex string provided as input.
* **Generation Logic**: 
  
    The `gen()` method in each node uses recursion to build all possible strings. For concatenations, it calculates the Cartesian product of possible left-side and right-side strings.

---

## Code

```python
import random

# --- Data Structures for AST ---
class Literal:
    def __init__(self, char):
        self.char = char
    def gen(self):
        return [self.char]
    def __str__(self):
        return self.char

class Concat:
    def __init__(self, left, right):
        self.left = left
        self.right = right
    def gen(self):
        return [l + r for l in self.left.gen() for r in self.right.gen()]
    def __str__(self):
        return f"Concat({self.left}, {self.right})"

class Or:
    def __init__(self, left, right):
        self.left = left
        self.right = right
    def gen(self):
        return self.left.gen() + self.right.gen()
    def __str__(self):
        return f"Or({self.left}, {self.right})"

class Repeat:
    def __init__(self, node, min_rep, max_rep):
        self.node = node
        self.min_rep = min_rep
        self.max_rep = max_rep
    def gen(self):
        results = []
        for i in range(self.min_rep, self.max_rep + 1):
            if i == 0:
                results.append("")
            else:
                current = [""]
                for _ in range(i):
                    current = [c + n for c in current for n in self.node.gen()]
                results.extend(current)
        return results
    def __str__(self):
        return f"Repeat({self.node}, {self.min_rep}..{self.max_rep})"
``` 


## Recursive Descent Parser
```python
class RegexParser:
    def __init__(self, text):
        self.text = text.replace(" ", "")
        self.pos = 0

    def parse(self):
        print(f"\nProcessing expression: {self.text}")
        return self.parse_E()

    def peek(self):
        return self.text[self.pos] if self.pos < len(self.text) else None

    def consume(self):
        char = self.peek()
        self.pos += 1
        return char

    def parse_E(self):
        node = self.parse_T()
        while self.peek() == '|':
            self.consume()
            right = self.parse_T()
            print(f" -> Found OR operation: ({node} | {right})")
            node = Or(node, right)
        return node

    def parse_T(self):
        nodes = [self.parse_F()]
        while self.peek() is not None and self.peek() not in (')', '|'):
            nodes.append(self.parse_F())
        if len(nodes) == 1: return nodes[0]
        node = nodes[0]
        for right in nodes[1:]:
            node = Concat(node, right)
        return node

    def parse_F(self):
        node = self.parse_P()
        modifier = self.peek()
        if modifier == '*':
            self.consume()
            print(f" -> Applying Kleene Star (*) to: {node}")
            node = Repeat(node, 0, 5)
        elif modifier == '+':
            self.consume()
            print(f" -> Applying Kleene Plus (+) to: {node}")
            node = Repeat(node, 1, 5)
        elif modifier == '^':
            self.consume()
            num = ""
            while self.peek() and self.peek().isdigit(): num += self.consume()
            print(f" -> Applying Exact Repetition ({num}) to: {node}")
            node = Repeat(node, int(num), int(num))
        return node

    def parse_P(self):
        char = self.peek()
        if char == '(':
            self.consume()
            node = self.parse_E()
            self.consume() # consume ')'
            return node
        return Literal(self.consume())
```

## Execution 
```python
variants = ["(S|T)(U|V)W*Y+24", "L(M|N)O^3P*Q(2|3)", "R*S(T|U|V)W(X|Y|Z)^2"]
for v in variants:
    p = RegexParser(v)
    tree = p.parse()
    results = list(set(tree.gen()))
    print(f"Example results: {random.sample(results, min(3, len(results)))}")
```
--- 
## Challenges and Difficulties
1. Implicit Concatenation: Unlike the OR operator (|), concatenation doesn't have a visible character. Implementing the parser to recognize when one expression ends and another begins (especially with parentheses) was the main logical challenge.
2. Combinatorial Explosion: Since some expressions generate thousands of combinations (due to Cartesian Product), I used random.sample to display results efficiently.
--- 

## Conclusions
The completion of this laboratory work allowed for a deep dive into the practical application of regular expressions within the framework of **Formal Language Theory**. Unlike standard string matching, this task required the creation of a generative model, which highlights the dual nature of regex: as both a recognizer (automaton) and a generator of languages.

### Technical Achievements:
1.  **Dynamic Interpretation**: The core of the implementation is a **Recursive Descent Parser**. By building an **Abstract Syntax Tree (AST)**, the program successfully avoids hardcoding. This means the system can process any valid regular expression input, maintaining the correct hierarchy of operations (Operator Precedence) regardless of complexity.
2.  **Generative Logic**: Using a recursive `gen()` method allowed for the systematic construction of the language defined by the regex. The implementation of the **Cartesian Product** for concatenation operations was crucial to ensuring that all possible combinations of prefixes and suffixes were accounted for.
3.  **Constraint Management**: To satisfy the requirement of avoiding infinite or excessively long strings, a localized limit was integrated into the `Repeat` node. By capping the Kleene Star ($^*$) and Kleene Plus ($^+$) at 5 iterations, the output remains readable and computationally efficient while still demonstrating the "infinite" potential of the regular language.
4.  **Parsing Transparency**: The added logging functionality (the "sequence of processing") provides a clear view of how the parser breaks down tokens and handles nested groups, fulfilling the bonus objective and proving the robustness of the algorithm.

### Final Results:
The system was tested with the three complex expressions assigned for Variant 4. The output confirmed that:

    - Grouping and Union** (`|`) correctly branch the generation process.
    - Repetition operators** ($^*$, $^+$, $^n$) accurately reflect the cardinalities of the resulting sets.
    - Nested structures** (like repetitions of a grouped choice) are handled without logical errors.

In conclusion, this work demonstrates how formal grammars can be translated into functional code. The project successfully bridges the gap between the mathematical definition of Regular Expressions and practical software engineering, providing a reliable tool for string generation based on formal rules.

---
### System Results
For the Variant 4 expressions, the system generated:

Exp 1: Strings like SUWY24, TVWWWYYYY24.

Exp 2: Strings like LMOOOPPQ3, LNOOOPPPQ2.

Exp 3: Strings like RRSVWY Y, STWXX.

---
## References
Hopcroft, J. E., Motwani, R., & Ullman, J. D. "Introduction to Automata Theory, Languages, and Computation"
