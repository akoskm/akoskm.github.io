---
title: Use TypeScript to DRY up React components
layout: post
type: post
date: 2019-09-12 00:00:01
featured: yes
excerpt: A practical guide to replacing React PropTypes with TypeScript.
redirect_to: https://akoskm.com/typescript-practical-example-prop-type
redirect_from:
 - /2019/09/12/typescript-practical-example-prop-type.html
---

{: .no_toc}

* TOC
{:toc}

# Introduction

In the [previous post](/blog/react-usecontext-practical-example), we talked about ”plumbing” in React and how it affects the readability of our code.

We used React Context to avoid passing down the same props to multiple layers of components and to avoid repeating PropTypes definitions.

While React Context did eliminate the repetitions in our code, it introduced a different problem.

Our components became less explicit:

<script src="https://gist.github.com/akoskm/69a69d6668246db3464ab01c0361d689.js?file=sidebar.jsx "></script>

hiding the fact that both of these need some data in order to work. We could argue that not passing down props negatively affects the readability and maintainability of this code.

So I asked myself, could we stay explicit without repeating PropTypes? The answer is yes, by using TypeScript to describe the props of our components.

Of course, TypeScript does a lot more than that. It adds types to JavaScript, which makes your code more readable and easier to maintain. It makes some editors smarter, enhances autocomplete features, and shows information about the TypeScript types you're using and many more.

We'll work on the [sidebar-table app](https://codesandbox.io/s/table-sidebar-layout-without-usecontext-xk591) from the previous post, which still has the prop repetitions.
To get started with TypeScript in your project, I recommend following one of the [official guides](https://www.typescriptlang.org/samples/index.html). Here we'll jump straight into adding types to the app.

One of the things I like about TypeScript is that you can add it incrementally to your project.
You can introduce TypeScript only to some parts of your codebase and leave the rest of the app untouched.

Having that in mind, first, we'll convert the `Table` component which currently looks like this:

<script src="https://gist.github.com/akoskm/69a69d6668246db3464ab01c0361d689.js?file=table.jsx"></script>

We immediately notice a few things:
 - `products` has the same props as `selectedProduct` wrapped in an array
 - the `isSelected` function receives two products but there is no indication of that

Let's start by defining our first type: `Product`.

# TypeScript types

When rewriting existing, PropTypes-based components to TypeScript we're looking for [basic TypeScript types](https://www.typescriptlang.org/docs/handbook/basic-types.html) that could replace PropTypes, but essentially:

**PropTypes.bool** ➡ **boolean**.

**PropTypes.number** ➡ **number**.

**PropTypes.string** ➡ **string**.

**PropTypes.arrayOf(PropTypes.number)** ➡ **Array\<number\>**.

Use `any` to opt-out compile time checks for a variable.

Use `void` to indicate that a function returns undefined.

A function that accepts any parameter and returns undefined (could be a click handler) would look like this:

<script src="https://gist.github.com/akoskm/69a69d6668246db3464ab01c0361d689.js?file=function.ts"></script>

Yeah, I know, such handlers are called with a specific event type, but we're going to talk about that later.

After having a basic understanding of which TypeScript types could replace certain PropTypes, we can define Product as:

<script src="https://gist.github.com/akoskm/69a69d6668246db3464ab01c0361d689.js?file=types.ts"></script>

Don't forget to rename the files when you're adding types, js to ts and jsx to tsx:

![typescript into ts or tsx](https://i.imgur.com/OKZ63OX.png)

Now that we have the Product type ready, we can replace the `propTypes` part of the `Table` component:

<script src="https://gist.github.com/akoskm/69a69d6668246db3464ab01c0361d689.js?file=table-props.ts"></script>

TypeScript also makes your editor smarter by giving clues about where you could improve your code:

![type suggestion](https://i.imgur.com/9ALoIYW.png)

# Components with TypeScript

Finally, our `Table` component looks like this:

<script src="https://gist.github.com/akoskm/69a69d6668246db3464ab01c0361d689.js?file=table.tsx"></script>

Let's convert `TableRow` in the same fashion:

<script src="https://gist.github.com/akoskm/69a69d6668246db3464ab01c0361d689.js?file=table-row.tsx"></script>

see <a href="https://codesandbox.io/embed/table-sidebar-layout-with-plumbing-xk591?fontsize=14">here</a> the original version.

# TypeScript Interfaces

We notice the repetition of the `onProductChange: (event: any) => void;` function in `TableProps` and `TableRowProps`, and we're going to use [Interfaces](https://www.typescriptlang.org/docs/handbook/interfaces.html) to solve this problem.

Interfaces are constraints between the interface and the implementing type.
They make sure that all (or some) properties or functions are present on the type that uses the interface.

Interfaces and types are mostly interchangeable, but there are some restrictions:
 - [Typescript: Interfaces vs Types](https://stackoverflow.com/a/52682220/407466)
 - [Interface vs Type alias in TypeScript 2.7](https://medium.com/@martin_hotell/interface-vs-type-alias-in-typescript-2-7-2a8f1777af4c)

We can create an interface with a method and share it between `TableRowProps` and `TableProps`.

We'll use [MouseEvent](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent) to describe the incoming event of our click handler (*Spoiler alert*: this declaration is wrong, but TypeScript will warn us 🎉):

<script src="https://gist.github.com/akoskm/69a69d6668246db3464ab01c0361d689.js?file=interface-v1.ts"></script>

Both types and interfaces can extend other interfaces, but the syntax is different:

 - types extending interfaces:
<script src="https://gist.github.com/akoskm/69a69d6668246db3464ab01c0361d689.js?file=type-and-interface.ts"></script>

 - interfaces extending interfaces
<script src="https://gist.github.com/akoskm/69a69d6668246db3464ab01c0361d689.js?file=interface-and-interface.ts"></script>

# Catching early bugs

After updating `TableRowProps`, the editor immediately warns us about the error we made:

![type error](https://i.imgur.com/NwzlVsD.png)

We defined our interface method to accept `MouseEvent` instead of `Product`. Let's fix the interface declaration:

<script src="https://gist.github.com/akoskm/69a69d6668246db3464ab01c0361d689.js?file=interface-v2.ts"></script>

We successfully cleaned up these two components from PropTypes repetitions, and you can find the updated code [here](https://codesandbox.io/s/table-sidebar-layout-with-typescript-2si7g).

Do you use TypeScript? Do you think this introduces more complexity than PropTypes?

Let me hear your thoughts on this in the comment section!