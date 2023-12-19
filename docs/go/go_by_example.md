- a go string is read-only slice of bytes. (as containers of text encoded in UTF-8)
- values enclosed in single quotes are rune literals
- structs are mutable
- this works because if a variable is created inside a function and as the return value which will be given to the caller, go will allocate it on the heap, not the stack
    ```go
    func newPerson(name string) *person {
        p := person{name: name}
        p.age = 42
        return &p
    }
    ```
- interfaces are named collections of method signatures
- if ;
- timers are for when you want to do something once in the future, tickers are for when you want to do something repeatedly at regular intervals
- defer is used to ensure that a function call is performed later in a program's execution
- a recover can stop a panic from aborting the program and let it continue with execution instead