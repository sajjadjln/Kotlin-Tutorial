# what is suspend function

### **How Suspend Functions Work**

### **What is a Suspend Function?**

- **Definition**: A suspend function is a function:
    - Declared with the `suspend` keyword.
    - Is asynchronous and might be long-running.
    - Introduces the concept of suspension points, where execution pauses and resumes later.

### **How Suspend Functions Work**

1. Suspend functions are logically callbacks
2. Code gets translated into a continuation style by the compiler.

**Continuation-Passing Style (CPS)**:

- Compiler transforms suspend functions into CPS.
    - Adds a **Continuation Object** as an extra parameter.
    - Converts normal code into a **giant switch statement** using labels to track execution points.

what is this label ⇒ inside the big switch case there is the function (our callback) but with new parameter (state model) that has all the state we need to carry on calling the parent function

we have MyStateModel  is a type of continuation which is implementing the interface of continuation that has a resume function 

so when a suspend function is finished and want to resume the coroutine this function is called.

so suspend function should have this hidden compiler generated parameter when its called thats why we cant call them from normal code because we need to provide that **Continuation Object** 

using coroutine builders which have this context that allows suspend functions to be called

### **Types of Coroutine Builders**

1. **`runBlocking`**:
    - **Behavior**: Blocks the current thread while running a coroutine.
    - **Uses**:
        - Entry point in the `main` function.
        - Testing suspend functions.
        - Rare cases, e.g., when a custom thread needs to call a suspend function.
    - **Caution**: Avoid excessive use as it blocks threads.
2. **`launch`**:
    - **Behavior**: Launches a coroutine asynchronously (non-blocking).
    - **Characteristics**:
        - Does not return a value (fire-and-forget).
    - **Common Use**: Most frequently used coroutine builder.
3. **Other Builders** (to be discussed later):
    - `async`
    - `coroutineScope`

what the label does???

---

`runBloccking` is a top level coroutine builder so we dont need scopes to call it

so one way of using `runBloccking` is being entry point of the function

- `runBloccking` creates a coroutine scope
- use runBlocking on main instead of suspend