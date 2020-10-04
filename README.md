# Fast'n'lazy stack
A funky stack that is lazy but fast

## Scenario
We have to implement 3 operations on a stack-like list structure: "push \[value\]", "pop". Additionally, an increment operation, "inc \[i\] \[value\]", that adds \[value\] to the bottom \[i\] elements of the stack (\[value\] can be negative). The push and pop operations are fairly simple, but we want to make the increment operation "lazy". So, we'll need to modify them and implement the increment operation such that we don't do any direct calculation on the stack values until the user needs it (ie peeks the top value).

The idea is that a lazy increment is an optimization that does less processing at the time that the instruction is applied. We need do some accounting with a running sum of value increments at each applicable index. When the value is retrieved, we'll have an extra arithmetic operation that accounts for any past increment operation that should have been applied to it. This is supposed to be faster than adding the values to each of the bottom elements at the moment that the increment function is called, especially when you have a very large stack and many increments on a significant bottom portion of the stack.

The following instructions() function is not part of the solution. It is just used to generate push/pop/inc instructions for testing the implementation.


```python
import random
def instructions(n):
    """
    Randomly generates operations: 'push v', 'pop', or 'inc i v'
    For ease of implementation, the function won't generate a pop instruction
    on an empty stack, and it will ask to increment i elements where
    i <= current size of the stack.
    The function is slightly more likely to generate 'push' than the other instructions.
    """
    capacity = 0
    for _ in range(n):
        if capacity > 0:
            select = random.randint(0,3)
        else:
            select = 2
            
        v = random.randint(-100,100)
        
        if select == 0:
            capacity -= 1
            operation = 'pop'
        elif select == 1:
            i = random.randint(1,capacity)
            operation = f'inc {i} {v}'
        elif select >= 2:
            capacity += 1
            operation = f'push {v}'
            
#         print(f"cap = {capacity}")
        yield operation

```

Using a fixed seed, we'll generate "consistently random" instructions 😁


```python
random.seed(42)
# c = 0
for instruction in instructions(50):
#     if 'pop' in instruction:
#         c -= 1
#     elif 'push' in instruction:
#         c+= 1
    print(instruction)
# print(c)

```

    push 63
    pop
    push 89
    push -38
    inc 1 -65
    pop
    push -92
    pop
    inc 1 -41
    inc 1 83
    inc 1 14
    pop
    push -60
    push -13
    push -61
    inc 2 95
    pop
    push -76
    push -12
    push -89
    push 37
    pop
    pop
    push 60
    push 47
    inc 1 80
    pop
    inc 3 97
    pop
    pop
    push 16
    push -59
    push -10
    inc 3 71
    pop
    inc 2 36
    inc 4 18
    push 63
    inc 3 75
    pop
    pop
    push -32
    pop
    push -46
    push 1
    push -64
    push -65
    inc 5 90
    push 49
    push -8
    

## Baseline implementation
This straightfowardly implements push, pop, and increment using the existing python list data structure for a stack. For 'inc i v', it immediately adds \[value\] to the bottom \[i\] elements in the stack. The following cell prints the entire stack after each operation, and the cell after it times the implementation without printing. Refer to the sequence instructions above to follow which operation occured.

The do_it() function is just a structure to demonstrate the implementation, the important part is loop and conditionals inside it.


```python
random.seed(42)

def do_it(number_of_instructions=50, print_func=(lambda x: x)):

    stack = []
    
    def pop_stack():
        """
        Assume global stack is available
        """
        if len(stack) > 0:
            stack.pop()
            
    def peek():
        """
        Assume global stack is available
        """
        if len(stack) > 0:
            return stack[-1]
        else:
            return "EMPTY"
        
    for instruction in instructions(number_of_instructions):
        op = instruction.split()
        if op[0] == 'push':
            stack.append(int(op[1]))
        elif op[0] == 'pop':
            pop_stack()
        else:
            i = int(op[1])
            v = int(op[2])
            for r in range(i):
                stack[r] += v
                
        print_func(stack)

    # Now let's simulate a user 1) removing half of the stack and 2) examining the next top 3 set of elements
    for _ in range(int(len(stack)/2)):
        pop_stack()
    
    # peek and pop 3 elements
    for _ in range(3):
        print_func(peek())
        pop_stack()
        
do_it(print_func=lambda x: print(x))
```

    [63]
    []
    [89]
    [89, -38]
    [24, -38]
    [24]
    [24, -92]
    [24]
    [-17]
    [66]
    [80]
    []
    [-60]
    [-60, -13]
    [-60, -13, -61]
    [35, 82, -61]
    [35, 82]
    [35, 82, -76]
    [35, 82, -76, -12]
    [35, 82, -76, -12, -89]
    [35, 82, -76, -12, -89, 37]
    [35, 82, -76, -12, -89]
    [35, 82, -76, -12]
    [35, 82, -76, -12, 60]
    [35, 82, -76, -12, 60, 47]
    [115, 82, -76, -12, 60, 47]
    [115, 82, -76, -12, 60]
    [212, 179, 21, -12, 60]
    [212, 179, 21, -12]
    [212, 179, 21]
    [212, 179, 21, 16]
    [212, 179, 21, 16, -59]
    [212, 179, 21, 16, -59, -10]
    [283, 250, 92, 16, -59, -10]
    [283, 250, 92, 16, -59]
    [319, 286, 92, 16, -59]
    [337, 304, 110, 34, -59]
    [337, 304, 110, 34, -59, 63]
    [412, 379, 185, 34, -59, 63]
    [412, 379, 185, 34, -59]
    [412, 379, 185, 34]
    [412, 379, 185, 34, -32]
    [412, 379, 185, 34]
    [412, 379, 185, 34, -46]
    [412, 379, 185, 34, -46, 1]
    [412, 379, 185, 34, -46, 1, -64]
    [412, 379, 185, 34, -46, 1, -64, -65]
    [502, 469, 275, 124, 44, 1, -64, -65]
    [502, 469, 275, 124, 44, 1, -64, -65, 49]
    [502, 469, 275, 124, 44, 1, -64, -65, 49, -8]
    44
    124
    275
    

