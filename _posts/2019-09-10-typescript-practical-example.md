---
title: Use React Context to DRY up your components
layout: post
type: post
date: 2019-09-03 00:00:01
---

In our [previous post](_posts/2019-09-03-react-usecontext-practical-example.md) we talked about the effects of plumbing in programming and how it affects the readability of our entire codebase.

We used used React Context to avoid passing down the same props to multiple layers of components, this way avoiding repeating prop types definitions.

While React Context did eliminate the repetitinons in our code, it introduced a different problem.

Our components became less explicit:

```
const Sidebar = () => (
  <div className="sidebar">
    <DetailsPanel />
    <DimensionsPanel />
  </div>
);
```

hiding the fact from the developer that both of these need a product to work. We could argue that this negatively affects readability and maintenance of this code.

Could we stay explicit while also not having to repeat PropTypes? The answer is yes, by using TypeScript instead of PropTypes.

Of couse, TypeScript does a lot more than that. It adds types to JavaScript which makes your code more readable and easier to maintain. It makes some editors more smarter, enabling autocomplete and showing information about the TypeScript you're writing.

We'll convert our sidebar-table example from the previous post. This is the original version which contains the prop repetitions: https://codesandbox.io/s/table-sidebar-layout-with-typescript-2si7g
To get started with TypeScript in your project I recommend following one of the official guides: https://www.typescriptlang.org/samples/index.html. Here we'll jump straight into adding types to our app.

Good thing about adding TypeScript to your project is that you can do it incrementally.
You can introduce TypeScript only to parts of your application while leaving the rest of the app untouched.

We'll be converting only the Table and TableRow components in this example:

```
import React from "react";
import PropTypes from "prop-types";
import TableRow from "./table-row";

const isSelected = (currProduct, selectedProduct) =>
  selectedProduct && selectedProduct.id === currProduct.id;

const Table = ({ selectedProduct, products, onProductChange }) => (
  <table>
    <thead>
      <tr>
        <th>ID</th>
        <th>Name</th>
        <th>SKU</th>
      </tr>
    </thead>
    <tbody>
      {products.map(currProduct => (
        <TableRow
          selected={isSelected(currProduct, selectedProduct)}
          key={currProduct.id}
          product={currProduct}
          onProductChange={onProductChange}
        />
      ))}
    </tbody>
  </table>
);
Table.propTypes = {
  selectedProduct: PropTypes.shape({
    id: PropTypes.number.isRequired,
    name: PropTypes.string.isRequired,
    sku: PropTypes.string.isRequired,
    dimensions: PropTypes.shape({
      x: PropTypes.number.isRequired,
      y: PropTypes.number.isRequired,
      z: PropTypes.number.isRequired
    })
  }),
  products: PropTypes.arrayOf(
    PropTypes.shape({
      id: PropTypes.number.isRequired,
      name: PropTypes.string.isRequired,
      sku: PropTypes.string.isRequired,
      dimensions: PropTypes.shape({
        x: PropTypes.number.isRequired,
        y: PropTypes.number.isRequired,
        z: PropTypes.number.isRequired
      }).isRequired
    }).isRequired
  ),
  onProductChange: PropTypes.func.isRequired
};
export default Table;

```

We immediatelly notice a few things:
 - `products` array has the same props like selectedProduct, they're just wrapped in an array
 - the `isSelected` function receives two products but there is no indication of that

Let's start by defining our first type: `Product`.

When rewriting your existing PropTypes-based components to TypeScript you're looking for [Basic TypeScript types](https://www.typescriptlang.org/docs/handbook/basic-types.html) that could replace your PropTypes, but essentially:

**PropTypes.bool** _becomes_ **boolean**.

**PropTypes.number** _becomes_ **number**.

**PropTypes.string** _becomes_ **string**.

**PropTypes.arrayOf(PropTypes.number)** _becomes_ **Array<number>**.

Use `any` to basically opt-out compile time checks for a variable.
Use void to indicate that a function returns "nothing".

A function that accepts any parameter and returns undefined (could be a click handler) would look like this:

```
onClick: (event: any) => void
```

Yeah, I know, such handlers are called with a specfic event type but we're going to talk about that later.

