# Знижки в темі Franky — як змінювати

Короткий гайд для клієнта: де і що міняти в Shopify admin, щоб змінити знижки.

---

## ⚠️ Головне правило

Знижки керуються у **трьох місцях** залежно від типу:

1. **Customizer** (тема) — показує знижену ціну на сторінці товару (для всіх типів)
2. **Every Possible Discount** (Admin → Apps) — застосовує знижки Packs / Bundle / Pre-order на checkout
3. **Loop Subscriptions** (Admin → Apps) — застосовує знижку за підписку на checkout

Якщо змінити лише одне — клієнт побачить одну ціну на сайті і іншу на оплаті. **Завжди оновлюй всі задіяні місця.**

### Як влаштовано Every Possible Discount

В апці є 6 активних discount-функцій (Tiered, Automatic). Кожна — для свого сценарію:

| Функція | Коли застосовується | Тип |
|---------|---------------------|-----|
| `packs_standard` | Режим Packs (PDP), звичайна купівля | Fixed amount per item |
| `packs_preorder` | Режим Packs (PDP) + pre-order | Fixed amount per item |
| `bundle_standard` | Режим Bundle (PDP), звичайна купівля | Tiered Percentage |
| `bundle_preorder` | Режим Bundle (PDP) + pre-order | Tiered Percentage |
| `classic_preorder` | Режим Classic (PDP) + pre-order | Tiered Percentage |
| **`bundle_listicle`** | **Секція Listicle Buy Box (landing-page funnel)** | **Tiered Percentage** |

Тема при додаванні товару в корзину автоматично ставить "тег" (`_mc_discount_bucket`) з відповідним значенням, і апка знаходить потрібну функцію та застосовує її tiered-знижку.

### Як стекаються знижки на checkout (важливо!)

Shopify checkout стекає знижки **лінійно** від базової ціни варіанта:

```
final = (variant × Loop_subscription_%) − EPD_fixed_amount_per_item
```

Спочатку Loop віднімає 15% (якщо є підписка), потім EPD віднімає фіксований pack discount. Це впливає на те, як виставлені значення EPD — для pre-order, наприклад, EPD має враховувати **і pack discount, і pre-order discount** одночасно.

---

## Типи знижок

### 1. Знижка за кількість паків (Pack discount) — Fixed price per unit

Зараз:
- 1 Bag → $39.95/ea
- 2 Bags → $37.95/ea (-$2 від базової)
- 3 Bags → $35.95/ea (-$4 від базової)

**Де змінити:**

- **PDP ціна:** Customizer → секція **Maison PDP Hero** → блоки **Pack Option** → поле `Price per unit (cents)`
  - `3995` = $39.95
  - `3795` = $37.95
  - `3595` = $35.95
- **Checkout discount:** Admin → Apps → Every Possible Discount → функція `packs_standard` → tiers (Fixed amount per item):
  - 1 → $0.00 off
  - 2 → $2.00 off
  - 3 → $4.00 off

### 2. Pre-order знижка

Знижка для товарів, які зараз out of stock (клієнт замовляє наперед). Зараз -15% від базової.

**Де змінити:**

- **PDP %:** Customizer → **Theme settings** → **Pre-Order** → `Pre-Order Discount %` (наприклад `15`)
- **Checkout discount:** Admin → Apps → Every Possible Discount → функція `packs_preorder` → tiers:
  - 1 → $5.99 off (= 15% від $39.95)
  - 2 → $7.99 off (= $5.99 pre-order + $2.00 pack discount)
  - 3 → $9.99 off (= $5.99 pre-order + $4.00 pack discount)

### 3. Subscription знижка (Subscribe & Save)

Знижка за підписку. Зараз -15%. Застосовується **не через Every Possible Discount**, а через **Loop Subscriptions** (selling plan).

Підписка має 3 плани, які прив'язані до кількості:

| Quantity | Plan ID | Frequency |
|----------|---------|-----------|
| 1 Bag | #5080383721 | Every 4 weeks |
| 2 Bags | #5271978217 | Every 6 weeks |
| 3 Bags | #5272010985 | Every 8 weeks |

**Де змінити:**

