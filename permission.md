# Odoo Permissions

Odoo's permission system operates on multiple levels to control user access to data and functionalities. This document outlines the key aspects of managing permissions in Odoo.

## 1. Logic Level: `ir.model.access.csv` (Access Rights)

At the core of Odoo's permission logic are **Access Rights**, defined in `ir.model.access.csv` files within your Odoo modules. These CSV files dictate what operations (read, write, create, unlink) users belonging to specific groups can perform on a given model.

**Syntax Example:**

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
property,property_finder.property_access_user,model_property_property,base.group_user,1,1,1,0
# Explanation of fields:
# id: A unique XML ID for this access rule.
# name: A human-readable name for the access rule.
# model_id:id: The external ID of the model this rule applies to (e.g., 'model_res_partner' for 'res.partner').
# group_id:id: The external ID of the group this rule applies to. If left empty, it applies to all users (global access).
# perm_read: 1 for read access, 0 for no read access.
# perm_write: 1 for write (update) access, 0 for no write access.
# perm_create: 1 for create access, 0 for no create access.
# perm_unlink: 1 for unlink (delete) access, 0 for no unlink access.
```

**Key Points about `ir.model.access.csv`:**

- **Granular Control:** You can define specific permissions for each model and for different user groups.
- **Default Behavior:** If no access rule is defined for a model, typically only users with "Technical Features" enabled (e.g., Administrator) will have full access.
- **Additive Permissions:** If a user belongs to multiple groups, their permissions are additive. If one group grants read access and another grants write access, the user will have both.
- **Global Access:** Leaving `group_id:id` empty grants the specified permissions to all users in the system. Use with caution.
- **Security Risk:** Incorrectly configured access rights can expose sensitive data or allow unauthorized modifications.

## 2\. Presentation Layer: View Attributes

While `ir.model.access.csv` controls the underlying data access, Odoo also provides attributes within view definitions (XML files) to control the _presentation_ of edit, create, and delete functionalities. These attributes are often used to enhance usability or reinforce access rights visually.

**Common View Attributes:**

- **`create`**: Controls the visibility of the "Create" button/action in a view.
  - `create="1"` (or omit): Allows creating new records (default if not specified).
  - `create="0"`: Hides the "Create" button/action, preventing users from creating new records directly from this view.
- **`edit`**: Controls whether records can be edited in a view.
  - `edit="1"` (or omit): Allows editing existing records.
  - `edit="0"`: Makes the view read-only, preventing users from editing records directly from this view.
- **`delete`**: Controls the visibility of the "Delete" button/action in a view.
  - `delete="1"` (or omit): Allows deleting records.
  - `delete="0"`: Hides the "Delete" button/action, preventing users from deleting records directly from this view.

**Example Usage in XML Views:**

```xml
<form string="My Record Form" create="1" edit="1" delete="0">
    </form>

<tree string="My Records List" multi_edit="1" create="1" edit="0" delete="0">
    </tree>
```

**Specific to List Views:**

- **`multi_edit="1"`**: This attribute, specifically for `tree` views, enables or disables the multi-record editing feature.
  - `multi_edit="1"`: Allows users to multi-select records and perform bulk edits if applicable.
  - `multi_edit="0"`: Disables multi-editing.

**Important Considerations for View Attributes:**

- **Presentation vs. Logic:** These view attributes **only affect the user interface**. They do _not_ override the underlying `ir.model.access.csv` rules. If `perm_create` is `0` in `ir.model.access.csv` for a user's group, but `create="1"` is set in the view, the user will see the "Create" button but will receive an "Access Denied" error upon trying to save a new record.
- **Redundancy and Clarity:** It's good practice to ensure consistency between your `ir.model.access.csv` rules and your view attributes. If a user doesn't have `perm_create` rights, it's better to set `create="0"` in the view to prevent them from seeing an action they can't perform, improving the user experience.
- **Security Layer:** The `ir.model.access.csv` is the primary security layer. View attributes are a usability and presentation layer.

## 3\. Other Permission Mechanisms (Advanced)

- **Record Rules (`ir.rule`):** Provide row-level security. You can define rules to restrict which _records_ a user can see or modify, even if they have general access rights to the model.
- **Field Access (`groups` attribute on fields):** You can hide or make specific fields read-only for certain user groups directly within the model definition.
- **Method Access (`@api.constrains`, `@api.depends`, etc.):** While not direct permissions, method decorators and custom logic within methods can enforce business rules and indirectly restrict actions.
- **Security Groups:** Users are assigned to security groups, and these groups are linked to access rights and record rules.

By effectively combining `ir.model.access.csv` definitions, view attributes, and other advanced mechanisms, you can build a robust and secure permission system in your Odoo applications.

```

```
