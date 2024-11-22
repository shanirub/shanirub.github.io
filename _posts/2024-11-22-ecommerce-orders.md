---
layout: post
title:  "Thoughts About Orders in E-commerce Project"
date:   2024-11-17 13:55:17 +0200
author: Shani
category: [ecommerce]
---

**Part 1: Some Kind of Introduction / Random Musing on Software Development**

When learning new technologies, one has to find the balance between reading documentation, manuals, guides, and how-tos, and practicing coding and designing.

I guess the balance is different for different people.

For me, it's about not finding myself hopeless against code or bored with endless reading.  
But mostly, it's about understanding my design (and code) won't be perfect, and I'll most likely need to revisit designs and refactor my code.  
And I guess that's a skill to master as well.

---

**Part 2: Specifics**

After this boring introduction, I'll get to the point:

In my `ecommerce` project, I created an app `orders` that should handle all ordering-related logic.  
This app contains two models and their respective managers:  
`Order` (with its `OrderManager`) and `OrderItem` (with its own `OrderItemManager`).  
I coded their CRUD operations easily. But things started to get complicated when I needed to address operations that were dependent on both models.

**`get_items_for_order(order_id)`** is, as one can assume, a method that returns a list of items when given an order ID.  
Where should I place this method?  
If I place it in `OrderManager`, I'll need to find a way to use `OrderItem` methods. This could mean circular imports, which lead to one of the most annoying things to debug.  
Placing it in `OrderItemManager` will create the same problem, only from a second point of view.

Also, I assume both managers were already created, and using them should be better than creating new ones.

`AI` suggested writing a utility method that won't be in any manager. I still haven't decided.

**`get_total_price_for_order(order_id)`**: When should this method be called? Whenever changing an order? What about a change in a `Product` that was used to create an item?  
Adding a new `total_price` attribute to the `Order` model seemed wrong.

I've decided that for now, calling this method should be done explicitly, and not by some other CRUD method.  
Also, I should look into a way to cache the total price, so I can display it (with a note saying the final price will be calculated when checking out).

All items should be fetched in one DB query, only then summarizing prices.