1. **PDP multiplier:** Customizer → секція **Maison PDP Hero** → `Subscription Multiplier` (`0.85` = 15% off)
2. **Plan ID per pack:** Customizer → секція **Maison PDP Hero** → блок **Pack Option** → поле `Subscription Plan ID (Loop)` для кожної кількості
3. **Frequency Label per pack:** Customizer → секція **Maison PDP Hero** → блок **Pack Option** → поле `Frequency Label (subscribe card)` (наприклад "Deliver every 4 weeks")
4. **Loop discount:** Admin → Apps → Loop Subscriptions → відкрити кожен plan → поле `discount` (мають всі бути однакові — `15.00%`)

**Усі 4 значення мають співпадати** — інакше PDP покаже одне, а checkout інше.

### 4. Bundle знижка (набори)

Знижка залежно від кількості доданих товарів до набору (2, 3, 4+).

**Для зміни потрібен розробник** — значення прописані в коді теми (Tiered Percentage у EPD теж треба оновити).

Поточні значення:
- 2 items → 20%
- 3 items → 25%
- 4+ items → 30%

### 5. Listicle bundle знижка (нова — для landing-page funnel)

Окрема знижка для секції **Listicle Buy Box** яка вмонтована в listicle сторінки (наприклад `maison-listicle-morning`). Працює як Tiered Percentage по quantity:

| Quantity | Discount (від variant base) |
|----------|------------------------------|
| 1 | 0% |
| 2 | **10%** |
| 3 | **15%** |

Знижка від підписки (15% через Loop) **стекається додатково** — для 2/3 packs subscribe фактична знижка = bundle% × subscription% (compound):

| Сценарій | Math | Per unit | Total |
|----------|------|----------|-------|
| 1 bag, one-time | $39.95 | $39.95 | $39.95 |
| 2 bags, one-time | $39.95 × 0.9 | $35.96 | $71.91 |
| 3 bags, one-time | $39.95 × 0.85 | $33.96 | $101.87 |
| 1 bag, subscribe | $39.95 × 0.85 | $33.96 | $33.96 |
| 2 bags, subscribe | round(3596 × 0.85) | $30.57 | **$61.14** |
| 3 bags, subscribe | round(3396 × 0.85) | $28.87 | **$86.61** |

**Де змінити:**

- **PDP ціни (one-time):** Customizer → секція **Listicle Buy Box** → блоки **Pack Option** → поле `Price per unit (cents)` АБО `Discount multiplier` (наприклад `0.9` для 10% off).
- **Subscription %:** Customizer → секція **Listicle Buy Box** → `Subscription discount (%)` (default `15`)
- **Plan ID per pack:** ті самі що в PDP — Pack Option → `Subscription Plan ID (Loop)`
- **Checkout discount:** Admin → Apps → **Every Possible Discount** → функція `bundle_listicle` → tiers (Tiered Percentage):
  - 1 → `0%`
  - 2 → `10%`
  - 3 → `15%`
- **EPD bucket trigger** (для розробника): кожен cart item з listicle buy-box має `_mc_discount_bucket: bundle_listicle` property. Якщо клієнт хоче змінити назву bucket — це робиться через customizer `Listicle Buy Box → EPD discount bucket`, а в EPD функції теж треба змінити condition Value.

> **Pre-order не підтримується** в Listicle Buy Box (свідомо — landing funnel для in-stock товарів). EPD не має `bundle_listicle_preorder` функції.

---

## Як знижки складаються (приклади)

| Сценарій | PDP та Cart показують |
|----------|----------------------|
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

## Швидка таблиця "Що → Де"

| Що змінити | Де |
|------------|-----|
| Pack ціна за одиницю (PDP) | Customizer → PDP Hero → Pack Option → `Price per unit (cents)` |
| Pack ціна (checkout) | Admin → Apps → **EPD** → `packs_standard` → tier discount |
| Pre-order % (PDP) | Customizer → Theme settings → Pre-Order |
| Pre-order checkout discount | Admin → Apps → **EPD** → `packs_preorder` |
| Subscription % (PDP) | Customizer → PDP Hero → `Subscription Multiplier` |
| Subscription % (checkout) | Admin → Apps → **Loop Subscriptions** → кожен plan |
| Subscription plan ID per pack | Customizer → PDP Hero → Pack Option → `Subscription Plan ID (Loop)` |
| Subscribe frequency text | Customizer → PDP Hero → Pack Option → `Frequency Label` |
| Увімкнути pre-order на товарі | Customizer → PDP Hero → `Enable Pre-Order` |
| Bundle % | Код теми (потребує розробника) |
| Listicle bundle % (PDP) | Customizer → Listicle Buy Box → Pack Option → `Discount multiplier` |
| Listicle subscription % (PDP) | Customizer → Listicle Buy Box → `Subscription discount (%)` |
| Listicle bundle % (checkout) | Admin → Apps → **EPD** → `bundle_listicle` → tier discount |
| Listicle EPD bucket name | Customizer → Listicle Buy Box → `EPD discount bucket` |

