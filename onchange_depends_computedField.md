## Odoo 18: `onchange` and `depends` Decorators

This document serves as a detailed guide to Odoo's `onchange` and `depends` decorators, as well as the concept of computed fields. It is intended to help with revision and provide a deeper understanding of these core Odoo concepts.

### 1\. Computed Fields and the `@api.depends` Decorator

A computed field is a field whose value is not stored directly in the database but is calculated on the fly. Its value is derived from the values of one or more other fields.

The `@api.depends` decorator is essential for a computed field. It specifies which fields, when changed, should trigger the re-computation of the decorated method.

#### Your Code Example:

```python
expecting_price = fields.Float()
selling_price = fields.Float()
diff = fields.Float(compute='_compute_diff')

@api.depends('expecting_price', 'selling_price', 'owner_id.phone')
def _compute_diff(self):
    print('inside _compute_diff')
    for rec in self:
        rec.diff = rec.expecting_price - rec.selling_price
```

#### Detailed Breakdown:

1.  **`diff = fields.Float(compute='_compute_diff')`**:
      * This line declares a new field called `diff`.
      * The `compute` argument tells Odoo that the value for this field will not be written to the database directly. Instead, it will be calculated by the method specified as its value, which is `_compute_diff`.
2.  **`@api.depends('expecting_price', 'selling_price', 'owner_id.phone')`**:
      * This is the core of the computed field logic.
      * It informs Odoo that the `_compute_diff` method should be executed whenever the value of `expecting_price` or `selling_price` changes.
      * **Crucially, it also handles related fields.** `owner_id.phone` means that if the phone number of the related owner record changes, the `_compute_diff` method will also be triggered.
      * Odoo's ORM (Object-Relational Mapper) is smart enough to trace these dependencies, ensuring that the computed field's value is always up-to-date.
3.  **`def _compute_diff(self):`**:
      * This is the method that performs the actual calculation.
      * It is decorated with `@api.depends`.
      * The `for rec in self:` loop is standard practice in Odoo. The `self` variable is a record set, and you should always iterate through it to ensure your method works correctly for single records and multiple records simultaneously.
4.  **`rec.diff = rec.expecting_price - rec.selling_price`**:
      * Inside the loop, the value of the `diff` field for each record (`rec`) is set to the difference between its `expecting_price` and `selling_price`.

#### Key Takeaway for `@api.depends`:

  * **`@api.depends` works on all model fields, regardless of whether they are visible in the view.**
  * It is the mechanism for keeping computed fields synchronized with their source fields.
  * The `compute` method is executed on the server, ensuring data consistency even when records are created or updated programmatically.

-----

### 2\. The `@api.onchange` Decorator

The `@api.onchange` decorator is used to automatically update other fields in the user interface (the form view) as soon as one field's value is modified. It is a client-side mechanism designed to improve the user experience.

Unlike computed fields, `onchange` methods do not save their changes to the database until the user explicitly saves the record. They are used for immediate feedback, validation, and dynamic UI updates.

#### Your Code Example:

```python
@api.onchange('expecting_price')
def _onchange_expecting_price(self):
    for rec in self:
        if rec.expecting_price < 0:
            print(rec)
            return {
                'warning': {
                    'title': 'be careful',
                    'message': 'the value is negative',
                    'type': 'notification'
                }
            }
```

#### Detailed Breakdown:

1.  **`@api.onchange('expecting_price')`**:
      * This decorator specifies that the `_onchange_expecting_price` method should run immediately when a user changes the value of the `expecting_price` field **in a form view**.
      * The method is executed on the client-side (in the browser), which is why it's so fast and interactive.
2.  **`if rec.expecting_price < 0:`**:
      * This is a simple validation check.
3.  **`return {'warning': ... }`**:
      * This is a special return value for `onchange` methods. Odoo's web client interprets this dictionary and displays a warning pop-up to the user.
      * You can also use `onchange` to update the values of other fields in the view. For example, if changing one field should automatically fill in another, an `onchange` method can do this.

#### Key Takeaway for `@api.onchange`:

  * **`@api.onchange` works only on view fields.** It is a UI-driven feature.
  * It's designed for **instant feedback and dynamic UI updates**. It does not perform an immediate database write.
  * It can be used to set field values, display warnings, and dynamically control the visibility or required status of other fields in the view.

-----

### Summary Table for Revision

| Feature       | `@api.depends` (Computed Fields)                              | `@api.onchange`                                     |
|---------------|---------------------------------------------------------------|-----------------------------------------------------|
| **Purpose** | To calculate a field's value based on other fields.           | To provide instant UI feedback and validation.      |
| **Trigger** | A change to any of the fields listed in the decorator.        | A user's change to a field in a form view.          |
| **Execution** | On the server (within the ORM).                               | On the client (in the browser).                     |
| **Scope** | Works for all model fields, regardless of view presence.      | Only works on fields present in the view.           |
| **Result** | The computed field's value is saved to the database (if `store=True`). | Changes are not saved until the user clicks `Save`. |
| **Use Case** | Calculating totals, statuses, or derived information.         | Dynamic form updates, showing warnings, or setting default values. |
| **Return** | The method sets the field's value directly (`rec.field = value`). | The method can return a dictionary for warnings or domain updates. |