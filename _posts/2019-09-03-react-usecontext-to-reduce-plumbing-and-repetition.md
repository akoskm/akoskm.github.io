# useContext to reduce plumbing and repeated propTypes at once

In the

    // table.jsx
    const Table = ({ data }) => (
      <table>
        <tbody>
        {data.forEach((d) => {
          <tr>
            <td>{d.id}</td>
            <td>{d.name}</td>
            <td>{d.sku}</td>
          </tr>
        })}
        </tbody>
      </table>
    );
    Table.propTypes = {
      data: PropTypes.arrayOf(
        PropTypes.shape({
          id: PropTypes.string,
          name: PropTypes.string,
          sku: PropTypes.string,
        }),
      ),
    };

and the sidebar:

    // sidebar.jsx
    const Sidebar = ({ value }) => (
      <div id={value.id}>
        <h1>{value.name}</h1>
        <div>SKU: {value.sku}</div>
      </div>
    );
    Sidebar.propTypes = {
      value: PropTypes.shape({
        id: PropTypes.string,
        name: PropTypes.string,
        sku: PropTypes.string,
      }),
    };

This leads to several problems, one of them is being refactoring, when the shape of the product changes. The other problem arises when your Sidebar component contains other components that need access to the selected product:

    // sidebar.jsx
    const Sidebar = ({ value }) => (
      <div>
        <h1>{value.name}</h1>
        <div>SKU: {value.sku}</div>
        <DetailsPanel product={selectedProduct} />
        <AdminPanel product={selectedProduct} />
      </div>
    )

because now both `DetailsPanel` and `AdminPanel` would need the same propTypes validation for `product`.

## Alternative solutions

You could use centralized PropTypes as shown here: https://www.freecodecamp.org/news/react-pattern-centralized-proptypes-f981ff672f3b/, where you would extract the PropType into a module and import it in components as needed.

A different approach shown here https://blog.getty.io/how-to-never-repeat-proptypes-declarations-again-abff2457c34c which doesn't create separate files containing only PropType definitions but takes advantage of the fact that you can access the exported component's `propTypes`.

Both of these provide a solution to eliminate the repeated PropType definitions, but what about ”plumbing” we do in `sidebar.jsx`?

## React.useContext

[`useContext`](https://reactjs.org/docs/hooks-reference.html#usecontext) is one of the three basic hooks. Creating a context as simple as:

    // table-context.jsx
    const TableContext = React.createContext();
    export default TableContext;

The specfic thing about hooks is that the component which uses this newly created context will automatically re-render when the context value changes. Our table/sidebar layout sounds like an ideal use case for this. Let's wrap our old products page with this context:

    // products-page.jsx
    import TableContext from './table-context';

    <TableContext.Provider value={selectedProduct}>
      <Table data={products}/>
      <Sidebar/>
    </TableContext.Provider>

instead of passing the selected product directly to the Sidebar component we're setting it as the current value of `TableContext`. Whenver we use:

    const selectedProduct = React.useContext(TableContext);

we have access to the currently selected product. This essentially turns our Sidebar code into this:

    // sidebar.jsx
    import TableContext from './table-context';

    const Sidebar = () => {
      const value = React.useContext(TableContext)
      return (
        <div>
          <h1>{value.name}</h1>
          <div>SKU: {value.sku}</div>
          <DetailsPanel />
          <AdminPanel />
        </div>
      );
    }

Because we removed the repeated product props from `DetailsPanel` and `AdminPanel` components, there is no need for PropTypes definitions in those components either.

There is one place where we set something to `selectedProduct`:

    <TableContext.Provider value={selectedProduct}>`

which is now the only place that needs PropTypes validation:

    // table-context.jsx
    const TableContext = React.createContext();
    TableContext.Provider.propTypes = {
      value: PropTypes.shape({
        id: PropTypes.string,
        name: PropTypes.string,
        sku: PropTypes.string,
      }),
    }
    export default TableContext;

-   example using useContext
	-   compare old/new code
-   benefits:
	-   no repeated proptypes
	-   no more plumbing
