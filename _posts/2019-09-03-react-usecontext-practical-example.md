---
title: Use React Context to DRY up your components
layout: post
type: post
date: 2019-09-03 00:00:01
---

## aka. How to reduce plumbing in React apps?

Plumbing in programming is something you have to do to make things work. It's sometimes repetitive or looks like a boilerplate you would usually remove. It's also crucial because it connects different layers of the application. In this post, I'll show you a typical example of plumbing inside a React application and a way to fix it.

Let's take a look at a pretty standard, table-sidebar layout:

<script src="https://gist.github.com/akoskm/71c234a90dc39823eacd3bb6426164e4.js?file=index.jsx "></script>

You can find the complete project [here](https://codesandbox.io/embed/table-sidebar-layout-with-plumbing-xk591?fontsize=14), without using hooks (this is not the recommended approach).

The table contains a list of products. When one of them, is clicked, the sidebar shows the details of the product and highlights the row clicked. Both `Table` and `Sidebar` need to know about the selected `product`.

`Table` needs to highlight the row:

<script src="https://gist.github.com/akoskm/71c234a90dc39823eacd3bb6426164e4.js?file=table.jsx "></script>

and `Sidebar` displays additional information about the selected product:

<script src="https://gist.github.com/akoskm/71c234a90dc39823eacd3bb6426164e4.js?file=sidebar.jsx "></script>

Later we decide to extend the sidebar with some sections:

<script src="https://gist.github.com/akoskm/71c234a90dc39823eacd3bb6426164e4.js?file=sidebar-v2.jsx "></script>

As the application grows, we create wrapper components like `Sidebar`. These components don't immediately use `product`, only pass it down to child components.


## React.useContext

[`useContext`](https://reactjs.org/docs/hooks-reference.html#usecontext) is one of the three basic React hooks.

The great thing about Contexts is that the component subscribed to it automatically re-renders when the Context value changes. Our Table/Sidebar layout sounds like an ideal use case for this.

Creating a context is as simple as:

<script src="https://gist.github.com/akoskm/71c234a90dc39823eacd3bb6426164e4.js?file=table-context.jsx "></script>

and to subscribe to it:

<script src="https://gist.github.com/akoskm/71c234a90dc39823eacd3bb6426164e4.js?file=subscribe-to-context.jsx "></script>

Use `<TableContext.Provider>` to set the `value` of the context. We wrap both `Table` and `Sidebar` components with this component because we want both of them to update when the context value changes:

<script src="https://gist.github.com/akoskm/71c234a90dc39823eacd3bb6426164e4.js?file=index-new.jsx "></script>

Now instead of passing the selected product to the Sidebar component, we set it as the current value of `<TableContext.Provider>`. Finally, we subscribe `Sidebar` to context updates:

<script src="https://gist.github.com/akoskm/71c234a90dc39823eacd3bb6426164e4.js?file=sidebar-new.jsx "></script>

The use of context allowed us to remove the product prop from the `Sidebar` component.
We see the real benefit of `useContext` as our application grows. Earlier we presented how the `Sidebar` would look like after some development and how different Sidebar panels would need a reference to the current product. With the use of React.useContext passing down this selection through multiple layers of components is not required anymore. We subscribe to product updates with each panel as we did above with the `Sidebar` and this simplifies the component:

<script src="https://gist.github.com/akoskm/71c234a90dc39823eacd3bb6426164e4.js?file=sidebar-v2-new.jsx "></script>

See this code in action [here](https://codesandbox.io/embed/table-sidebar-layout-with-usecontext-os7wr?fontsize=14).

Another benefit of not passing down the same object to multiple components is avoiding repeated PropTypes declarations. Googling for "react PropTypes repeated" this looks like a problem React developers encounter. With the above approach, there is one place left to define the shape of the product:

<script src="https://gist.github.com/akoskm/71c234a90dc39823eacd3bb6426164e4.js?file=table-context-proptypes.jsx "></script>

Did you find React Context useful in some other scenarios? Let me hear your story in the comment section below!

If you enjoyed this content, please show the love by sharing it with others. Thank you!