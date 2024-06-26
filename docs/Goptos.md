# Goptos documentation

> IMPORTANT: Goptos CLI, libs and docs are still under development!

- [Overview](#overview)
- [Setup](#setup)
- [Reactive system](#reactive-system)
    - [Components](#components)
    - [Signals and Effects](#signals-and-effects)
- [Flow control](#flow-control)
    - [If](#if)
    - [Each](#each)
    - [Nesting components](#nesting-components)
- [Goptos GLI](#goptos-cli)

## Overview

What is Goptos? - it's not built to be fast, it's defiantly not pretty - so what benefit does Goptos provide?

Goptos is intended to be a no-frills, 'get the job done' tool for quickly building demo applications.

It is component based and written in Go and embraces the idea that some teams prefer separation between frontend and backend; but at the same time, acknowledges that using a common language (preferably one that is strongly typed) can make development life easier. Not suggesting that sharing libs and actual type definitions between frontend and backend code is a new idea, but it sure is nice.

Goptos isn't the right choice if you want client side hydration from server side rendering, etc., again, not built for speed here. Instead choosing to focus on simplicity and helping to bring application ideas to fruition.

If you want to create a single-page-application that can interface with a backend API and you want to write it all in Go, then Goptos could be of interest to you.

## Setup

If you have made it this far then you have likely already created a new Goptos project and hopefully built the starter template. If not please follow the [Usage](../README.md#usage) instructions for initial setup.

## Reactive system

Goptos was inspired by Leptos and SolidJS, you might have guessed by it's name. To understand signals in a fine-grained reactive system there is a lot to learn from the hard work of those who are true masters of such a system. As it stands at the moment, this 'docs' will be quite anemic and has a lot to look up to!

### Components

Everything starts with an App component. Here is a 'Hello world' example.

``` Go
// src/comp/App/App.go

package App

import (
	"github.com/goptos/runtime"
	"github.com/goptos/system"
)

type Scope = runtime.Scope
type Elem = system.Elem

func View(cx *Scope) *Elem {
    var view *Elem
    return view
}

/* View
<p>Hello world</p>
*/
```

Let's break this down before making it reactive.

#### Code generation

Before looking at the anatomy of a component, it is important to know that a Goptos component cannot be built until it has undergone a pre-build step called **code generation**.

During code generation the `/* View` template (introduced soon) is parsed and turned into go code, which then replaces the `view` variable (introduced soon). Only after this has completed can a Goptos component be successfully built.

#### View function

Each component provides a View function that takes a reactive `Scope` and returns an `Elem`.

The View function defines a `view` variable which is directly returned. This variable is used during code generation (see above).

#### View template

A magic comment with the following structure.

``` Go
/* View
*/
```

> Could definitely use a light-weight LSP here, maybe adopting a proxy approach like templ's!

This magic comment is considered a View template which is read during the code generation.

In our above 'Hello world' example the template...

``` Go
/* View
<p>Hello world</p>
*/
```

...will generate the following code and update the view variable in our view function accordingly.

``` Go
func View(cx *Scope) *Elem {
    /* macro:generated:view:start */
    var view *Elem = (*Elem).New(nil, "p").Text("Hello").Text(" ").Text("world")
    /* macro:generated:view:end */
    return view
}
```

A View template can only contain a single root element. When displaying multiple elements a surrounding parent `<div>` is needed, as you will see later.

#### Creating new components

Here is an example of a new Button component. We will expand on this more later.

``` Go
// src/comp/Button/Button.go

package Button
...
```

Remember, new components must:
- Use a capital first letter.
- Exist within the comp directory.
- Share the same name for the package name, '.go' file and containing package directory.
- Have a View function.

### Signals and Effects

Now that we understand what a Goptos component looks like and learned about code generation, we can start making things reactive.

At the heart of the reactive system are Signals and Effects. In practice you will generally only create Signal, Effects are present but these will be generated for you based on how you use signals within your View template.

#### Signals

Let's add new signal to our 'Hello world' component.

``` Go
import "github.com/goptos/runtime"

type SignalInt = runtime.Signal[int]

func View(cx *Scope) *Elem {
    var count = (*SignalInt).New(nil, cx, 0)
    ...
}
```

> Signals can take the form of any type, even custom structs defined by you.

#### Derived signals

A derived signal is simply a function that uses another signal's `Get()` method.

``` Go
    ...
    var doubleCount = func() int { return count.Get() * 2 }
    ...
```

#### Using Signals in a View template

Simply surround your signal with code braces `{...}` to use it in a View template.

``` Go
/* View
<div>
    <p>Hello world</p>
    <p>{count.Get()}</p>
    <p>{doubleCount}</p>
</div>
*/
```

## Flow control

At this time there is limited flow control features, however a lot can be done with simple If and Each logic.

### if

Add the `if` attribute to any element and provide it with a function which returns a `bool`.

``` Go
/* View
<div if={ func() bool {...} }></div>
*/
```

In practice is it common to use a variable for the function as it makes reading the View template a lot easier.

``` Go
var showF = func() bool {
    if count.Get() > 2 {
        return true
    } else {
        return false
    }
}
```
``` Go
/* View
<div if={ showF }>I only show for 2 or more.</div>
*/
```

In this example the `if` function references a Signal. In doing so this causes the reactive system to register a new Effect against `count` making that element reactive to changes of `count`.

### each

Add the `each` attribute to any element and provide it with two functions. A collect function (cF) and a key function (kF). A child component must be used within the attributed element to process each item.

``` Go
/* View
<ul each={ func() []T {...} } key={ func(T) int {...} }>
    <ChildComp />
</ul>
*/
```

To help explain the following examples, let's create a custom type. In practice this will be some type that represents the data you are working with.

``` Go
type SomeRecordType struct {
	Id   int
	Name string
}
```

The important part to focus on is that there is an Id property on this type.

#### Collect function

The collect function must return an array of objects. They can be of any type and in this, contrived, example we will simply generate an array of our custom type defined above.

``` Go
var cF = func() []SomeRecordType {
    var arr = []SomeRecordType{}
    for i, v := range []string{"Border Collie", "German Shepherd", "Golden Retriever"} {
        arr = append(arr, SomeRecordType{Id: i, Name: v})
    }
    return arr
}
```

#### Key function

The key function must take an item of the type returned by the collect function and return an `int`. This function is used to identify the unique key what will be used when the reactive system looks to determine if an item needs to be added or removed.

``` Go
var kF = func(item SomeRecordType) int {
    return item.Id
}
```

#### Child component

The child component is used to process and display each item. The child component must take an additional parameter of the appropriate type.

You do not need to pass the current item to the child component from the parent component, this is assumed and done for you during code generation.

In our example the child component looks like this.

``` Go
// src/comp/Li/Li.go

package Li

import (
	"github.com/goptos/runtime"
	"github.com/goptos/system"
)

type Scope = runtime.Scope
type Elem = system.Elem

type SomeRecordType struct {
	Id   int
	Name string
}

func View(cx *Scope, item SomeRecordType) *Elem {
    var view *Elem
	return view
}

/* View
<li>{item.Name}</li>
*/
```

The parent component looks like this.

``` Go
// src/comp/App/App.go

package App

import (
	"github.com/goptos/app/comp/Li"
    "github.com/goptos/runtime"
	"github.com/goptos/system"
)

type SomeRecordType = Li.SomeRecordType
type Scope = runtime.Scope
type Elem = system.Elem

func View(cx *Scope) *Elem {
    var cF = func() []SomeRecordType {
        var arr = []SomeRecordType{}
        for i, v := range []string{"Border Collie", "German Shepherd", "Golden Retriever"} {
            arr = append(arr, SomeRecordType{Id: i, Name: v})
        }
        return arr
    }
    var kF = func(item SomeRecordType) int {
        return item.Id
    }
    var view *Elem
	return view
}

/* View
<ul each={cF} key={kF}>
	<Li />
</ul>
*/
```

We have also just demonstrated here how to use a custom component. It is worth explaining this more explicitly though.

### Nesting components

When creating a new component you are creating a new variable scope, not a new reactive scope. The reactive scope of each parent is passed into each child component for you during code generation.

You will need to consider passing other variables, including Signal variables to a component as well as how to import the component for the parent.

Let's start by creating a new Button component and see how to use it. Our Button component looks like this.

``` Go
// src/comp/Button/Button.go

package Button

import (
	"github.com/goptos/runtime"
	"github.com/goptos/system"
)

type Scope = runtime.Scope
type IntSignal = runtime.Signal[int]
type Elem = system.Elem
type Event = system.Event

func View(cx *Scope, parentCount IntSignal) *Elem {
	var count = (*IntSignal).New(nil, cx, 0)
	var clickF = func(Event) {
		count.Set(count.Get() + 1)
		parentCount.Set(parentCount.Get() + 1)
	}
    var view *Elem
	return view
}

/* View
<div>
	<button on:click={clickF}>Child Add</button>
	<p>Child Count {count.Get()}</p>
</div>
*/
```

#### Importing

As the Go dev tools tend to remove unused imports we can prefix the imported component with an underscore (`_`) and tag it with another magic comment `/* macro:import */`.

This will indicate we want to add this import during code generation.

``` Go
// src/comp/App/App.go

package App

import (
	_ "github.com/goptos/app/comp/Button" /* macro:import */
	"github.com/goptos/runtime"
	"github.com/goptos/system"
)
...
```

To use the component specify it in the View template using a self closing tag.

``` Go
/* View
<div>
    <Button />
</div>
*/
```

#### Passing variables

To pass a variable to a child component simply provide the variable name (without enclosing it in any brackets) within the component self closing tag.

``` Go
/* View
<div>
    <Button count />
</div>
*/
```

> If passing multiple variables, pass them space separated and in argument order (as defined by the imported components View function).

The parent component for this example would look like this.

``` Go
// src/comp/App/App.go

package App

import (
	_ "github.com/goptos/app/comp/Button" /* macro:import */
	"github.com/goptos/runtime"
	"github.com/goptos/system"
)

type Scope = runtime.Scope
type SignalInt = runtime.Signal[int]
type Elem = system.Elem
type Event = system.Event

func View(cx *Scope) *Elem {
    var count = (*SignalInt).New(nil, cx, 0)
    var clickAddF = func(Event) {
		count.Set(count.Get() + 1)
	}
	var clickSubF = func(Event) {
		count.Set(count.Get() - 1)
	}
    var view *Elem
	return view
}

/* View
<div>
	<button type="button" on:click={clickAddF}>Parent Add</button>
	<button type="button" on:click={clickSubF}>Parent Sub</button>
    <p>Parent Count {count.Get()}</p>
    <Button count />
</div>
*/
```

## Goptos CLI

``` bash
goptos
    init [-version <string default: latest>]
    genview [-src <string default: .>]
    build [-src <string default: src>]
    package [-dist <string default: dist>]
    serve [-src <string default: src> -dist <string default: dist> port <int default: 8080>]
```
