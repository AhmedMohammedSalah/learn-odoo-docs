# Odoo 18 Workflow Implementation Guide

This document explains how to implement a basic state-based workflow in Odoo 18.

## Workflow States

The workflow consists of three states:

1. **Draft** - Initial state
2. **Pending** - Intermediate state
3. **Sold** - Final state

## Implementation Steps

### 1. Model Definition

Add a selection field to track the state in your model (`*.py` file):

```python
state = fields.Selection([
    ('draft', 'Draft'),
    ('pending', 'Pending'),
    ('sold', 'Sold'),
], default='draft', string="Status")

# State transition methods
def action_draft(self):
    self.state = 'draft'

def action_pending(self):
    self.state = 'pending'

def action_sold(self):
    self.state = 'sold'
```

### 2. View Configuration

In your form view XML (`*.xml` file), add buttons and status bar:

```xml
<sheet>
    <header>
        <!-- Draft button - only visible when NOT in draft state -->
        <button
            type="object"
            name="action_draft"
            invisible="state in ('draft')"
            string="Reset to Draft"
            class="oe_highlight"/>

        <!-- Pending button - only visible in draft state -->
        <button
            type="object"
            name="action_pending"
            invisible="state in ('pending', 'sold')"
            string="Mark as Pending"/>

        <!-- Sold button - only visible in pending state -->
        <button
            type="object"
            name="action_sold"
            invisible="state in ('sold', 'draft')"
            string="Mark as Sold"
            class="oe_highlight"/>

        <!-- Visual status bar -->
        <field name="state"
               widget="statusbar"
               options="{'clickable': '1'}"/>
    </header>
    <!-- Your form content goes here -->
</sheet>
```

## Key Points to Remember

1. **State Field**:

   - `Selection` field defines possible states
   - `default='draft'` sets initial state

2. **Transition Methods**:

   - Simple methods that change the state value
   - Naming convention: `action_<state_name>`

3. **View Buttons**:

   - `type="object"` calls Python methods
   - `invisible` attribute controls button visibility based on state
   - Use `oe_highlight` class for primary actions

4. **Status Bar**:
   - `widget="statusbar"` shows visual progression
   - `clickable="1"` allows clicking between states (if methods exist)

## Visual Workflow

Draft → (Pending) → Sold

- From **Draft**: Can go to Pending
- From **Pending**: Can go back to Draft or forward to Sold
- From **Sold**: Can only go back to Draft

## Tips

1. Add security checks in transition methods if needed
2. Consider adding constraints with `@api.constrains('state')`
3. For complex workflows, explore `base.automation` module
