# Cascade operator proposal — native fluent interfaces
The cascade operator `..` allows you to make a sequence of operations on the same object which can dramatically reduce boilerplate. The operator continues execution until the next cascade or the end of the statement then implicitly discards the result and evaluates to the callee.

Take this simple example.

    foo
        ..a.b = c
        ..d()

Both `a.b = c` and `d()` are applied to the root object, `foo` in this case. Note that `d()` is applied to `foo` not to `c`.

This would be equivalent to

    foo.a.b = c
    foo.d()

Or more accurately (to prevent `foo` from being referenced multiple times)

    (obj => {
        obj.a.b = c
        obj.d()
    })(foo)

## Babel implementation

Here is a[ fork of babel](https://github.com/RedHatter/babel/tree/proposal-cascade-operator) with the cascade operator implemented. Note: The changes are on the `proposal-cascade-operator` branch.

Enable the plugin with `{ plugins: [ "@babel/plugin-proposal-cascade-operator" ] }`.

## Overview and motivation

Due to the lack of language support, libraries that desire to provide a fluent interface, such as jQuery, use method chaining to approximate cascading — this severely limits the API design by requiring methods instead of properties and requiring artificially flat class hierarchies.

The method chaining pattern has become more and more widespread since jQuery popularized it. It's quite clearly a pattern that many programmers want to use. Adding support for cascading at the language level gives the choice of whether or not to use a fluent interface pattern to the programmer. Library designers wouldn't be forced into deciding between providing a fluent interface and making concessions or providing a more classic API. It would open the design space allowing a fluent interface to be used when it makes sense and a more classic approach when it doesn't.

### DOM manipulation

DOM manipulation is one area where the benefit of fluent interfaces can easily be seen.

Let's start with a simple example using standard methods.

    let node = document.querySelector("div#container")
    node.style.backgroundColor = "blue"
    node.classList.add("selected")
    node.addEventListener("click", e => {})
    let a = document.createElement("a")
    a.setAttribute("href", "http://example.com")
    a.textContent = "Example"
    node.appendChild(a)

The same block of code written with a fluent interface is both more concise and more expressive. Here it is again written with jQuery's chaining approach. Notice how it doesn't require superfluous variables.

    $("div#container")
        .css({ backgroundColor: "blue" })
        .addClass("selected")
        .click(e => {})
        .append(
            $("<a>")
                .attr("href", "http://example.com")
                .text("Example")
        )

This approach comes at a cost. We lose the `style` and `classList` sub-objects and are forced to use methods instead of fields. Chaining APIs can quickly grow needlessly complex and become hard to reason about, with additional methods required to navigate the statement stack. In addition, chained APIs are often forced to behave in unintuitive ways to maintain the requirement of always returning the caller.

The cascading operator gives us the benefits of a fluent interface without the concessions of method chaining. The interface can be used and designed in a traditional manner while still supporting fluent operation when desired.

    document.querySelector("div#container")
        ..style.backgroundColor = "blue"
        ..classList.add("selected")
        ..addEventListener("click", e => {})
        ..appendChild(document.createElement("a")
            ..setAttribute("href", "http://example.com")
            ..textContent = "Example"
        )

## Prior Art
The following languages implement the operator with the same general semantics as this proposal.

* Dart — [Cascade notation](https://www.dartlang.org/guides/language/language-tour#cascade-notation-)
* Smalltalk — [message cascades](https://en.wikipedia.org/wiki/Method_cascading#Smalltalk)

## References
* [original dart proposal](https://docs.google.com/document/d/1U0PeHtVQHMQ8usy7xI5Luo01W5LuWR1acN5odgu_Mtw/edit?pli=1#heading=h.tkyl552ayct9)
* [Fluent interfaces](https://en.wikipedia.org/wiki/Fluent_interface)
