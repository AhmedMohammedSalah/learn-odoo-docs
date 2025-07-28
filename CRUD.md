# CRUD Operations in Odoo 18

CRUD (Create, Read, Update, Delete) operations are fundamental to any Odoo model. Here's how they're implemented in Odoo 18:

## Create

The `create` method is used to create new records in the database.

```python
@api.model_create_multi
def create(self, vals_list):
    """
    Create one or multiple new records
    :param vals_list: List of dictionaries with field values
    :return: Created record(s)
    """
    # Add pre-create logic here if needed
    res = super().create(vals_list)
    
    # Add post-create logic here
    # Example: Send notification after creation
    # self.env['mail.thread'].message_post(
    #     body='New record created',
    #     partner_ids=[...],
    # )
    
    return res
```

Key points:
- `@api.model_create_multi` decorator handles both single and multiple record creation
- `vals_list` can be a single dictionary or a list of dictionaries
- Always call `super()` to maintain ORM behavior

## Read

Reading records is primarily done through search methods:

```python
@api.model
def _search(self, domain, offset=0, limit=None, order=None, access_rights_uid=None):
    """
    Custom search implementation
    :param domain: Search domain (list of conditions)
    :param offset: Number of records to skip
    :param limit: Maximum number of records to return
    :param order: Sorting order
    :param access_rights_uid: Optional user ID for access rights check
    :return: List of record IDs matching the criteria
    """
    # Add pre-search logic here
    res = super()._search(domain, offset, limit, order, access_rights_uid)
    
    # Add post-search logic here
    print("Records found:", len(res))
    
    return res
```

For simple reads, you can also use:
- `browse([ids])` - get records by ID
- `search_read(domain, fields, offset, limit, order)` - search and read fields in one call

## Update

The `write` method updates existing records:

```python
def write(self, vals):
    """
    Update records with new values
    :param vals: Dictionary of field:value pairs to update
    :return: True if successful
    """
    # Add pre-update logic here
    # Example: Track changes
    # old_values = {field: self[field] for field in vals.keys()}
    
    res = super().write(vals)
    
    # Add post-update logic here
    # Example: Send notification on specific field change
    # if 'state' in vals:
    #     self.message_post(body=f"State changed to {vals['state']}")
    
    return res
```

## Delete

The `unlink` method deletes records:

```python
def unlink(self):
    """
    Delete records from database
    :return: True if successful
    """
    # Add pre-delete logic here
    # Example: Check constraints before deletion
    # if any(record.state != 'draft' for record in self):
    #     raise UserError("Only draft records can be deleted!")
    
    res = super().unlink()
    
    # Add post-delete logic here
    # Example: Log deletion in another model
    # self.env['deletion.log'].create({
    #     'model': self._name,
    #     'records_deleted': len(self),
    # })
    
    return res
```

## Best Practices

1. Always call `super()` to maintain ORM behavior
2. Keep business logic separate from CRUD operations when possible
3. Use decorators appropriately (`@api.model`, `@api.model_create_multi`)
4. Consider security implications in each operation
5. Add appropriate error handling and validation
6. Document your customizations clearly

Remember that Odoo's ORM handles many basic operations automatically, so only override these methods when you need custom behavior.