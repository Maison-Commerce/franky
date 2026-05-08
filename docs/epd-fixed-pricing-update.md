# Every Possible Discount — fixed-amount config (узгоджено з checkout-математикою)

Інструкція для конфігурації EPD-функцій так, щоб ціни на PDP, у cart і на checkout збігались.

---

## Контекст

Shopify checkout стекає знижки **лінійно** від variant base price:

```
final = (variant × Loop_subscription_%) − EPD_fixed_amount_per_item
```

PDP-математика тепер віддзеркалює цей же порядок (комміт після цього документа). Тому EPD значення мають враховувати **і pack tier discount, і pre-order discount**, бо обидва — fixed amount у Shopify cart.

Базові ціни (PDP цільові):
- 1 Bag → $39.95/ea (повна ціна)
- 2 Bags → $37.95/ea (-$2 pack discount)
- 3 Bags → $35.95/ea (-$4 pack discount)

Pre-order discount = -15% від variant base = **$5.99/ea** (однаковий для всіх pack tiers).

---

## packs_standard (звичайна купівля, без pre-order)

**Admin → Apps → Every Possible Discount → функція `packs_standard`**

- **Discount type:** `Fixed amount per item`
- **Tiers:**

| Quantity | Discount per item | Math | Final per item |
|----------|-------------------|------|----------------|
| 1 | **$0.00** off | $39.95 − $0 | $39.95 |
| 2 | **$2.00** off | $39.95 − $2.00 | $37.95 |
| 3 | **$4.00** off | $39.95 − $4.00 | $35.95 |

При підписці Loop додатково віднімає 15% від variant base (до EPD), все стекається лінійно.

---

## packs_preorder (pre-order купівля)

**Discount = pack discount + 15% pre-order = $5.99 + pack_discount per item**

| Quantity | Pack disc | Pre-order disc (15%) | **Total off per item** | Math |
|----------|-----------|----------------------|------------------------|------|
| 1 | $0.00 | $5.99 | **$5.99** | $39.95 − $5.99 = $33.96 |
| 2 | $2.00 | $5.99 | **$7.99** | $39.95 − $7.99 = $31.96 |
| 3 | $4.00 | $5.99 | **$9.99** | $39.95 − $9.99 = $29.96 |

Кроки:

1. Відкрити функцію `packs_preorder` → **Discount type:** `Fixed amount per item`
2. Заповнити tiers:
   - 1 → **$5.99**
   - 2 → **$7.99**
   - 3 → **$9.99**
3. Save & активувати

> ⚠️ **Увага:** попередня версія цього документа давала значення $5.99 / $5.69 / $5.39 — це було помилкою (не враховувало що pack discount теж має бути в EPD для pre-order, бо Loop subscription для pre-order може стекатись теж).

---

## bundle_standard / bundle_preorder / classic_preorder

**Не чіпаємо** — bundle і classic mode залишаються на percentage logic.

---

## Subscription discount (Loop)

Subscription знижка (15%) застосовується **через Loop selling plan**, не через EPD. Loop сам віднімає -15% від variant base, коли line item має `selling_plan` ID.

EPD підлаштовується автоматично — після Loop, EPD стримує fixed amount від уже-знижeної ціни.

---

## Test matrix

| Scenario | Variant | Loop | EPD | Final per item | Total |
|----------|---------|------|-----|----------------|-------|
| 1 bag, one-time | $39.95 | — | $0 | $39.95 | $39.95 |
| 2 bags, one-time | $39.95 | — | -$2.00 | $37.95 | $75.90 |
| 3 bags, one-time | $39.95 | — | -$4.00 | $35.95 | $107.85 |
| 1 bag, subscribe | $39.95 | -15% → $33.96 | -$0 | $33.96 | $33.96 |
| 2 bags, subscribe | $39.95 | -15% → $33.96 | -$2.00 | $31.96 | **$63.92** |
| 3 bags, subscribe | $39.95 | -15% → $33.96 | -$4.00 | $29.96 | **$89.88** |
| 1 bag, pre-order, one-time | $39.95 | — | -$5.99 | $33.96 | $33.96 |
| 2 bags, pre-order, one-time | $39.95 | — | -$7.99 | $31.96 | **$63.92** |
| 3 bags, pre-order, one-time | $39.95 | — | -$9.99 | $29.96 | **$89.88** |
| 2 bags, pre-order, subscribe | $39.95 | -15% → $33.96 | -$7.99 | $25.97 | **$51.94** |

PDP після цього оновлення показуватиме точно ці значення — повна синхронізація з checkout.

---

## Чек-ліст після зміни

- [ ] EPD `packs_standard` → Fixed amount per item: $0 / $2 / $4
- [ ] EPD `packs_preorder` → Fixed amount per item: $5.99 / $7.99 / $9.99
- [ ] Перевірити PDP в incognito: 2 bags subscribe має показати $63.92
- [ ] Add to cart → cart drawer має показати ту саму суму
- [ ] Proceed to checkout → final amount = PDP amount
