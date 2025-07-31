# Mellea should be as close to a library as possible

We should make it possible to use mellea as a library (as opposed to a framework).

In the context of LLM applications, the library vs framework distinction really boils down to how you treat the backend.

If a piece of software insists on having an exclusive handle on the backend, then that piece of software does nto compose with any other piece of software that also insists on an exclusive handle. They both want to be privileged with respect to the backend, so they cannot "play well" together. The `outlines` library is a good example of software that could've been a library but instead acts like a framework. Even `granite-io` takes on a framework-like role when it decides to actually call the backend, as opposed to operating over strings (or perhaps chat histories).

Writing LLM libraries is kind of difficult. There is a very strong instinct to try to grab control of the backend. Mellea is no exception. In the "intro path", mellea definitely behaves like a framework. We hide the actual backend objects (`PretrainedModel`, `openai.Client`, etc.) from the user.

But should try to make it easy for certain parts of mellea to be used as a library. There are many ways in which we could allow mellea to compose with other librares:

1. We could have a `m.start_session_with_shared_backend(client:openai.Client)` and similarly for local ollama models and transformers models. Everything would work mostly the same after that, except we would have to make much weaker assumptions about the state of the backend (e.g., cache and LoRAs).
2. We could strive to keep the `Formatter` logic completely separate from Backend-specific code, and the legacy model behavior should treat each Component like a standalone user message. This way people could use `mellea` components without using the `mellea` backend and context managemetn code.
3. We could trive to keep the `Cache` strategies agnostic to the rest of the code base, and figure out what their interface should be with respect to various backend sdks (and transformers in particular)
