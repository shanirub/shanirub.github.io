---
layout: post
title:  "Updating order"
date:   2024-11-27 16:09:16+02:00
author: Shani
---

I had some problems with the view for updating an order item.

It all started with testing this simple request:

```python
        response = self.client.post(
            reverse('orderitem-update', args=[order_item.pk]),  # Pass the pk in the URL
            {'quantity': new_quantity,}  # Pass the update data as form data
        )
```

The expected behavior was for this request to return a redirect for a successful update:

```python
class OrderItemUpdateView(GroupRequiredMixin, OwnershipRequiredMixin, UpdateView):
    model = OrderItem
    fields = ['quantity', 'price']
    template_name = 'update_order_item.html'
    allowed_groups = ['customers', 'shift_manager']

    def get_success_url(self):
        return reverse_lazy('orderitem-detail', kwargs={'pk': self.object.pk})
```

I kept getting `200` which meant something went wrong.

Turned out, the view was expecting an updated `price` value as well.

My first (wrong) approach, was to fill in the current price value, if not given. And I did just that.
I was so hang on on making this request work, I didn't stop to think my logic here is off.

I tested an edge case, where while a customer updates one of its order items, the price for the respective product was updated by the staff.
I expected the price of the order item to be updated accordingly, but that didn't happen.

This bug was a blessing, since it made me stop and rethink my approach. Until now, the `price`	attribute for an `OrderItem` object was editable. This enabled me to write shorter tests on the manager/model level.

But now, handling views and simulating user operations, I understood that `price` field should not be open to manipulation. The only value assigned to this field must be the result of a calculation (product.price * order_item.quantity).

While this may sound very obvious, I consider this a big step forward in my way to better my web development skills.

So... refactoring:

model was updated to include the price calculation and the field was set as non-editable:
```python
class OrderItem(models.Model):
    """
    individual item on a customer's order
    """
    order = models.ForeignKey(Order, on_delete=models.CASCADE, related_name='items')
    product = models.ForeignKey('products.Product', on_delete=models.CASCADE)
    quantity = models.PositiveIntegerField(default=1)
    price = models.DecimalField(max_digits=10, decimal_places=2, editable=False)  # Make price non-editable

    objects = OrderItemManager()

    def calculate_price(self):
        if not self.product or not self.quantity:
            raise ValueError("Cannot calculate price without a product or quantity of items.")
        return self.quantity * self.product.price

    def save(self, *args, **kwargs):
        # Always calculate price before saving
        self.price = self.calculate_price()
        super().save(*args, **kwargs)
```

`OrderItemManager` was updated as well, and I removed the calculation part, knowing it would be handled by the model:

```python
    def update_order_item(self, order_item_id: int, **kwargs) -> Optional['OrderItem']:
        try:
            with transaction.atomic():
                order_item = self.get(id=order_item_id)
                for key, value in kwargs.items():
                    if key == 'quantity':
                        old_quantity = order_item.quantity
                        new_quantity = value
                        change_in_stock = old_quantity - new_quantity
                        updated_stock = order_item.product.stock + change_in_stock
                        updated_product = ProductManager.update_product(
                            self, order_item.product, stock=updated_stock)
                        if not updated_product:
                            raise ValidationError(
                                f"Failed to update order item with this attributes: "
                                f"order_item= {order_item}, key= {key}, value= {value}"
                            )
                        updated_product.save()
                    setattr(order_item, key, value)
                order_item.save()  # Automatically calculates price
                return order_item
        except self.model.DoesNotExist:
            logger.error(f"No order_item found for id {order_item_id}")
            return None
        except ValidationError as e:
            logger.error(f"An error occurred: {str(e)}", exc_info=True)
            raise
        except Exception as e:
            log_level = EXCEPTION_LOG_LEVELS.get(type(e), logging.ERROR)
            logger.log(log_level, f"An error occurred: {str(e)}", exc_info=True)
            return None
```

Now I need to add some testing, to make sure I didn't forget any thing.

---

General thoughts:

1. All this work is part of the issue "implement and test orders views". I didn't plan for so many side work and refactoring, so this is a bit discouraging.
2. Writing this blog and keeping track of my work helps here: instead of thinking "oh, another day went by and I haven't closed the issue yet", I can see the work I'm putting it and reflect on it.
3. I wanted to write tests that make sure various elements are presented only when expected (for example, a customer should not be able to see "update" button when viewing a product. A staff member should). But seeing this issue is taking longer than I thought, I'll postpone this for later.
4. Previous point makes me especially proud, since this "letting go" and breaking down an issue to merge done work in shorter intervals was one of my weaknesses.

