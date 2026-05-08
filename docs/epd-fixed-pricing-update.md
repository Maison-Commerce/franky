# Every Possible Discount — оновлення для fixed-price pack discount

Інструкція для оновлення EPD-функцій після переходу packs mode з percentage на fixed-amount discount.

---

## Контекст

Тема тепер пише в `_mc_discount_bucket` ті самі значення (`packs_standard`, `packs_preorder`), але PDP розраховує **fixed unit price** замість percentage. Щоб checkout збігався з PDP — функції в EPD теж треба перевести на Fixed Amount.

Базові ціни:
- 1 Bag → $39.95/ea
- 2 Bags → $37.95/ea (-$2 від базової)
- 3 Bags → $35.95/ea (-$4 від базової)

Pre-order = -15% від unit price → еквівалентно фіксованим знижкам.

---

## packs_standard (звичайна купівля)

**Admin → Apps → Every Possible Discount → знайти функцію `packs_standard`**

1. Відкрити функцію → змінити **Discount type** з `Percentage` на `Fixed amount per item`
2. Tiers:

| Quantity | Discount per item | Result per item |
|----------|-------------------|-----------------|
| 1 | $0.00 off | $39.95 |
| 2 | $2.00 off | $37.95 |
| 3 | $4.00 off | $35.95 |

3. Save & переконатись що toggle справа `Active`

---

## packs_preorder (pre-order купівля)

Pre-order = pack discount + 15% від unit price.

| Quantity | Unit base | Pre-order discount per item | Result per item |
|----------|-----------|------------------------------|-----------------|
| 1 | $39.95 | $5.99 off | $33.96 |
| 2 | $37.95 | $5.69 off | $32.26 |
| 3 | $35.95 | $5.39 off | $30.56 |

Кроки:

1. Відкрити функцію `packs_preorder` → **Discount type** → `Fixed amount per item`
2. Заповнити tiers за таблицею вище
3. Save & активувати

---

## bundle_standard / bundle_preorder / classic_preorder

**Не чіпаємо** — bundle і classic mode залишаються на percentage logic. Перетворення на fixed-amount тільки для packs.

---

## Subscription discount (Loop)

Subscription знижка (15%) застосовується **через Loop selling plan**, не через EPD. Жодних змін в EPD для підписки не потрібно — Loop сам застосовує -15% при checkout, коли line item має `selling_plan` ID.

---

## Як перевірити що все працює

1. Customizer → Save (щоб оновити live theme)
2. Open product PDP в incognito
3. Click "2 Bags" — має показатись `$37.95/ea` під назвою
4. Click "3 Bags" — `$35.95/ea`
5. Add to cart → checkout — фінальна сума має співпадати з PDP

Тест-матриця:

| Сценарій | Очікувана ціна на checkout |
|----------|----------------------------|
| 1 bag, one-time | $39.95 |
| 2 bags, one-time | $75.90 |
| 3 bags, one-time | $107.85 |
| 1 bag, subscribe | $33.96 (Loop -15%) |
| 2 bags, subscribe | $64.52 |
| 3 bags, subscribe | $91.67 |
| 2 bags, pre-order, one-time | $64.52 ($32.26 × 2) |

Якщо ціни на checkout відрізняються від PDP — перевір що EPD функція активна і tier-значення збереглись.
