# When to Mock

Mock at **system boundaries** only:

- External APIs (payment, email, etc.)
- Databases (sometimes - prefer test DB)
- Time/randomness
- File system (sometimes)

Don't mock:

- Your own classes/modules
- Internal collaborators
- Anything you control

## Designing for Mockability

At system boundaries, design interfaces that are easy to mock:

**1. Use dependency injection**

Pass external dependencies in rather than creating them internally. What this looks like depends on the language: a passed-in module/behaviour in Elixir, a trait object or generic in Rust, an interface in Go, a function value or interface in F#.

```
// Easy to mock
function process_payment(order, payment_client):
  return payment_client.charge(order.total)

// Hard to mock
function process_payment(order):
  client = new StripeClient(env.STRIPE_KEY)
  return client.charge(order.total)
```

**2. Prefer SDK-style interfaces over generic fetchers**

Create specific functions for each external operation instead of one generic function with conditional logic:

```
// GOOD: Each function is independently mockable
get_user(id)
get_orders(user_id)
create_order(data)

// BAD: Mocking requires conditional logic inside the mock
fetch(endpoint, options)
```

The SDK approach means:
- Each mock returns one specific shape
- No conditional logic in test setup
- Easier to see which operation a test exercises
- Each operation can carry its own precise types/specs (structs in Go/Rust, typed records in F#, structs/schemas in Elixir)
