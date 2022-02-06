# The Uniform Access Principle

2020-11-15

In languages that support them, properties typically have a lot of syntax support for their declaration and definition.

One way to avoid this is the "<a href="https://en.wikipedia.org/wiki/Uniform_access_principle">Uniform Access Principle</a>": 
Just make field read access and no-argument function calls indistinguishable.

So `foo.bar` could mean access to the field `bar` or an invocation of the no-argument `bar` function, depending on what's declared on foo.

This has one obvious downside:  If `bar` is a function, it's no longer possible to distinguish the invocation 
`foo()` from the function reference `foo`. However, this can be fixed by adding specific syntax for this (less common) case. 
This might be better for readability in any case.

Similarly, `foo.bar = x` write access could be equivalent to `foo.set_bar(x)`.

## Advantages of this approach seem to be:

- Whether something is a field or a no-arg function becomes an implementation detail hidden from the exposed API.
  This means that simple properties can later be changed without client changes. 
- No extra syntax is needed for declaring properties. Just by convention, no-arg functions can be accessed like properties and `set_` functions 
  implicitly can be called via the assignment operator. If there is a setter and a field of the same name, the setter can take precedence 
  (except inside the setter declaration).

The Uniform Access Principle was initially proposed by Bertrand Meyer and implemented in Eiffel and I am supporting a similar 
design in <a href="http://tantilla.org/">tantilla.org</a>.</p>

