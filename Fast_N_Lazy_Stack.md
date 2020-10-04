
# Fast'n'lazy stack
A funky stack that is lazy but fast.

## Scenario
We have to implement 3 operations on a stack-like list structure: `push [value]`, `pop`, and an increment operation, `inc [i] [value]`, that adds `[value]` to the bottom `[i]` elements of the stack (`[value]` can be negative). For example, `inc 3 10` changes the bottom 3 data points in a stack as such:
```
| 5|      | 5| (top)
| 4|      | 4|
| 2|  =>  |12|
| 2|      |12|
| 3|      |13| (bottom)
```
The push and pop operations are fairly simple, but we want to make the increment operation "lazy". So, we'll need to modify them as we implement the increment operation such that we don't do any direct calculation on the stack values until the user actually reads data from the stack (i.e. peeks the top value).

The idea is that a lazy increment is an *optimization* that does less processing at the time that the instruction is recieved. We need do some accounting with a running sum of value increments at each applicable index. When the value is retrieved, we'll have an extra arithmetic operation that accounts for any past increment operation that should have been applied to it. This is supposed to be faster than adding the values to each of the bottom elements at the moment that the increment function is called, especially when you have a very large stack and many increments on a significant bottom portion of the stack.

The following `instructions()` function is not part of the solution. It is just used to generate push/pop/inc instructions for testing the implementation.


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

The `do_it()` function is just a structure to demonstrate the implementation, the important part is loop and conditionals inside it.


```python
random.seed(42)

class BaseStack:
    
    def __init__(self):
        self.stack = []
        
    def push(self, v):
        self.stack.append(v)
        
    def pop(self):
        if len(self.stack) > 0:
            self.stack.pop()        

    def inc(self, i, v):
        for r in range(i):
            self.stack[r] += v
    
    def peek(self):
        if len(self.stack) > 0:
            return self.stack[-1]
        else:
            return "EMPTY"   
    
    def __len__(self):
        return len(self.stack)
    
    def __str__(self):
        return f"{self.stack}"
    
    
def do_it(number_of_instructions=50, print_func=(lambda x: x)):

    stack = BaseStack()
        
    for instruction in instructions(number_of_instructions):
        op = instruction.split()
        if op[0] == 'push':
            stack.push(int(op[1]))
        elif op[0] == 'pop':
            stack.pop()
        else:
            stack.inc(int(op[1]), int(op[2]))
                
        print_func(stack)

    # Now let's simulate a user 1) removing half of the stack and 2) examining the next top 3 set of elements
    for _ in range(int(len(stack)/2)):
        stack.pop()
    
    # peek and pop 3 elements
    for _ in range(3):
        print_func(stack.peek())
        stack.pop()
        
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

do_it(number_of_instructions=20000)
```

    998 ms ± 209 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
    

## Make it lazy
In general, `inc i v` applies to stack elements at indices i-1 down to 0. The approach to make the increment lazy is to use another data structure to account for what values to increment and at which section indices. We "save" information from any increment instruction into this collection, and only apply the increment when data from the stack is needed. The extra expense\* of this optimization is that it takes up more space: O(n) where n is the number of unique i of increments encountered in the instructions. The collection is a dictionary where each entry is of the format `{k: value}`, where `value` is the summation of all `value` in `inc k value` received for that specific `k`. To ensure the summation applies downward, we'll do the following...

The `incs` dictionary is in lockstep with the the top of the stack. It also holds all various increments applicable to lower sections of the stack. Let's refer to the index at the top of the stack as `j`. If we have a 0-indexed stack, then j == length of the stack minus 1. When data from the top of the stack is accessed (i.e. peek), we will get the value from the top of the stack and add to it the value collected from all inc instruction that applies to it. Getting this value is simply the value at dictionary's key = length of stack. Whenever we pop the stack, we need to preserve the value increments in the dictionary add the value to the very next index/key down.

Observe on the output of the following cell that:

- after each processing of an instruction, we have an intermediate print out of the stack, and it shows the "unincremented" elements, but...
    
- when we simulate the user examining the elements at the top of the actual stack, the values are consistent to what they should be when the increment operations were applied. 

\*Outside the scope of this demonstration, this could be improved by a "maintenance" in processor idle time: applying the collective increments when the user is not interacting with the stack, then freeing up `incs` dictionary until more instructions come in.


```python
random.seed(42)

class LazyStack:
    
    def __init__(self):
        self.stack = []
        self.incs = {}
        
    def push(self, v):
        self.stack.append(v)
        
    def pop(self):    
        length = len(self.stack)
        if length > 0:
            # clear top transaction and preserve its downward transactions by moving it one position down
            if self.incs.get(length):
                self.incs[length-1] = self.incs.get(length-1, 0) + self.incs.get(length)
                self.incs[length] = 0
            self.stack.pop()            

    def inc(self, i, v):
        self.incs[i] = self.incs.get(i, 0) + v
    
    def peek(self):
        """
        Note the ith_element means apply to indices [0: ith_element-1] inclusive of the stack
        """
        ith_element = len(self.stack)
        if ith_element > 0:
            return self.stack[-1] + self.incs[ith_element]
        else:
            return "EMPTY"
    
    def __len__(self):
        return len(self.stack)
    
    def __str__(self):
        return f"{self.stack}"
    
def do_it_better(number_of_instructions=50, print_func=(lambda x: x)):
    stack = LazyStack()
    
    for instruction in instructions(number_of_instructions):
        op = instruction.split()
        if op[0] == 'push':
            stack.push(int(op[1]))
        elif op[0] == 'pop':
            stack.pop()
        else:
            stack.inc(int(op[1]), int(op[2]))

        print_func(stack)
    
    
    # Now let's simulate a user 1) removing half of the stack and 2) examining the next top 3 set of elements
    for _ in range(int(len(stack)/2)):
        stack.pop()
    
    # peek and pop 3 elements
    for _ in range(3):
        print_func(stack.peek())
        stack.pop()

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

do_it_better(number_of_instructions=20000)       
```

    100 ms ± 4.47 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
    

### So, after 20000 randomized pop/push/inc instructions, the lazy stack speed up is 10 times faster over the baseline (0.1 s vs 1 s)! 😎
Most of the speedup come from not increment each and every value when the increment instruction is received, and postponing the increments until the data is actually read. Integrating the value increments so that just one addition is done on the read also saves time.

The `incs` dictionary stores minimal but precise information to make this work!