Now let's see the timing.


```python
%%timeit
random.seed(42)

do_it(number_of_instructions=10000)
```

    245 ms ± 29.4 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
    

## Make it lazy
The approach to make the increment lazy is to use another collection to account for what values to increment and at which section indices. This extra expense\* of this optimization is that it takes up more space: O(n) where n is the number of unique i of increments encountered in the instructions.

Observe that

    1. after each processing of an instruction, we have an intermediate print out of the stack, and it shows the "unincremented" elements, but...
    
    2. when we simulate the user examining the elements at the top of the actual stack, the values are consistent to what they should be when the increment operations were applied. 

\*Outside the scope of this demonstration, this could be improved by a "maintenance" in processor idle time: applying the collective increments when the user is not interacting with the stack, then freeing up space.


```python
random.seed(42)

def do_it_better(number_of_instructions=50, print_func=(lambda x: x)):
    # Globals
    stack = []
    incs = {}

    #TODO: move these into class
    def push_stack(op):
        """
        op param required, but assume global stack and inc is available
        """
        stack.append(int(op[1]))

    def pop_stack():
        """
        Assume global stack and inc is available
        """
        length = len(stack)
        if length > 0:
            # clear top transaction and preserve its downward transactions by moving it one position down
            if incs.get(length):

                incs[length-1] = incs.get(length-1, 0) + incs.get(length)
                incs[length] = 0
            stack.pop()
    
    def inc_stack(op):
        """
        op param required, but assume global stack and inc is available
        Note the ith_element means apply to indices [0: ith_element-1] inclusive of the stack;
        this also makes incs[0] meaningless
        """
        ith_element = int(op[1])
        v = int(op[2])
        incs[ith_element] = incs.get(ith_element, 0) + v
    
    def peek_stack():
        """
        assume global stack and inc is available
        Note the ith_element means apply to indices [0: ith_element-1] inclusive of the stack
        """
        
        ith_element = len(stack)
        if ith_element > 0:
            return stack[-1] + incs[ith_element]
        else:
            return "EMPTY"
    
    for instruction in instructions(number_of_instructions):
        op = instruction.split()
        if op[0] == 'push':
            push_stack(op)
        elif op[0] == 'pop':
            pop_stack()
        else:
            inc_stack(op)

        print_func(stack)
    
    
    # Now let's simulate a user 1) removing half of the stack and 2) examining the next top 3 set of elements
    for _ in range(int(len(stack)/2)):
        pop_stack()
    
    # peek and pop 3 elements
    for _ in range(3):
        print_func(peek_stack())
        pop_stack()

do_it_better(print_func=(lambda x: print(x)))       
```

    [63]
    []
    [89]
    [89, -38]
    [89, -38]
    [89]
    [89, -92]
    [89]
    [89]
    [89]
    [89]
    []
    [-60]
    [-60, -13]
    [-60, -13, -61]
    [-60, -13, -61]
    [-60, -13]
    [-60, -13, -76]
    [-60, -13, -76, -12]
    [-60, -13, -76, -12, -89]
    [-60, -13, -76, -12, -89, 37]
    [-60, -13, -76, -12, -89]
    [-60, -13, -76, -12]
    [-60, -13, -76, -12, 60]
    [-60, -13, -76, -12, 60, 47]
    [-60, -13, -76, -12, 60, 47]
    [-60, -13, -76, -12, 60]
    [-60, -13, -76, -12, 60]
    [-60, -13, -76, -12]
    [-60, -13, -76]
    [-60, -13, -76, 16]
    [-60, -13, -76, 16, -59]
    [-60, -13, -76, 16, -59, -10]
    [-60, -13, -76, 16, -59, -10]
    [-60, -13, -76, 16, -59]
    [-60, -13, -76, 16, -59]
    [-60, -13, -76, 16, -59]
    [-60, -13, -76, 16, -59, 63]
    [-60, -13, -76, 16, -59, 63]
    [-60, -13, -76, 16, -59]
    [-60, -13, -76, 16]
    [-60, -13, -76, 16, -32]
    [-60, -13, -76, 16]
    [-60, -13, -76, 16, -46]
    [-60, -13, -76, 16, -46, 1]
    [-60, -13, -76, 16, -46, 1, -64]
    [-60, -13, -76, 16, -46, 1, -64, -65]
    [-60, -13, -76, 16, -46, 1, -64, -65]
    [-60, -13, -76, 16, -46, 1, -64, -65, 49]
    [-60, -13, -76, 16, -46, 1, -64, -65, 49, -8]
    44
    124
    275
    

So now we've verfied the results are consistent. Let's see how fast it would be to process many instructions and a larger stack


```python
%%timeit
random.seed(42)

do_it_better(number_of_instructions=10000)       
```

    50.2 ms ± 6.37 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
    

### So, after 10000 randomized pop/push/inc instructions, the lazy stack speed up is 5 times faster over the baseline (50ms vs 245 ms)! 😎
