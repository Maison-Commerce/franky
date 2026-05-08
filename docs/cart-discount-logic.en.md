# Cart Discounts in the Franky Theme — Client Guide

A short guide explaining where and how to change discounts in Shopify admin.

---

## ⚠️ Main Rule

Discounts are managed in **three places** depending on the discount type:

1. **Customizer** (theme) — shows the discounted price on the product page (for all types)
2. **Every Possible Discount** (Admin → Apps) — applies Packs / Bundle / Pre-order discounts at checkout
3. **Loop Subscriptions** (Admin → Apps) — applies the subscription discount at checkout

If you change only one of them, the customer will see one price on the site and a different price at checkout. **Always update all relevant places.**

### How Every Possible Discount is Set Up

The app has 5 active discount functions (Tiered, Automatic). Each one covers a specific scenario:

| Function | When it applies | Type |
|----------|-----------------|------|
| `packs_standard` | Packs mode, regular purchase | Fixed amount per item |
| `packs_preorder` | Packs mode + pre-order | Fixed amount per item |
| `bundle_standard` | Bundle mode, regular purchase | Tiered Percentage |
| `bundle_preorder` | Bundle mode + pre-order | Tiered Percentage |
| `classic_preorder` | Classic mode + pre-order | Tiered Percentage |

When a product is added to the cart, the theme automatically attaches a "tag" (`_mc_discount_bucket`) with the matching value, and the app finds the corresponding function and applies its tiered discount.

### How Discounts Stack at Checkout (Important!)

Shopify checkout stacks discounts **linearly** off the variant base price:

```
final = (variant × Loop_subscription_%) − EPD_fixed_amount_per_item
```

First Loop subtracts 15% (if subscription is selected), then EPD subtracts the fixed pack discount. This affects how the EPD values are configured — for pre-order, for example, EPD must account for **both pack discount and pre-order discount** combined.

---

## Discount Types

### 1. Pack quantity discount — Fixed price per unit

Currently:
- 1 Bag → $39.95/ea
- 2 Bags → $37.95/ea (-$2 from base)
- 3 Bags → $35.95/ea (-$4 from base)

**Where to change it:**

- **PDP price:** Customizer → section **Maison PDP Hero** → **Pack Option** blocks → `Price per unit (cents)` field
  - `3995` = $39.95
  - `3795` = $37.95
  - `3595` = $35.95
- **Checkout discount:** Admin → Apps → Every Possible Discount → function `packs_standard` → tiers (Fixed amount per item):
  - 1 → $0.00 off
  - 2 → $2.00 off
  - 3 → $4.00 off

### 2. Pre-order discount

A discount for products that are currently out of stock (pre-order). Currently -15% from base.

**Where to change it:**

- **PDP %:** Customizer → **Theme settings** → **Pre-Order** → `Pre-Order Discount %` (e.g. `15`)
- **Checkout discount:** Admin → Apps → Every Possible Discount → function `packs_preorder` → tiers:
  - 1 → $5.99 off (= 15% of $39.95)
  - 2 → $7.99 off (= $5.99 pre-order + $2.00 pack discount)
  - 3 → $9.99 off (= $5.99 pre-order + $4.00 pack discount)

### 3. Subscription discount (Subscribe & Save)

Discount for subscribing. Currently -15%. Applied **not through Every Possible Discount**, but through **Loop Subscriptions** (selling plan).

The subscription has 3 plans, each tied to a quantity:

| Quantity | Plan ID | Frequency |
|----------|---------|-----------|
| 1 Bag | #5080383721 | Every 4 weeks |
| 2 Bags | #5271978217 | Every 6 weeks |
| 3 Bags | #5272010985 | Every 8 weeks |

**Where to change it:**

1. **PDP multiplier:** Customizer → section **Maison PDP Hero** → `Subscription Multiplier` (`0.85` = 15% off)
2. **Plan ID per pack:** Customizer → section **Maison PDP Hero** → **Pack Option** block → `Subscription Plan ID (Loop)` field for each quantity
3. **Frequency Label per pack:** Customizer → section **Maison PDP Hero** → **Pack Option** block → `Frequency Label (subscribe card)` field (e.g. "Ships every 4 weeks")
4. **Loop discount:** Admin → Apps → Loop Subscriptions → open each plan → `discount` field (all should match — `15.00%`)

**All 4 values must match** — otherwise PDP shows one price and checkout shows another.

### 4. Bundle discount

Discount based on the number of items added to the bundle (2, 3, 4+).

**Changing this requires a developer** — values are hardcoded in the theme (Tiered Percentage in EPD also needs updating).

Current values:
- 2 items → 20%
- 3 items → 25%
- 4+ items → 30%

---

## How Discounts Combine (Examples)

