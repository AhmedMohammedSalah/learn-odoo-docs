
# Understanding `self` in Odoo Models

In Odoo, the `self` keyword, much like in standard Python classes, refers to the instance of the object on which a method is being called. However, in Odoo's ORM (Object-Relational Mapping) context, `self` has a unique characteristic: **it's always a recordset**, even if that recordset contains only one record.

## `self` is Always a Recordset

This is a crucial concept to grasp. Whether you're working with a single record or multiple records, `self` will always be an instance of `odoo.models.Model` (or a subclass), representing a collection of records.

### Case 1: `self` as a Single Record Recordset

When you're interacting with a form view and performing an action on a **single record** (e.g., clicking a button on a product form, editing a specific customer), the method invoked will receive `self` as a recordset containing just that one record.

**Example:**

Let's say you have a `Product` model, and you add a button to mark a product as "unavailable."

```python
# my_module/models/product.py

from odoo import fields, models

class Product(models.Model):
    _name = 'product.product'
    _inherit = 'product.product' # Inherit from the base product model

    is_available = fields.Boolean(string='Available', default=True)

    def action_mark_unavailable(self):
        # In this context, self will typically be a recordset with one record
        self.is_available = False
        # You could also access self.name, self.list_price, etc. for this single record
        print(f"Product '{self.name}' marked as unavailable.")

```

If a user clicks this button on a single product's form, `self` inside `action_mark_unavailable` will be a recordset containing only that one product record. You can then directly modify its fields like `self.is_available = False`.

### Case 2: `self` as a Multi-Record Recordset

When you perform an action that affects **multiple records** (e.g., selecting several products in a list view and clicking an "Archive" button, or running a scheduled action that processes many records), `self` will be a recordset containing all the selected or processed records.

**Example:**

Continuing with the `Product` model, imagine an an action that marks multiple selected products as "unavailable."

```python
# my_module/models/product.py

from odoo import fields, models

class Product(models.Model):
    _name = 'product.product'
    _inherit = 'product.product'

    is_available = fields.Boolean(string='Available', default=True)

    def action_mark_unavailable_multiple(self):
        # In this context, self can be a recordset with multiple records
        for product_record in self:
            product_record.is_available = False
            print(f"Product '{product_record.name}' marked as unavailable.")

```

If a user selects five products in the list view and triggers `action_mark_unavailable_multiple`, `self` will be a recordset containing those five product records. You'll then need to iterate over `self` to process each record individually, as shown with `for product_record in self:`.

## Why is `self` Always a Recordset?

This design choice in Odoo's ORM provides significant flexibility and efficiency:

  * **Consistency:** It simplifies method signatures. You don't need separate methods for single-record and multi-record operations. The same method can handle both.
  * **Batch Operations:** It enables efficient batch operations. When you modify fields on `self` directly (e.g., `self.is_available = False` when `self` is a multi-record recordset), Odoo's ORM intelligently groups these operations into a single database transaction, improving performance.
  * **Chaining:** Recordsets support chaining operations, allowing for fluid and readable code (e.g., `self.filtered(lambda r: r.price > 100).mapped('name')`).

## The Singleton Problem (or "Expected Singleton")

The "Singleton Problem" or "Expected Singleton" error is a common pitfall for new Odoo developers, directly related to `self` being a recordset. It occurs when your code attempts to access a field directly on a recordset (`self.field_name`) in a context where `self` might contain **more than one record**.

Odoo expects that if you're accessing a field directly without iterating, `self` should logically represent a *single record*. If `self` contains multiple records, Odoo doesn't know *which* record's `field_name` you are trying to access, and it raises a `ValueError`.

**When does it happen?**

This typically occurs in `@api.depends`, `@api.constrains`, and regular methods if you forget to iterate over `self` when it might contain multiple records.

**Example of the Singleton Problem:**

Consider a computed field that calculates a `total_value` for a sale order.

```python
# my_module/models/sale_order.py

from odoo import fields, models, api

class SaleOrder(models.Model):
    _name = 'sale.order'
    _inherit = 'sale.order'

    total_value = fields.Float(string='Total Value', compute='_compute_total_value')

    @api.depends('order_line.price_total')
    def _compute_total_value(self):
        # THIS CODE WILL CAUSE A SINGLETON ERROR IF self HAS MORE THAN ONE RECORD
        # self.total_value = sum(self.order_line.mapped('price_total'))

        # Correct way to handle: iterate over self
        for order in self:
            order.total_value = sum(order.order_line.mapped('price_total'))
```

**Explanation of the Error:**

If you were to use `self.total_value = ...` in the `_compute_total_value` method, and Odoo recalculates this field for multiple `sale.order` records at once (e.g., after an import, or a batch update), `self` would be a recordset containing more than one `sale.order`. When Odoo tries to assign a value to `self.total_value`, it doesn't know *which* `total_value` among the multiple records in `self` it should update, leading to the "Expected singleton: sale.order(...)" error.

**The Solution:**

**Always iterate over `self` when you need to access or modify fields on individual records within the recordset.**

```python
# Corrected example from above
@api.depends('order_line.price_total')
def _compute_total_value(self):
    for order in self: # Iterate through each sale order record in self
        order.total_value = sum(order.order_line.mapped('price_total'))
```

By iterating with `for order in self:`, `order` becomes a singleton recordset (a recordset containing exactly one record) in each iteration, allowing you to safely access its fields like `order.total_value` or `order.order_line`.

## Key Takeaway

Always remember to **iterate over `self`** if your method needs to perform logic that depends on the individual field values of each record within the recordset. If you are setting a common value for all records in the recordset, you can often do it directly on `self` without explicit iteration. However, for computed fields, constraints, and methods where individual record logic is required, iteration is essential to avoid the singleton error.

This understanding of `self` and the singleton problem is fundamental to writing effective and efficient Odoo code\!

-----