After having a basic understanding of what are the appropriate TypeScript type euivalents for PropTypes we can define our Product as:

```
export type Product = {
  id: number;
  name: string;
  sku: string;
  dimensions: {
    x: number;
    y: number;
    z: number;
  };
};
```

Decorating a parameter with a certain type is done in the following way: `variable:type`. Don't forget to rename the files where you're introducing types to ts or tsx, from js or jsx:

![typescript into ts or tsx](https://i.imgur.com/OKZ63OX.png)

Now that we have the Product type we can replace the `propTypes` part of the Table component.

```
type TableProps = {
  selectedProduct: Product;
  products: Array<Product>;
  onProductChange: (event: any) => void;
};
```

TypeScript also makes your editor smarter by giving clues about where you could improve your code:

![type suggestion](https://i.imgur.com/9ALoIYW.png)

Finally our Table component looks like this:

```
import React from "react";
import TableRow from "./table-row";
import { Product } from "./types";

const isSelected = (currProduct: Product, selectedProduct: Product) =>
  selectedProduct && selectedProduct.id === currProduct.id;

type TableProps = {
  selectedProduct: Product;
  products: Array<Product>;
  onProductChange: (event: any) => void;
};

const Table = ({ selectedProduct, products, onProductChange }: TableProps) => (
  <table>
    <thead>
      <tr>
        <th>ID</th>
        <th>Name</th>
        <th>SKU</th>
      </tr>
    </thead>
    <tbody>
      {products.map((currProduct: Product) => (
        <TableRow
          selected={isSelected(currProduct, selectedProduct)}
          key={currProduct.id}
          product={currProduct}
          onProductChange={onProductChange}
        />
      ))}
    </tbody>
  </table>
);
export default Table;
```

Let's convert TableRow in the same fashion:

```
import React from "react";
import { Product } from "./types";

type TableRowProps = {
  product: Product;
  onProductChange: (event: any) => void;
  selected: boolean;
};

const TableRow = ({ product, onProductChange, selected }: TableRowProps) => {
  return (
    <tr
      className={selected ? "selected" : undefined}
      key={product.id}
      onClick={() => {
        onProductChange(product);
      }}
    >
      <td>{product.id}</td>
      <td>{product.name}</td>
      <td>{product.sku}</td>
    </tr>
  );
};
export default TableRow;
```

We notice that we repeated the definition of the `onProductChange: (event: any) => void;` function.

This is where TypeScript interfaces come into play.

Interfaces as constraints between the interface and the imlementing type.
They enforce the presence of certain (or all) properties or functions on the type that uses the interface.

Interfaces and types are somewhat interchangable but there are some restrictions, see:
 - https://stackoverflow.com/a/52682220/407466
 - https://medium.com/@martin_hotell/interface-vs-type-alias-in-typescript-2-7-2a8f1777af4c


We could define an interace that has this method and simple reuse it when we define TableRowProps and TableProps.

We'll be using [MouseEvent](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent) to describe the incoming event of our click handler:

*Spoiler alert*: this declaration is wrong but TypeScript will warn us! 🎉

```
export interface ProductChangeListener {
  onProductChange: (event: MouseEvent) => void;
}
```

Both types and interfaces can extend other interfaces, only the syntax is different:

 - types implementing interfaces:
```
type TableRowProps = ProductChangeListener & {
  product: Product;
  selected: boolean;
};
```

 - interfaces implementing interfaces
```
interface TableRowProps extends ProductChangeListener {
  product: Product;
  selected: boolean;
}
```

After updating the `TableRowProps` the editor immeditelly warns us about an error we made:

![type error](https://i.imgur.com/NwzlVsD.png)

We defined our interface method to accept `MouseEvent` instead of `Product`. Let's fix the interface declaration:

```
export interface ProductChangeListener {
  onProductChange: (event: Product) => void;
}
```

This is how we cleaned up our two components that previously had PropTypes repetitions. You'll find the complete example here: https://codesandbox.io/s/table-sidebar-layout-with-typescript-2si7g.

Are you already using TypeScript? I think it adds slightly more complexity to your code than PropTypes but has much more benefits. Let me hear your story in the comment section below!

If you enjoyed this content, please show the love by sharing it with others. Thank you!