| Scenario | PDP & Cart show |
|----------|-----------------|
| 1 bag, one-time | $39.95 |
| 2 bags, one-time | $75.90 ($37.95 × 2) |
| 3 bags, one-time | $107.85 |
| 1 bag, subscribe | $33.96 (Loop −15%) |
| 2 bags, subscribe | $63.92 ($33.96 × 2 − $4 EPD) |
| 3 bags, subscribe | $89.88 |
| 1 bag, pre-order, one-time | $33.96 |
| 2 bags, pre-order, one-time | $63.92 |
| 2 bags, pre-order, subscribe | $51.94 |

---

## Quick Reference Table

| What to change | Where |
|----------------|-------|
| Pack price per unit (PDP) | Customizer → PDP Hero → Pack Option → `Price per unit (cents)` |
| Pack price (checkout) | Admin → Apps → **EPD** → `packs_standard` → tier discount |
| Pre-order % (PDP) | Customizer → Theme settings → Pre-Order |
| Pre-order checkout discount | Admin → Apps → **EPD** → `packs_preorder` |
| Subscription % (PDP) | Customizer → PDP Hero → `Subscription Multiplier` |
| Subscription % (checkout) | Admin → Apps → **Loop Subscriptions** → each plan |
| Subscription plan ID per pack | Customizer → PDP Hero → Pack Option → `Subscription Plan ID (Loop)` |
| Subscribe frequency text | Customizer → PDP Hero → Pack Option → `Frequency Label` |
| Enable pre-order on a product | Customizer → PDP Hero → `Enable Pre-Order` |
| Bundle % | Theme code (developer required) |

---

## Examples

### Change the 2-pack price from $37.95 to $36.95

1. Customizer → PDP Hero → "2 Bags" block → `Price per unit (cents)`: `3695`
2. Admin → Apps → EPD → `packs_standard` → tier for 2 → `$3.00 off` (= $39.95 − $36.95)
3. Admin → Apps → EPD → `packs_preorder` → tier for 2 → `$8.99 off` (= $5.99 pre-order + $3.00 pack)
4. Save everywhere

### Change the pre-order discount from 15% to 20%

1. Customizer → Theme settings → Pre-Order → `Pre-Order Discount %`: `20`
2. Recalculate pre-order discount per item: $39.95 × 0.20 = $7.99
3. Admin → Apps → EPD → `packs_preorder` → update tiers:
   - 1 → $7.99
   - 2 → $9.99 (= $7.99 + $2.00)
   - 3 → $11.99 (= $7.99 + $4.00)

### Add a new pack (e.g. 5 Bags @ $33.95/ea)

1. Customizer → PDP Hero → add a **Pack Option** block:
   - `Pack Name`: "5 Bags"
   - `Number of Bottles to Display`: `5`
   - `Price per unit (cents)`: `3395`
   - `Subscription Plan ID (Loop)`: new plan ID from Loop (you need to create one in Loop first)
   - `Frequency Label`: "Ships every 14 weeks" (for example)
2. Admin → Apps → EPD → `packs_standard` → add tier: 5 → `$6.00 off`
3. Admin → Apps → EPD → `packs_preorder` → add tier: 5 → `$11.99 off`
4. Admin → Apps → Loop Subscriptions → create a new plan with 15% discount (if not yet)

### Disable the pre-order discount

1. Customizer → Theme settings → Pre-Order → `Pre-Order Discount %`: `0`
2. Admin → Apps → EPD → disable functions `packs_preorder`, `bundle_preorder`, `classic_preorder` (toggle on the right)

### Change subscription discount from 15% to 20%

1. Customizer → PDP Hero → `Subscription Multiplier`: `0.80`
2. Admin → Apps → Loop Subscriptions → open each plan (4w / 6w / 8w) → change discount to `20.00%`

---

## Cart Merge Behaviour

When a pack is added to the cart, the theme **automatically merges** with an existing line item (same variant + same mode + same pre-order state, combined qty ≤ 3) and reassigns the correct plan ID.

Example:
- Cart: 1 Bag subscribe (4-week plan)
- User adds 2 Bags subscribe → **one line: 3 Bags @ 8-week plan, $35.95/ea**

If combined qty > 3 — a **separate line** is created (no merge).

When quantity changes via cart drawer (`+` / `−` buttons) — the plan also auto-swaps to the correct one for the new quantity.

---

## Checklist After Changing Discounts

- [ ] Customizer — Save
- [ ] EPD — all relevant functions updated
- [ ] Loop — discount updated in plan (if subscription was touched)
- [ ] Open the product in **incognito** — verify the price on the page
- [ ] Add to cart — verify the price in the cart
- [ ] Proceed to checkout — verify the final amount
- [ ] Subscription — verify the correct plan ID was assigned for each quantity
