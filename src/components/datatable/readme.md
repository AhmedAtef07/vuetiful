# Datatable

The datatable component is essentially a "glorified table" designed to efficiently display dense data. Datatables aren't a heavily used component but they are fundamental to business applications - particularly ones displaying financial data or dashboards. A good datatable is hard to come by these days. Often they are too bulky, poorly designed, rigid, lack features or have too many dependencies on other frameworks. I have aimed to mitigate most of these problems with the Vuetiful datatable but I expect there will be features lacking that you need. 

If you would like a feature added to the datatable or if you find a bug, please open an issue on this repo.

**Note:** The line numbering is a little bit quirky in multi-group mode. I'll hopefully have it fixed soon. 

To see a demo of this component in action check out this codepen example: [http://codepen.io/andrewcourtice/full/woQzpa](http://codepen.io/andrewcourtice/full/woQzpa).

- [Getting Started](#getting-started)
- [Props](#props)
	- [Datatable](#datatable-1)
	- [Datatable Column](#datatable-column)
- [Formatting Data](#formatting-data)
- [Sorting Data](#sorting-data)
- [Grouping Data](#grouping-data)
- [Editing Data](#editing-data)
- [Customizing the Datatable](#customizing-the-datatable)
	- [Header Templates](#header-templates)
	- [Cell Templates](#cell-templates)
		- [View Mode](#view-mode)
		- [Edit Mode](#edit-mode)
- [Pagination](#pagination)

## Getting Started

Using the datatable is trivial. Just drop a `datatable` element in your html and start defining some `datatable-columns`. Refer to the **props** section below to see what props you can use with this component. I'll also talk about customizing cell and header templates using slots later. Here's a basic example:

```html
<datatable>
    <datatable-column id="datatable-column-1" label="Column 1"></datatable-column>
    <datatable-column id="datatable-column-2" label="Column 2"></datatable-column>
    <datatable-column id="datatable-column-3" label="Column 3"></datatable-column>
</datatable>
```

You can also use Vue's list rendering features to define columns in your viewmodel.

```javascript
new Vue({

    el: "#app",

    data: function() {
        return {
            columns: [
                { id: "datatable-column-1", label: "Column 1" },
                { id: "datatable-column-2", label: "Column 2" },
                { id: "datatable-column-3", label: "Column 3" }
            ]
        };
    }
});
```

```html
<datatable>
    <datatable-column v-for="column in columns" :id="column.id" :label="column.label"></datatable-column>
</datatable>
```


## Props

### Datatable

```html
<datatable 
    :source="customers.data"
    :striped="customers.striped"
    :filterable="customers.filterable"
    :editable="customers.editable"
    :line-numbers="customers.lineNumbers">

    <!-- datatable-column -->
</datatable>
```

| Prop | Type | Required | Default | Description |
| ---- | ---- | :------: | ------- | ----------- |
| source | array[object] | no | [] | An array of objects to display in the table. a.k.a. - the data. This is often JSON fetched from your application and parsed |
| striped | boolean | no | true | Whether the table should display alternate row coloring |
| filterable | boolean | no | true | Whether the table should display a textbox in the footer for filtering the current dataset |
| editable | boolean | no | false | Whether the table should be displayed in edit mode. See [Editing Data](#editing-data) |
| line-numbers | boolean | no | true | Whether the table should display line numbers on the left of each row |
**Note:** For specifying literal true values on props you can use a shorthand version. eg. instead of writing `editable="true"` you can just use the existence of the prop to define it's value eg. `editable`.

### Datatable Column

```html
<datatable-column 
    :id="column.id" 
    :label="column.label"
    :width="column.width"
    :sortable="column.sortable"
    :groupable="column.groupable"
    :total="column.total"
    :formatter="column.formatter">
</datatable-column>
```

| Prop | Type | Required | Default | Description |
| ---- | ---- | :------: | ------- | ----------- |
| id | string | yes |  | The uniqie id of this column. The id must correspond to a property on the objects in your data source |
| label | string | no | this.id | The label of this column to display in the header of the table. If none is specified, the id of this column will be used |
| width | string or number | no | | The width of the column. If a number is supplied the component will assume a percentage (eg. 25 = 25%). If a string is supplied the component will interpret it literally (eg. 3px, 3.5rem, 4em etc.) |
| sortable | boolean | no | true | Whether this row can be used for sorting |
| groupable | boolean | no | true | Whether this row can be used for grouping |
| total | boolean | no | false | Whether data in this column should be totalled and displayed at the bottom of the table. **Note:** for obvious reasons this is only available if the column has numeric data |
| formatter | function | no | | A function used to format the data in this column before it is displayed. This is particularly useful for dates and numeric value. See [Formatting Data](#formatting-data) |


## Formatting Data

More often than not you will likely run into a situation where you need to display data in a different format than it's raw form. This is common for dates and numbers. As seen above, each column has a `formatter` prop which allows you to specify a function that formats the data for that cell. There are several ways you could do this but using our basic example above, let's take a look at one way how you would implement a formatter:

```javascript
new Vue({

    el: "#app",

    data: function() {
        return {
            /* replace with real data */
            data: []
        };
    },

    methods: {

        formatCurrency: function(value) {
            var currency = parseFloat(value);

            if (isNaN(currency)) {
                return value;
            }

            return "$" + currency.toFixed(2).replace(/(\d)(?=(\d{3})+\.)/g, "$1,");
        }

    }
});
```

The format function signature expects one parameter which will be the value of the cell. 

```html
<datatable>
    <datatable-column id="datatable-column-1" label="Column 1"></datatable-column>
    <datatable-column id="datatable-column-2" label="Column 2"></datatable-column>
    <datatable-column id="datatable-column-3" label="Column 3" :formatter="formatCurrency"></datatable-column>
</datatable>
```


## Sorting Data

The datatable allows you to easily sort your data. To sort by a particular column, click the column header at the top of the table. The data will be displayed in ascending order by default. Clicking the same column header again will reverse the sort order (descending).


## Grouping Data

In addition to sorting, the datatable allows you to group your data by multiple columns. 

This probably best illustrated with an example. Let's say you have an online T-Shirt business and want to see specific profiles for purchases. Your purchases dataset may have the following columns: **Name**, **Email**, **Product Name**, **Quantity**, **Purchase Date**. Now that you have your data you may want to know which customers bought more than one of a particular T-Shirt on any given day. The process for this would be to group the data by **Purchase Date**, then group those groups by **Product Name**, then finally group the resulting group by **Quantity**. This can be repeated for as many columns as you have in any order.

To group by a particular column, simply drag the column header from the top of the table into the middle of the table. Keep repeating this step for each subsequent column you would like to group by. The columns being grouped on will be displayed at the top of the table under the headers.

To remove a column from the grouping, click the **x** button on the corresponding group at the top of the table.


## Editing Data

A quick note on editing data. When `editable` is set to true on the datatable, the default bahaviour of the table is to convert all the cells into textboxes and remove data formatting. In other words the user will be able to edit the raw data. Typing into the textbox will mutate (change) the data in the array bound to the `source` prop. 


## Customizing the Datatable

The datatable is very flexible with customizing how your users interact with the component. One of the ways you can change the component to suit your requirements is templating. This component makes use of Vue's scoped templates to allow you to define your own markup (and custom components) for column headers and cells.

### Header Templates

The default slot on the `datatable-column` component allows you to easily customize the content displayed in the column header. Let's look at an example where we want to have a column to allow users to select rows using a checkbox.

```html
<datatable-column id="select-all" width="3.25rem" :sortable="false" :groupable="false">
    <checkbox id="select-all-checkbox" v-model="selectAll"></checkbox>
</datatable-column>
```

**Note:** The `checkbox` component used in the example above is also part of the Vuetiful component framework. Refer to **components/toggles** for how to use `checkbox`, `radio` and `toggle` components.

### Cell Templates

Here is where the flexibility of this component really comes in handy. Sometimes you may want to go beyond just using the default cell template and a custom formatter. Custom cell templates allow you to define specific types of markup or custom components to be used for a particular column instead of the default one built into the component. By default, each cell gets a different template for view and edit modes respectively. Here's what gets rendered into each cell by default:

#### View Mode
```html
<span>{{ column.formatData(row[column.id]) }}</span>
```

#### Edit Mode
```html
<input type="text" v-model="row[column.id]" />
```

To override this all we have to do is define a template in our datatable and tell it to slot into the column we want. Here's an example extending on the custom header template example above:

```html
<datatable :source="customers.data" :editable="customers.editable">
    <datatable-column id="select-all" width="3.25rem" :sortable="false" :groupable="false">
        <checkbox id="select-all-checkbox" v-model="selectAll"></checkbox>
    </datatable-column>

    <!-- Define the rest of our columns here -->

    <template slot="sel" scope="cell">
        <div class="checkable-column">
            <checkbox :id="cell.row.id" :val="cell.row" v-model="customers.selected"></checkbox>
        </div>
    </template>
</datatable>
```

Notice how the `slot="select-all"` prop on the `template` matches the `id="select-all"` prop of the column we want to define a custom template for. Within your custom template a `cell` variable will be available for binding. Here's an outline of the properties available on the `cell` variable:

| Property | Type | Description |
| -------- | ---- | ----------- |
| row | object | The current object that represents this row. This object will have all the properties on it as defined in your data source. This is handy for accessing values for other columns in the current row. |
| column | datatable-column | This is the viewmodel of the current column that the repeater is using to get the value in the row. Calling `formatData(value)` on the column will call the formatter function defined in your `datatable-column`. |
| value | any | This is just some sugar to simplify getting the value for the cell. It is equivalent to calling `row[column.id]` | 

At this point you may be wondering how to define a different template for when the datatable is in edit mode. The short answer is: you don't have to. Just use Vue's `v-if` and `v-else` conditional bindings to change the content of the cell. Using this approach means you are in complete control over what gets rendered for the cell. You could even change the components based on the type of value that is in the current cell. Let's see what conditional rendering based on the datatables `editable` prop would look like:

```html
<template slot="sel" scope="cell">
    <div v-if="customers.editable">
        <!-- Put your custom edit template here -->
    </div>
    <div v-else>
        <!-- Put your custom view template here -->
    </div>
</template>
```


## Pagination

Pagination is particularly useful for when you need to display a large set of data but don't want to take up large amounts of screen real-estate displaying it all. It also means our datatable has to do less work by rendering smaller chunks of data at a time. 

You may have noticed that there is no option on the datatable for pagination. This is because pagination is not built into the datatable component. The `paginator` is actually a standalone component that allows you to paginate data and inject any child component into it's scope. Given that the `paginator` is a separate component I won't go into too much detail here on how to use it but I will show you a simple example of bundling it with the `datagrid`. To see how the `paginator` component works in detail, check out the **components/paginator** folder.

Here's a basic example of using the paginator with the `datagrid`:

```html
<paginator :source="customers.data" :page-size="5">
    <template scope="page">
        <!-- Notice here how we bind the datatable source to the data property exposed from the paginator -->
        <!-- this ensures that the only data the datatable is aware of is the current page -->
        <datatable id="data-table-main" :source="page.data">
            <!-- Datatable columns & templates -->
        </datatable>
    </template>
</paginator>
```

All we've done here is bound the `paginator` to the root data source instead of the `datatable`. The `datatable` source is then bound to the data exposed by the current page.