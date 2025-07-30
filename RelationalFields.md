# Odoo Relational Fields

Odoo's ORM (Object-Relational Mapping) provides several relational field types to define relationships between different models (tables) in your database. These fields are crucial for building interconnected data structures and user interfaces.

## 1. Many2One

A `Many2one` field establishes a "many-to-one" relationship, meaning multiple records of the current model can be linked to one record of another model. It creates a foreign key relationship in the database.

- **Concept:**

  - Think of it as: "Many [Current Model] records belong to one [Related Model] record."
  - **Example:** An `Order` record is related to one `Client` record. Many orders can belong to the same client.
  - **Database Result:** The current model's table will have a foreign key column pointing to the ID of the related model's record.

- **Code Example:**

  - **Python Model Definition (`.py` file):**

    ```python
    from odoo import fields, models

    class Order(models.Model):
        _name = 'sale.order'
        _description = 'Sales Order'

        # ... other fields ...
        client_id = fields.Many2one('res.partner', string='Client', required=True)
        # 'res.partner' is the target model (the 'one' side of the relationship)
        # 'string' is the label displayed in the UI
        # 'required=True' makes it a mandatory field
    ```

  - **XML View (`.xml` file):**
    ```xml
    <field name="client_id"/>
    ```

## 2. One2Many

A `One2many` field establishes a "one-to-many" relationship. It's the inverse of a `Many2one` relationship. It means one record of the current model can be linked to multiple records of another model.

- **Concept:**

  - Think of it as: "One [Current Model] record has many [Related Model] records."
  - **Example:** A `Client` can place one or more `Order`s.
  - **Example:** An `Invoice` can have many `Invoice Line Item`s.
  - **Database Result:** This field **does not create a column in the current model's table**. Instead, it relies on a `Many2one` field existing in the _related_ model, pointing back to the current model.

- **Code Example:**

  - **Python Model Definition (`.py` file):**

    ```python
    from odoo import fields, models

    class Owner(models.Model):
        _name = 'owner.model'
        _description = 'Owner'

        name = fields.Char(string="Owner Name")
        # ... other fields ...

        # 'property_model' is the target model (the 'many' side)
        # 'owner_id' is the name of the Many2one field in 'property_model'
        #           that links back to this 'owner.model'
        property_ids = fields.One2many('property.model', 'owner_id', string='Properties')
    ```

    ```python
    # And in the 'property.model' definition, you would have:
    class Property(models.Model):
        _name = 'property.model'
        _description = 'Property'

        name = fields.Char(string="Property Name")
        owner_id = fields.Many2one('owner.model', string='Owner')
        # This is the Many2one field that the One2many in 'owner.model' links to
    ```

  - **XML View (`.xml` file):**
    ```xml
    <field name="property_ids">
        <tree string="Properties">
            <field name="name"/>
            </tree>
        </field>
    ```

## 3. Many2Many

A `Many2many` field establishes a "many-to-many" relationship, meaning multiple records of the current model can be linked to multiple records of another model, and vice-versa.

- **Concept:**

  - Think of it as: "Many [Current Model] records can be linked to many [Related Model] records."
  - **Example:** `Students` can apply for multiple `Courses`, and `Courses` can have multiple `Students`.
  - **Database Result:** This relationship is typically managed through an **intermediate joining table (or pivot table)** in the database. Odoo automatically creates and manages this table.

- **Code Example:**

  - **Python Model Definition (`.py` file):**

    ```python
    from odoo import fields, models

    class Student(models.Model):
        _name = 'student.model'
        _description = 'Student'

        name = fields.Char(string="Student Name")
        # ... other fields ...

        # 'course.model' is the target model
        # Odoo automatically manages the intermediate table for you.
        course_ids = fields.Many2many('course.model', string='Courses Enrolled')
    ```

    ```python
    # And in the 'course.model' definition, you might have the inverse:
    class Course(models.Model):
        _name = 'course.model'
        _description = 'Course'

        name = fields.Char(string="Course Title")
        student_ids = fields.Many2many('student.model', string='Enrolled Students')
    ```

  - **XML View (`.xml` file):**
    ```xml
    <field name="course_ids" widget="many2many_tags"/>
    ```

**Summary Table:**

| Field Type  | Relationship                      | Database Representation                            | Common Widget/View                   |
| :---------- | :-------------------------------- | :------------------------------------------------- | :----------------------------------- |
| `Many2one`  | Many of current to One of target  | Foreign key in current model                       | Dropdown / Searchable Selector       |
| `One2many`  | One of current to Many of target  | Relies on `Many2one` in target model pointing back | Embedded Tree/Kanban view            |
| `Many2many` | Many of current to Many of target | Intermediate joining table                         | `many2many_tags`, `many2many_kanban` |
