# React.useContext to reduce plumbing

Plumbing in programming is something you have to do to make things work. It's sometimes repetitive, it looks like a boilerplate you would normally remove, but it also crucial because it connects different layers of your application. In this post, I'll show you a classic example plumbing inside a React application.

Let's take a look at a pretty common, table-sidebar layout:

    <Table
      selectedProduct={product}
      products={products}
      onProductChange={handleProductChange}
    />
    <Sidebar product={product} />

You can find the complete, working project here: https://codesandbox.io/embed/table-sidebar-layout-with-plumbing-xk591?fontsize=14.

The table contains a list of products. When one of them is clicked the sidebar shows the details of that product and also highlights the row that was clicked. We see that both `Table` and `Sidebar` needs to know what is the currently selected `product`.

`Table` needs it for the highlight:

    const Table = ({ selectedProduct, products, onProductChange }) => (
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
    );
    export default Table;

and `Sidebar` needs to display additional information about the selected product:

    const Sidebar = ({ product }) => (
      <div className="sidebar">
        <h3>{product.name}</h3>
        <p>ID: {product.id}</p>
        <p>SKU: {product.sku}</p>
      </div>
    );
    export default Sidebar;

Later we decide to extend the sidebar with additional details:

    const Sidebar = ({ product }) => (
      <div className="sidebar">
        <DetailsPanel product={product} />
        <DimensionsPanel product={product} />
      </div>
    );

As our application grows we end up creating wrapper components like `Sidebar` that aren't using `product`, only passing it down to child components.

## React.useContext

[`useContext`](https://reactjs.org/docs/hooks-reference.html#usecontext) is one of the three basic React hooks.

The greatest thing here is that the component which is subscribed to the Context will automatically re-render when the Context value changes. Our Table/Sidebar layout sounds like an ideal use case for this.

Creating a context is simple as:

    const TableContext = React.createContext();
    export default TableContext;

and in order to subscribe to it:

    const value = React.useContext(TableContext);

We're using <Context.Provider> to set the `value` of the context. We'll wrap both `Table` and `Sidebar` components with this component because we want both of them to update when the context value changes:

    <TableContext.Provider value={product}>
      <Table
        products={products}
        onProductChange={handleProductChange}
      />
      <Sidebar />
    </TableContext.Provider>

Now instead of passing the selected product to the Sidebar component, we're setting it as the current value of `<TableContext.Provider>`. Finally, we subscribe `Sidebar` to context updates:

    const Sidebar = () => {
      const product = React.useContext(TableContext);

      return (
        <>
          <h3>{product.name}</h3>
          <p>ID: {product.id}</p>
          <p>SKU: {product.sku}</p>
        </>
      );
    }
    export default Sidebar;

The use of context allowed us to remove the product prop from the `Sidebar` component.
The real benefit of `useContext` shows as our application grows. Earlier we presented how the Sidebar would look like after some development and how different sidebar panels would need a reference to the current product. This won't be the case with useContext:

    const Sidebar = () => (
      <div className="sidebar">
        <DetailsPanel />
        <DimensionsPanel />
      </div>
    );

They'll also have child components using the current selection to display additional data or call APIs.
With the use of React.useContext passing down this selection through multiple layers of components is not required anymore.

See this code on https://codesandbox.io/embed/table-sidebar-layout-with-usecontext-os7wr?fontsize=14