---

## Приклади

### Змінити ціну 2-pack з $37.95 на $36.95

1. Customizer → PDP Hero → блок "2 Bags" → `Price per unit (cents)`: `3695`
2. Admin → Apps → EPD → `packs_standard` → tier для 2 → `$3.00 off` (= $39.95 − $36.95)
3. Admin → Apps → EPD → `packs_preorder` → tier для 2 → `$8.99 off` (= $5.99 pre-order + $3.00 pack)
4. Save усюди

### Змінити pre-order знижку з 15% на 20%

1. Customizer → Theme settings → Pre-Order → `Pre-Order Discount %`: `20`
2. Перерахувати pre-order discount per item: $39.95 × 0.20 = $7.99
3. Admin → Apps → EPD → `packs_preorder` → оновити tiers:
   - 1 → $7.99
   - 2 → $9.99 (= $7.99 + $2.00)
   - 3 → $11.99 (= $7.99 + $4.00)

### Додати новий пак (наприклад, 5 Bags @ $33.95/ea)

1. Customizer → PDP Hero → додати блок **Pack Option**:
   - `Pack Name`: "5 Bags"
   - `Number of Bottles to Display`: `5`
   - `Price per unit (cents)`: `3395`
   - `Subscription Plan ID (Loop)`: новий plan ID з Loop (треба створити план у Loop)
   - `Frequency Label`: "Deliver every 14 weeks" (наприклад)
2. Admin → Apps → EPD → `packs_standard` → додати tier: 5 → `$6.00 off`
3. Admin → Apps → EPD → `packs_preorder` → додати tier: 5 → `$11.99 off`
4. Admin → Apps → Loop Subscriptions → створити новий plan з 15% discount (якщо ще немає)

### Вимкнути pre-order знижку

1. Customizer → Theme settings → Pre-Order → `Pre-Order Discount %`: `0`
2. Admin → Apps → EPD → вимкнути функції `packs_preorder`, `bundle_preorder`, `classic_preorder` (toggle справа)

### Змінити subscription знижку з 15% на 20%

1. Customizer → PDP Hero → `Subscription Multiplier`: `0.80`
2. Admin → Apps → Loop Subscriptions → відкрити кожен plan (4w / 6w / 8w) → змінити discount на `20.00%`

---

## Cart merge поведінка

### PDP merge (Maison PDP Hero)

При додаванні pack у корзину з PDP тема **автоматично об'єднує** з existing line item (Same variant + Same mode + Same pre-order state, qty ≤ 3) і перепризначає правильний plan ID.

Приклад:
- Корзина: 1 Bag subscribe (4-week plan)
- Користувач додає ще 2 Bags subscribe → **одна лінія: 3 Bags @ 8-week plan, $35.95/ea**

Якщо combined qty > 3 — створюється **окрема лінія** (без merge).

### Listicle Buy Box (новий flow)

Не мерджить з cart — додавання завжди створює **нову line**. За замовчуванням після ATC одразу redirect на `/checkout` (`What happens after add-to-cart: Redirect to checkout`). Альтернативно в кастомайзері можна обрати "Open cart drawer (no redirect)" — тоді ATC просто відкриє drawer.

### Cart drawer quantity swap (працює на ВСІХ сторінках)

При зміні quantity у cart drawer (`+` / `−` кнопки) — plan **автоматично swap'ається** на правильний для нової кількості. Тепер працює не тільки на PDP/listicle, а й на **будь-якій сторінці** (collection, blog, home), бо `_qty_plans` property зберігається на cart line при ATC.

---

## Чеклист після зміни знижок

- [ ] Customizer — Save
- [ ] EPD — оновлено всі задіяні функції
- [ ] Loop — оновлено discount у плані (якщо торкалось підписки)
- [ ] Відкрити товар у **incognito** — перевірити ціну на сторінці
- [ ] Додати в корзину — перевірити ціну в корзині
- [ ] Пройти до checkout — перевірити фінальну суму
- [ ] Підписка — перевірити що правильний plan ID присвоївся для кожної quantity
