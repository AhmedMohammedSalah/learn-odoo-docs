

## Adding Custom CSS in Odoo 18

This guide explains how to add custom CSS files to your Odoo 18 module to modify the look and feel of your views. We will cover two key steps:

1.  Adding a CSS class to your view.
2.  Including the custom CSS file in your module's manifest.

### 1\. The Custom CSS File

First, you need to create your CSS file within your module's structure. The standard and recommended location for static assets like CSS files is inside the `static/src/styles/` directory of your module.

For example, if your module is named `property_finder`, you would create a file like this:

`property_finder/static/src/styles/style.css`

Inside this file, you can write your custom CSS rules. The best practice is to be specific with your selectors to avoid affecting other parts of Odoo. For example, if you want to style an element in a specific list view, you should use the custom class you've added to the view.

**Example `style.css` file:**

```css
/* Custom styles for the property_finder list view */
.o_list_table.custom_css .o_data_row {
    background-color: #f0f8ff; /* A light blue background for rows */
}

.o_list_table.custom_css .o_list_table_header th {
    background-color: #3e5e78; /* Darker blue header */
    color: white;
}

.o_list_table.custom_css .o_data_row.o_list_selected {
    background-color: #e0e0ff; /* A slightly different color for selected rows */
}
```

In this example, the `.custom_css` class is used to scope the styles so they only apply to views that have this class.

### 2\. Adding the CSS Class to the View

To apply your custom CSS rules to a specific view (in this case, a list view), you need to add a `class` attribute to the root element of the view's XML definition.

You provided the following example:

`<list multi_edit="1" class="custom_css" >`

This is the correct way to add a class. This line should be inside the `<field name="arch" type="xml">` section of your view's definition.

**Example View (`property_finder/views/property_finder_views.xml`):**

```xml
<odoo>
    <data>
        <record id="view_property_finder_tree" model="ir.ui.view">
            <field name="name">property.finder.tree</field>
            <field name="model">property.finder</field>
            <field name="arch" type="xml">
                <list multi_edit="1" class="custom_css">
                    <field name="name"/>
                    <field name="expecting_price"/>
                    <field name="selling_price"/>
                    <field name="diff"/>
                </list>
            </field>
        </record>
    </data>
</odoo>
```

By adding `class="custom_css"`, the Odoo web client will render this specific list view with an HTML element that has the `custom_css` class. Your CSS rules from step 1 will then apply to this view.

### 3\. Including the CSS File in the Manifest

For Odoo to know about and load your custom CSS file, you must declare it in the `assets` key of your module's `__manifest__.py` file. The `assets` key specifies which asset bundles your module contributes to.

You provided the following manifest entry:

```python
'assets': {
    'web.assets_background': [
        'property_finder/static/src/styles/style.css'
    ]
},
```

This is the correct syntax. Let's break down what `web.assets_background` means.

#### Understanding Asset Bundles

Odoo 18 uses asset bundles to manage and load static resources (CSS, JavaScript, etc.) efficiently. By including your file in a specific bundle, you control where and when it is loaded.

  * `web.assets_backend`: This is the most common bundle for backend modules. It loads your assets for the entire Odoo backend application. If you want your styles to be available everywhere in the backend, this is the bundle to use.
  * `web.assets_frontend`: This bundle is for resources used on the Odoo website.
  * **`web.assets_background`**: **(NOTE: This is not a standard Odoo 18 asset bundle.)** Odoo's asset bundles are named like `web.assets_backend`, `web.assets_frontend`, `web.assets_web`. While you could create your own bundle and add it to another one, the most straightforward approach for styling the backend is to use `web.assets_backend`.

**Correction and Best Practice:**

For a backend module like `property_finder`, it's almost always best to use `web.assets_backend`.

**Corrected Manifest Example (`property_finder/__manifest__.py`):**

```python
{
    'name': "Property Finder",
    'version': '1.0',
    'summary': "A module to manage properties",
    'depends': ['base'],
    'data': [
        'views/property_finder_views.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'property_finder/static/src/styles/style.css',
        ],
    },
    'installable': True,
    'application': True,
    'auto_install': False,
}
```

### Summary of Steps for Revision:

1.  **Create your CSS file:** Place your custom CSS rules in a file, typically located at `your_module/static/src/styles/style.css`.
2.  **Add a class to your view:** In your XML view definition, add a `class="your_class_name"` attribute to the root element (`<tree>`, `<list>`, `<form>`, etc.) to act as a selector.
3.  **Include the CSS file in the manifest:** In your `__manifest__.py`, add your CSS file's path to the `web.assets_backend` asset bundle under the `assets` key.
4.  **Update the module:** After making these changes, you must update your module in Odoo for the new assets and views to take effect. You can do this from the Odoo command line with `-u your_module_name` or from the Apps menu by clicking "Update."