# Odoo 18 Validation: A Comprehensive Guide

Validation in Odoo is crucial for maintaining data integrity and ensuring a smooth user experience. It helps prevent incorrect or incomplete data from being saved into the system. Odoo offers several layers of validation, each serving a specific purpose. Let's explore these in detail with examples.

---

## 1\. Database Tier Validation

Database-level constraints are the most fundamental form of validation. They are enforced directly by the database management system (e.g., PostgreSQL) and ensure data integrity even if an operation bypasses Odoo's application logic.

### `_sql_constraints`

This is the primary way to define database constraints in Odoo. You declare them as a list of tuples within your model.

**Syntax:**

```python
_sql_constraints = [
    ('constraint_name', 'SQL_constraint_definition', 'Error message for the user'),
]
```

**Common `SQL_constraint_definition` examples:**

- **`UNIQUE(field_name)`**: Ensures that the values in the specified field are unique across all records in the table.
- **`CHECK(condition)`**: Enforces a condition that must be true for every row.
- **`FOREIGN KEY (field_name) REFERENCES other_table (id)`**: Establishes a link between two tables, ensuring referential integrity. (While less common directly in `_sql_constraints` for standard relational fields, it's good to understand the concept).

**Example:**

Let's say you have a `Course` model, and each course needs a unique code.

```python
# custom_addons/my_module/models/course.py

from odoo import fields, models

class Course(models.Model):
    _name = 'my_module.course'
    _description = 'Course'

    name = fields.Char(string='Course Name', required=True)
    code = fields.Char(string='Course Code', required=True)
    description = fields.Text(string='Description')

    _sql_constraints = [
        ('unique_course_code', 'UNIQUE(code)', 'The course code must be unique! Please choose a different code.'),
        ('check_course_name_length', 'CHECK(LENGTH(name) > 5)', 'The course name must be at least 6 characters long.'),
    ]

```

In this example:

- `unique_course_code` ensures that no two courses can have the same `code`.
- `check_course_name_length` ensures that the `name` field has a minimum length of 6 characters.

---

## 2\. Logic Tier Validation

This validation occurs within Odoo's Python models, allowing for more complex and dynamic validation rules that might involve multiple fields or external logic.

### a. Field Attributes

Odoo provides built-in validation through field attributes, which are simple and effective for common scenarios.

- **`required=True`**: This attribute makes a field mandatory. If a user tries to save a record without a value in this field, Odoo will raise a validation error.

  ```python
  name = fields.Char(string='Product Name', required=True)
  ```

- **`size`**: For `Char` fields, `size` limits the maximum number of characters allowed.

  ```python
  product_code = fields.Char(string='Product Code', size=10) # Max 10 characters
  ```

- **`digits`**: For `Float` fields, `digits` specifies the precision (total digits, digits after the decimal point).

  ```python
  price = fields.Float(string='Price', digits=(10, 2)) # e.g., 12345678.99
  quantity = fields.Float(string='Quantity', digits=(0, 5)) # Only decimal places, no integer part specified
  ```

### b. Python Constraints (`@api.constrains`)

For more intricate validation logic, you use the `@api.constrains` decorator. These methods are executed when a record is created or updated, specifically when the fields listed in the decorator are modified.

**Syntax:**

```python
from odoo import api
from odoo.exceptions import ValidationError

class MyModel(models.Model):
    _name = 'my.model'

    field_one = fields.Integer()
    field_two = fields.Integer()

    @api.constrains('field_one', 'field_two')
    def _check_fields_logic(self):
        for rec in self:
            if rec.field_one <= 0:
                raise ValidationError("Field One must be greater than zero!")
            if rec.field_one > rec.field_two:
                raise ValidationError("Field One cannot be greater than Field Two!")
```

**Example: Validating a Start and End Date**

Consider a `Project` model where the `end_date` must always be after the `start_date`.

```python
# custom_addons/my_module/models/project.py

from odoo import fields, models, api
from odoo.exceptions import ValidationError

class Project(models.Model):
    _name = 'my_module.project'
    _description = 'Project'

    name = fields.Char(string='Project Name', required=True)
    start_date = fields.Date(string='Start Date', required=True)
    end_date = fields.Date(string='End Date', required=True)
    budget = fields.Float(string='Budget', digits=(16, 2))

    @api.constrains('start_date', 'end_date')
    def _check_dates(self):
        for rec in self:
            if rec.start_date and rec.end_date and rec.start_date > rec.end_date:
                raise ValidationError("The end date cannot be earlier than the start date!")

    @api.constrains('budget')
    def _check_budget_positive(self):
        for rec in self:
            if rec.budget < 0:
                raise ValidationError("Project budget cannot be negative!")

```

In this example:

- `_check_dates` ensures the chronological order of start and end dates.
- `_check_budget_positive` verifies that the budget is not a negative value.

---

## 3\. Presentation Tier Validation

This layer of validation is primarily handled in the Odoo user interface (UI) through XML views. While it doesn't prevent data from being saved at the database or logic level if bypassed (e.g., through API calls), it significantly improves the user experience by providing immediate visual feedback.

### `required="1"` in XML Views

You can make a field visually required in the UI by adding `required="1"` to its definition in the form view. This adds a small asterisk next to the field label, indicating it's mandatory.

**Example:**

```xml
<record id="project_form_view" model="ir.ui.view">
    <field name="name">project.form</field>
    <field name="model">my_module.project</field>
    <field name="arch" type="xml">
        <form string="Project">
            <sheet>
                <group>
                    <field name="name" required="1"/>
                    <field name="start_date" required="1"/>
                    <field name="end_date" required="1"/>
                    <field name="budget"/>
                </group>
            </sheet>
        </form>
    </field>
</record>
```

In this view, the `name`, `start_date`, and `end_date` fields will visually be marked as required in the Odoo UI.

---

## Choosing the Right Validation Tier

- **Database Tier (`_sql_constraints`)**: Use for fundamental, unchangeable rules that must always hold true, regardless of how data is entered. Ideal for uniqueness, basic range checks, and referential integrity.
- **Logic Tier (`@api.constrains` / Field Attributes)**: Use for more complex business rules that might involve multiple fields, conditional logic, or interactions with other records. Field attributes are for simple, direct validations.
- **Presentation Tier (`required="1"` in XML)**: Primarily for user experience, guiding users to provide necessary information. This is a visual cue and should be backed up by database or logic tier validation for true data integrity.

By combining these validation tiers, you can create robust and user-friendly Odoo applications that ensure the highest quality of data.
