# Tool Calling

## Problem Statement

Context management and execution of tool calls are inextricably linked, because most
models expect the output of a tool call to be added to the context at the
moment when the too lcall happens. This means that the `Session` must own the
code that actual performs a tool call.

This is annoying because *what to do with a tool call* -- or even *how to
implement a tool call* -- is going to vary from application to application.

We are then faced with two options:

1. Provide some sort of object protocol for handling tool calls, whereby the
   client responsible for tool calling is also responsible for executing a
   callback on the session which appropriately modifies the session's context
   in light of the tool response; or,
2. Come up with a small number of ways in which a tool may be called, and
   expose those in the session. Anyone who wants to do something more complex
   must then extend the Session class and implement their own too lcalling
   logic.

## Proposals


### Tool Calling Protocol Option

Basically (2).

Certain things such as `transform` have a default semantics in the
`MelleaSession` base class. 

For anyone who wants to do free-form tool calling,
there is a `MelleaSessionToolProtocol` mixin which must be inherited from and
implemented.

### Nothing Fancy Option

Pass back the `ModelOutputThunk` with tool calls, and do nothing else.

Note that we already have a `ctx.insert` function, si instead of a mixin with
a protocol, the user is just supposed to know what they are supposed to do and
then use `m.ctx.insert` to implement the relevant logic.

This is what's done with openai sdk in the status quo anyways.

### Compromise?

Can this be implemented such that if you don't specify a tool calling protocol
implementation then the behavior is equivalent to the Nothing Fancy Option?
Probably so.


## Final Proposal

The ModelOutputThunk has a `tools` field where parsed tool calls are surfaced
to the user. This already exists and probably does not need additional
modification.

1. For certain special tool calling protocols, the Session handles things
   automatically for the user. E.g., `m.transform` and `m.query`. We need to
   specify the precise semantics for what happens when a user provides tools
   in the model_options when using `m.transform` -- probably, you flow through
   into the next two cases.
2. If the `Session` has a `SessionToolCallingProtocol` implemented, then the
   `def tool_call_result(...)` on that protocol must be called by the user
   after a tool is executed. When that method is called, the context is
   updated appropriately. We can also provide a `def call_tool(tool)` method
   for convienance, which does both the tool call and the context management
       for the user.
3. Otherwise, nothing happens. The user is responsible for updating their
   context as needed.
