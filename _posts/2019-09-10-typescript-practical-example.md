---
title: Use React Context to DRY up your components
layout: post
type: post
date: 2019-09-03 00:00:01
---

In our previous [post](_posts/2019-09-03-react-usecontext-practical-example.md) we talked about the effects of plumbing and how it affects the readability of our entire codebase.

We came up with the idea of using React Context to avoid repeatedly passing down the same props to multiple layers of components, avoiding repeating prop types definitions.

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

essentially hiding the fact from the developer that both of these need a product to function.

Could we stay explicit while not having to repeat PropTypes? The answer is yes, by using TypeScript instead of PropTypes.
Of course TypeScript does a lot more than that, it adds types to JavaScript and now you can write strongly typed code, catch bugs before they are deployed, write code that's easier to read & maintain.

We'll be converting our sidebar & table example from the previous post, before we converted it to use React Context.
You'll find the sandbox here: https://codesandbox.io/s/table-sidebar-layout-with-typescript-2si7g
To get started with TypeScript in your project I recommend following one of the official guides: https://www.typescriptlang.org/samples/index.html. Here we'll jump straight into adding types to our app.

Good thing about adding TypeScript to your projects is that you can do it incrementally.
You can convert only components you want to typescript and leave the rest of the app as is.

We'll be converting only the Table component in this example:

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

Let's start by defining our first type: `Product`. When adding TypeScript support to an existing project you're looking for [Basic TypeScript types](https://www.typescriptlang.org/docs/handbook/basic-types.html) that could replace your PropTypes, essentially:

```
PropTypes.bool becomes boolean
PropTypes.number becomes number
PropTypes.string becomes string
PropTypes.arrayOf(PropTypes.number) becomes Array<number>
```
Use `any` to basically opt-out compile time checks for a variable.
Use void to indicate that a function returns "nothing".

So a function that accepts any parameter and returns nothing (could be a click handler) would look like this:

```
onClick: (event: any) => void
```

We know that these handlers are called with a specfic event type but we're going to talk about that later.

After having a basic understanding of what are the approriate TypeScript types for PropTypes we can define our Product type as:

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

Applying it to parameters is done in the fashion on `variable:type`. Don't forget to rename the files where you're introducing types to ts or tsx, from js or jsx:

![typescript into ts or tsx](https://i.imgur.com/OKZ63OX.png)

Now that we have the Product type we can replace the propTypes of the Table component.

```
type TableProps = {
  selectedProduct: Product;
  products: Array<Product>;
  onProductChange: (event: any) => void;
};
```

TypeScript also makes your editor smarter by giving cloues about where you could improve your code:

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

We can convert TableRow in the same fashion:

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
They enforce the the presence of certain (or all) properties or functions on the type that uses the interface.
Interfaces and types are somewhat interchangable but there are some restrictions, see:
 - https://stackoverflow.com/a/52682220/407466
 - https://medium.com/@martin_hotell/interface-vs-type-alias-in-typescript-2-7-2a8f1777af4c


We could define an interace which has this method and simple reuse that interface when we define TableRowProps and TableProps.

We'll be using [MouseEvent](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent) to describe the incoming event of our click handler:

```
export interface ProductChangeListener {
  onProductChange: (event: MouseEvent) => void;
}
```

you can implement interfaces with both types and other interfaces, only the syntax is different:

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

After update the `TableRowProps` accordingly the editor immeditelly warns us about an error we made:

![type error](https://i.imgur.com/NwzlVsD.png)

We defined our interface method to accept MouseEvent and not Product. To correct definition simply change `MouseEvent` to `Product` in the definition of your interface:

```
export interface ProductChangeListener {
  onProductChange: (event: Product) => void;
}
```

You'll find the complete example here: https://codesandbox.io/s/table-sidebar-layout-with-typescript-2si7g.

Did you find React Context useful in some other scenarios? Let me hear your story in the comment section below!

If you enjoyed this content, please show the love by sharing it with others. Thank you!