# erpnext-variations-for-subscription-plans
avoid having extra items for each plan

One shortcoming of the otherwise lovely subscriptions: No specific attributes can be added to the item. Auto repeats allowed that through editing the original invoice.
Use case: Billing domains names, one would have a plan and item "Commercial Top Level Domain" for whenever a .com domain is billed. To mention the actual domain name, a separate item for each one is needed.

This server script together with a custom field allows to avoid that. Put it under Sales Invoice and fire "after save"
```
if doc.subscription is not None: # we stem from a subscription

    # add sustom field "custom_variation" to Subscription's subtable "Subscription Plan Detail"
    # join and query that from Sales Invoice together with the item
    SQL_1 = f"""
    SELECT DISTINCT item, custom_variation
    FROM `tabSales Invoice` si
    JOIN `tabSubscription` s ON si.subscription = s.name
    JOIN `tabSubscription Plan Detail` spd ON s.name = spd.parent
    JOIN `tabSubscription Plan`sp ON spd.plan = sp.name
    WHERE si.name = '{doc.name}'
    """
    item_variation = frappe.db.sql( SQL_1 )
    
    # loop through results and match items in sales invoice by item_code.
    # then amend custom_variation to description in Sales Invoice Item
    for iv in item_variation:
        sii = frappe.db.sql( # only select allowed here, so two steps
            f"""
            SELECT name, description FROM `tabSales Invoice Item` WHERE item_code = '{iv[0]}' AND parent = '{doc.name}'
            """
            )
        frappe.db.set_value('Sales Invoice Item', sii[0][0], 'description', sii[0][1] + ' ' + iv[1])
        
# two issues with this:
# 1. There is no way of knowing whether descriptions alredy are amended; Repeatingly saving a subscription-generated invoice will hence just lengthen the description
# 2. Subscription plans should only be repeated via the quantity column, otherwise the amendmend will be the same for each
```
