# Cascade operator proposal — native fluent interfaces
The cascades operator `..` allows you to make a sequence of operations on the same object which can dramatically reduce boilerplate. The operator implicitly discards the result and evaluates to the callee.

This simplistic example

    foo
        ..b = ""
        ..a()

Would be equivalent to

    foo.b = ""
    foo.a()

Or more accurately (to prevent `foo` from being referenced multiple times)

    (obj => {
        obj.b = ""
        obj.a()
    })(foo)

## Babel implementation

Here is a[ fork of babel](https://github.com/RedHatter/babel/tree/proposal-cascade-operator) with the cascade operator implemented. Note: The changes are on the `proposal-cascade-operator` branch.

Enable the plugin with `{ plugins: [ "@babel/plugin-proposal-cascade-operator" ] }`.

## Overview and motivation
The cascades operator `..` allows you to make a sequence of operations on the same object which can dramatically reduce boilerplate. The operator implicitly discards the result and evaluates to the callee. This allows most APIs to be used in a fluent manner without having been explicitly designed for it.

Due to the lack of language support, libraries such as jQuery have to use method chaining to implement a fluent interface and approximate cascading — this severely limits the API design by requiring methods instead of properties and requiring artificially flat class hierarchies.

### DOM manipulation

DOM manipulation is one area where the benefit of fluent interfaces can easily be seen.

Lets start with a simple example using standard methods.

    let node = document.querySelector("div#container")
    node.style.backgroundColor = "blue"
    node.classList.add("selected")
    node.addEventListener("click", e => {})
    let a = document.createElement("a")
    a.setAttribute("href", "http://example.com")
    a.text = "Example"
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

This approach comes at a cost. We lose the `style` and `classList` sub-objects and are forced to use methods instead of fields. Chaining APIs can quickly grow needlessly complex and become hard to reason about with additional methods required to navigate the statement stack. In addition chained APIs are often force to behave in unintuitive ways to maintain the requirement of always returning the caller.

The cascading operator gives us the benefits of a fluent interface without the concessions of method chaining. The interface can be used and designed in a traditional manner while still supporting fluent operation when desired.

    document.querySelector("div#container")
        ..style.backgroundColor = "blue"
        ..classList.add("selected")
        ..addEventListener("click", e => {})
        ..appendChild(document.createElement("a")
            ..setAttribute("href", "http://example.com")
            ..text = "Example"
        )

## Prior Art
The following languages implement the operator with the same general semantics as this proposal.

* Dart — [Cascade notation](https://www.dartlang.org/guides/language/language-tour#cascade-notation-)
* Smalltalk — message cascades

## References
* [original dart proposal](https://docs.google.com/document/d/1U0PeHtVQHMQ8usy7xI5Luo01W5LuWR1acN5odgu_Mtw/edit?pli=1#heading=h.tkyl552ayct9)
* [Fluent interfaces](https://en.wikipedia.org/wiki/Fluent_interface)