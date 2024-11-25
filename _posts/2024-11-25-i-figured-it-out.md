---
layout: post
title:  "I figured it out"
date:   2024-11-25 18:55:17 +0200
author: Shani
---

So, the solution for those `Order`/`OrderItem` methods was not that complicated after all.  
I’m a bit annoyed that I took the long route to my solution, but then again, these projects were created to learn and practice, so every detour is welcome!

I added two methods under `OrderManager`:

```python
def get_items_for_order(self, order_id):
    ...
    order = self.get_order(order_id)
    order_items = order.items.all()
```

The first one is a simple query to find all order items that use the given `order_id` as a foreign key.

The second is:

```python
def get_total_price(self, order_id):
    ...
```

It uses the first method to find all items in the order, then iterates over them, summing their prices and returning the total.

---

And some blog news: I updated my "About" page.
