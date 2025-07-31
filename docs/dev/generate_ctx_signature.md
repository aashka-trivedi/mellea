# Splitting the `head` and `tail` of the Context on generate calls

We have decided to split the context into an "action" and "the rest of the context"; i.e., instead of `generate : ctx, ... -> output`, we use `generate: action, ctx, ... -> output`.

 This "car/cdr" separation of the final element from the rest is done because there are many situations where many different requests are made over the same context. Examples include multiple requirement checking, rejection sampling, and so on.

Advantages of this approach:
    * shared context is referentially equal, which makes memory management extremely simple.
    * Certain types of code -- especially requirement checking -- are much easier to write. Because the Context does not have to be deep-copied.
    
Disadvantages of this approach:
    * This solution is extremely specific to a few examples/patterns from stdlib. When we have `span`-based backends, there could be many different points in the span from which generation could continue. The solutino to that problem will sort of rhyme -- separating the generation target from th rest of the context.t However, the current signature is NOT a good solution. So it's possible we will have to change how this works in the fture.
    * Not parsimonious with how context is normally used, and perhaps confusing, particularly in the most-common situation whwere the context is "just" a normal chat history.
    * It is not yet clear what meaning this will have when contexts cannot be linearized. In particular: what if there's a poset and multiple generation opportunities within that poset? How do we "place the cursor"? Does this design choice make it harder to "place the cursor"?
    * Contexts are not in fact immutable, so we have to be extremely careful about when a context gets modified, and may even need to introduce semaphores.
    
