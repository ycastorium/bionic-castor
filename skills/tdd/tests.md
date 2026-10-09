# Good and Bad Tests

## Good Tests

**Integration-style**: Test through real interfaces, not mocks of internal parts.

```
// GOOD: Tests observable behavior
test "user can checkout with valid cart":
  cart = create_cart()
  cart.add(product)
  result = checkout(cart, payment_method)
  assert result.status == confirmed
```

Characteristics:

- Tests behavior users/callers care about
- Uses public API only
- Survives internal refactors
- Describes WHAT, not HOW
- One behavior per test; several assertions on the same result are fine

## Bad Tests

**Implementation-detail tests**: Coupled to internal structure.

```
// BAD: Tests implementation details
test "checkout calls payment_service.process":
  mock_payment = mock(payment_service)
  checkout(cart, payment)
  assert mock_payment.process was called with cart.total
```

Red flags:

- Mocking internal collaborators
- Testing private methods
- Asserting on call counts/order
- Test breaks when refactoring without behavior change
- Test name describes HOW not WHAT
- Verifying through external means instead of interface

```
// BAD: Bypasses interface to verify
test "create_user saves to database":
  create_user(name: "Alice")
  row = db.query("SELECT * FROM users WHERE name = ?", "Alice")
  assert row is not empty

// GOOD: Verifies through interface
test "create_user makes user retrievable":
  user = create_user(name: "Alice")
  retrieved = get_user(user.id)
  assert retrieved.name == "Alice"
```

## Tests Not Worth Writing

These are correct, pass, and catch nothing. Delete them.

```
// WASTE: pass-through, no decision
test "order.total returns total":
  order = Order(total: 10)
  assert order.total == 10

// WASTE: tests the framework
test "router maps /users to UsersController":
  assert routes["/users"] == UsersController

// WASTE: same branch, different input, five times
test "discount for 10 items": ...
test "discount for 11 items": ...
test "discount for 12 items": ...

// BETTER: one test per outcome plus the boundary
test "no discount below 10 items":   # 9
test "discount applies from 10 items":  # 10
```

```
// WASTE: test infrastructure for two uses
class OrderTestBuilder:
  with_items(n) ...
  with_discount(d) ...
  build() ...

// BETTER: inline; repeat three lines rather than own a builder
test "discount applies from 10 items":
  order = Order(items: ten_items(), discount: none)
  assert checkout(order).total == 90
```
