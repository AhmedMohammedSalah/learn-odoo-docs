# Odoo Inheritance Guide

## Inheritance in Odoo

Inheritance in Odoo is a powerful mechanism that allows developers to customize, extend, and reuse existing code. It is mainly divided into three categories:

---

## 1. Python Inheritance

This form of inheritance is used primarily to override or extend the behavior of models in Python code.

### Features:

* Inherit existing models using `_inherit`
* Override or extend CRUD methods like `create`, `write`, `unlink`, `action_confirm`, etc.

### Example:

In `__manifest__.py`:

```python
'depends': [
    'base', 'sale_management', 'account'
],
```

In `models.py`:

```python
from odoo import models

class SaleOrder(models.Model):
    _inherit = 'sale.order'

    def action_confirm(self):
        res = super().action_confirm()
        print("Inside Python override method")
        return res
```

Another example - overriding `create` method:

```python
class ResPartner(models.Model):
    _inherit = 'res.partner'

    def create(self, vals):
        vals['name'] = vals.get('name', '').upper()  # Capitalize name before save
        return super().create(vals)
```

---

## 2. Model Inheritance

Model inheritance lets you reuse and extend models. There are three types:

### a. Extension of a Model

This method adds new fields or methods to an existing model.

```python
class SaleOrder(models.Model):
    _inherit = 'sale.order'
    property_id = fields.Many2one('property')
```

### b. Classical Inheritance (Copy an Existing Model)

This copies all fields from a base model into a new model.

```python
class Client(models.Model):
    _name = 'client'
    _inherit = 'owner'
    # This will create a new table 'client' with all fields of 'owner'
```

### c. Delegation Inheritance

Delegation inheritance lets you link your model to another model and reuse its fields by referencing instead of copying.

#### Syntax:

```python
class Client(models.Model):
    _name = 'client'
    _inherits = {'owner': 'owner_id'}

    owner_id = fields.Many2one('owner', required=True, ondelete='cascade')
    client_reference = fields.Char()
```

* In this case, `client` table will have its own fields (like `client_reference`) and will also expose all the fields of `owner` via the delegated relation through `owner_id`.

#### Benefits:

* Keeps models normalized (reduces duplication)
* Enables reuse of functionality from parent models

#### Example Use Case:

You have a model `owner` with personal info:

```python
class Owner(models.Model):
    _name = 'owner'

    name = fields.Char()
    address = fields.Text()
```

And you want `client` to use all fields of `owner` without duplication:

```python
class Client(models.Model):
    _name = 'client'
    _inherits = {'owner': 'owner_id'}

    owner_id = fields.Many2one('owner', required=True, ondelete='cascade')
    subscription_type = fields.Selection([
        ('basic', 'Basic'),
        ('premium', 'Premium')
    ])
```

Now, when you create a `client`, you get access to `name`, `address`, and `subscription_type` all in one model.

#### Important:

In the backend database, `client` and `owner` are stored in different tables.

---

## 3. View Inheritance

View inheritance allows you to modify existing views (forms, trees, etc.) using XML.

### Example:

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="sale_order_form_inherit" model="ir.ui.view">
            <field name="name">sale_order.form</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_order_form"/>
            <field name="arch" type="xml">
                <field name="partner_id" position="after">
                    <field name="property_id"/>
                </field>

                <!-- Alternatively using XPath -->
                <xpath expr="//field[@name='partner_id']" position="after">
                    <field name="property_id"/>
                </xpath>
            </field>
        </record>

    </data>
</odoo>
```

### Tips:

* Use `position="after"` or `before` to place fields in specific locations.
* Use `xpath` for more control over placement and structure.

#### 🔍 How to Get Model Name, View Name, or Action ID in Odoo:

1. **Activate Developer Mode**:

   * Go to the settings page and enable developer mode (or append `?debug=1` to the URL).

2. **Inspect Views and Models**:

   * Open the form or tree view you want to extend.
   * Click the **bug icon** (top right) > **Edit View: Form/List** to see:

     * `View ID`
     * `Model`
     * `XML ID` (used in `inherit_id`)

3. **Get Action Name**:

   * Click the **bug icon** > **Edit Action** to view:

     * Action name and technical ID
     * Used for adding menus or extending existing ones

4. **Get Model Name from Records**:

   * Open any record (e.g., a sale order) and check the URL:

     * Example: `/web#id=15&model=sale.order&view_type=form`
     * Here, `model=sale.order`

5. **View in Database**:

   * Use Odoo's `ir.model`, `ir.ui.view`, and `ir.actions.act_window` models to query metadata about views, models, and actions.

---

## Summary:

| Type                   | Use Case                                                  |
| ---------------------- | --------------------------------------------------------- |
| Python Inheritance     | Override/extend model behavior in Python code             |
| Model Inheritance      | Reuse or extend fields and logic                          |
| View Inheritance       | Modify UI components in XML views                         |
| Delegation Inheritance | Link models via foreign keys and reuse fields dynamically |

Let me know if you want me to add practical tasks or exercises based on this content.
