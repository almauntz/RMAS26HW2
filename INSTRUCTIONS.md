# Homework 2 – Jetpack Compose: state, input and calculation

**Course:** Mobile Application and Services Development (RMAS), 2026/27
**Published:** Saturday, 7 Nov 2026 at 23:00
**Deadline:** Saturday, 14 Nov 2026 at 23:00 (last commit on the `main` branch)
**Defense:** live, in a separate slot outside the lab (the slot will be announced on Google Classroom)

Choose **one of two homework versions**:

| Version | Topic | Points |
|---|---|---|
| **A** | Build your own pizza | 6 (+2 for the bonus, section 4.3) |
| **B** | Split the bill | 8 |

Write your chosen version in `README.md` (the `Homework version:` field).

Together with the instructions you get **screen recordings of the reference apps for both versions** (on Google Classroom). The recordings show what the app should do. **The look does not have to be the same** (colours, layout, icons, top and bottom bars are an example, not a requirement), but all rules from the instructions must work.

The text in your app (labels, messages) can be in Bosnian or English.

---

## 1. AI rules

- **You may use AI to build** the homework.
- **The defense is without AI.** You must explain every part of your code: where the state lives, who changes it, why the screen redraws, and how you implemented the rules of the task. At the defense you also get an **extra task**: a change to one rule, which you make yourself, live.
- If you cannot explain the code you submitted, the homework is not accepted, even if the app works.

> Tip: AI easily builds a screen that *looks* right. The hard part is rules that affect each other. So check your app against the **reference examples** at the end of each version: your result must match exactly, to the last fening.

## 2. What we have not covered in lectures

### 2.1 Input components

We have not covered input components. **Part of the homework is to find and learn them from the documentation:**

- `TextField` / `OutlinedTextField`
- `Checkbox`, `RadioButton`, `Switch`
- `FilterChip` (or `SegmentedButton`)

Documentation: https://developer.android.com/develop/ui/compose/components and https://developer.android.com/develop/ui/compose/text/user-input

All of these work on the same principle as our `WindowControls`: **they receive a value and report an event** (`value` + `onValueChange`, `checked` + `onCheckedChange`, `selected` + `onClick`). They remember nothing themselves: if you don't change the state on the event, nothing changes on screen. You must be able to explain this at the defense.

Compose has no ready-made "group" of RadioButtons. You build the group yourself from several `RadioButton`s.

### 2.2 Project setup (Gradle)

- **Create the project inside your repository:** first clone the repository (e.g. **File → New → Project from Version Control** in Android Studio, or `git clone`), then **New Project → Empty Activity**, and choose the cloned repository folder as the **Save location**. Android Studio will warn that the folder is not empty, which is fine. The files `README.md`, `UPUTE.md` and `INSTRUCTIONS.md` should stay in the root, next to the `app` folder.
- Language Kotlin, Minimum SDK as offered.
- **No extra libraries are needed.** The template already includes everything you need: Compose, Material 3, `@Preview` and JUnit for unit tests. Don't add libraries an AI suggests unless you know why.
- **Don't upgrade versions** in the middle of the homework. You can ignore the yellow warnings "A newer version of … is available".
- **Icons:** the easiest way is right-click on `res` → **New → Vector Asset → Clip art**. The icon becomes a file in the project, with no library. If you do use `Icons.Default...`, add `implementation("androidx.compose.material:material-icons-core")` to `app/build.gradle.kts` (no version, the Compose BOM provides it).
- **`TopAppBar`** requires `@OptIn(ExperimentalMaterial3Api::class)` above the function that uses it.
- **Checking the calculation:** a `main` function cannot be run in an Android project (error "SourceSet with name 'main' not found"). Check your calculation with a **unit test** in `src/test/...` (`ExampleUnitTest`), using the green arrow next to the test.
- **Screen rotation is not part of the homework.** You don't need `ViewModel`, `@Parcelize` or `Saver`.

## 3. Common requirements (both versions)

These are graded in both versions.

### 3.1 Logic separate from UI
- The data model and the calculation are in a **separate file** with **no `androidx.compose` imports**. It is plain Kotlin, as in Homework 1.
- The calculation is a **pure function**: it takes the state, returns a result, and changes nothing.
- **Money is calculated in fening as `Int`** (12.50 KM = 1250). Do not use `Double` for money.
- Amounts are displayed as e.g. "12,50 KM" (decimal comma, always two decimals).
- Rounding percentages: **to the nearest fening, with half a fening rounded up** (e.g. 1.875 KM → 1.88 KM).

### 3.2 State

The app has **three things** to tell apart: data, UI state and the result. Each has its own place.

**1. App data** (what the user has chosen and entered: pizzas, toppings, coupon, people, items…)
- Hoisted into **one** composable at the top of the screen, as in the lecture on state. The components below it are **stateless**.

**2. UI state** (we have not covered this in detail)

Some things on screen need to remember something that is not app data. The most common example is **scrolling**.

- A `Column` does not scroll by itself. If the content does not fit on the screen, whatever is below is simply not visible.
- To make it scroll, you add a vertical scroll modifier: `Modifier.verticalScroll(...)`. That modifier needs a **scroll state**, an object that remembers how far the user has scrolled.
- That state is created with `rememberScrollState()`. The name already says `remember`: it is kept between redraws, just like `remember { mutableStateOf(...) }`.

```kotlin
Column(
    modifier = Modifier.verticalScroll(rememberScrollState())
) {
    // screen content
}
```

- **Why it must be remembered:** when the user clicks a checkbox, the screen is redrawn. If the scroll state were created anew on every redraw, the position would go back to 0 and the screen would **jump to the top** after every click.
- **Where it lives:** in the composable that scrolls, not at the top with the data. Nobody else cares about the scroll position: not the bill, not the cards, not the calculation. That is why this is an exception to the rule "components have no `remember`". That rule applies to **app data**, not to UI state.

**3. The result** (the bill, the amounts)
- **Is not stored as separate state.** It is calculated from the data every time the screen is drawn. At the defense you must explain why this is better than remembering it and updating it by hand.

### 3.3 Lists
- Both versions have lists the user changes (adds and removes items).
- Show lists with **`Column` + `forEach`**, with scrolling (see 3.2). `LazyColumn` is not needed.
- Lists in state are not changed "in place".

### 3.4 Code and look
- Every composable that draws something on screen has a `modifier: Modifier = Modifier` parameter.
- **Colours only from the theme**, no `Color.Red` and similar.
- **A `@Preview` for every component** with fixed data.
- In addition, **two `@Preview`s of the whole screen** filled with data from two reference examples (your choice).
- The project opens and runs in Android Studio without errors (Empty Activity, Kotlin, Jetpack Compose), in the root of the repository.

---

## 4. Version A – Build your own pizza

An app where the user builds their own pizza (there are no ready-made types like Margherita) and makes an order of **one or more pizzas**, and the bill updates live as they change their choices.

### 4.1 Prices

**Base price of a pizza.** Every pizza includes **tomato sauce and cheese** in the base price.

| Size | Price |
|---|---|
| S (small) | 8.00 KM |
| M (medium) | 11.00 KM |
| L (large) | 14.00 KM |

**Toppings.** The price of a topping depends on the pizza size:

| Topping | Type | S | M | L |
|---|---|---|---|---|
| mushrooms, olives, extra cheese, chicken bologna, chicken salami | regular | 1.00 | 1.50 | 2.00 |
| dried beef, sujuk, tuna | premium | 2.00 | 3.00 | 4.00 |

### 4.2 Rules for one pizza

1. **Size** S / M / L. The default is M.
2. **Maximum number of toppings:** S up to 2, M up to 3, L up to 5. When the limit is reached, the checkboxes for the other toppings are **disabled**.
3. **Reducing the size:** if the pizza has more toppings than the new size allows, **the most recently added toppings are removed** until the count is within the limit. A message is shown below the pizza, e.g. "Removed: tuna, sujuk".
4. **Quantity** with − and + buttons, from 1 to 10. The − button is disabled at 1, and + at 10.

Line price = (base price + toppings) × quantity.

### 4.3 Bonus (+2 points): half-and-half

**Half-and-half** (Switch) exists **only for L**:
- The switch is disabled when the size is not L.
- While half-and-half is on, S and M are disabled.
- Each half has its own toppings, **at most 3 per half**.
- A topping on a half costs **50% of the L price** (regular 1.00 KM, premium 2.00 KM).
- When half-and-half is **turned on**, the current toppings become the left half's toppings (at most 3; any extra are removed and a message is shown, as in rule 4.2.3).
- When half-and-half is **turned off**, the left half's toppings become the whole pizza's toppings, and the right half's toppings are discarded.

### 4.4 Rules for the order

1. **List of pizzas:** an "Add pizza" button adds a new pizza (M, no toppings, quantity 1). Each pizza has a delete button, which is disabled when the order has only one pizza.
2. **Coupon** (TextField). Upper/lower case does not matter, and leading and trailing spaces are ignored. Only one coupon can be entered:

   | Code | Effect |
   |---|---|
   | `STUDENT15` | 15% off the pizza subtotal |
   | `MINUS5` | 5.00 KM off, **only if the pizza subtotal is at least 25.00 KM**. Otherwise the message "MINUS5 is valid from 25.00 KM" is shown and there is no discount. |
   | `DOSTAVA0` | free delivery |
   | empty | no coupon |
   | anything else | the message "Unknown coupon" **below the field**, and the bill is calculated without a coupon |

   Model the coupon types as a **sealed type**.
3. **Delivery** (Switch) costs 3.00 KM. Delivery is **free if the amount after the discount is at least 30.00 KM**, or if `DOSTAVA0` is entered. Note: **the discount is subtracted before checking the 30 KM threshold.**

### 4.5 Bill display

A line-by-line bill, live:
- for each pizza: description (size and toppings, e.g. "Medium: mushrooms, tuna", or "Medium (sauce, cheese)" with no toppings), quantity, line price;
- pizza subtotal;
- discount (if any);
- delivery (0.00 KM if free, with a note why);
- **total**.

### 4.6 Reference examples (version A)

Your app must give exactly these amounts for these inputs.

| # | Order | Delivery | Coupon | Pizza subtotal | Discount | Delivery | **Total** |
|---|---|---|---|---|---|---|---|
| A1 | 1× M: extra cheese, tuna | no | – | 15.50 | 0.00 | 0.00 | **15.50 KM** |
| A2 | 2× L: extra cheese, mushrooms, sujuk | yes | `student15` | 44.00 | 6.60 | 0.00 | **37.40 KM** |
| A3 | 1× L: extra cheese, tuna + 1× M: extra cheese, mushrooms | yes | `" MINUS5 "` | 34.00 | 5.00 | 3.00 | **32.00 KM** |
| A4 | 2× S: extra cheese | yes | `MINUS5` | 18.00 | 0.00 (message) | 3.00 | **21.00 KM** |
| A5 | 1× M: extra cheese | yes | `STUDENT15` | 12.50 | 1.88 | 3.00 | **13.62 KM** |
| A6 | 1× M, no toppings | yes | `DOSTAVA0` | 11.00 | 0.00 | 0.00 | **11.00 KM** |
| A7 | 1× S: dried beef | yes | `PIZZA50` | 10.00 | 0.00 ("Unknown coupon") | 3.00 | **13.00 KM** |
| A8 (bonus) | 1× L half-and-half (left: extra cheese, tuna · right: mushrooms, olives, chicken bologna) + 1× M: extra cheese, mushrooms | yes | `MINUS5` | 34.00 | 5.00 | 3.00 | **32.00 KM** |

Individual line prices: A1 15.50 · A2 22.00 × 2 · A3 20.00 and 14.00 · A4 9.00 × 2 · A5 12.50 · A6 11.00 · A7 10.00 · A8 20.00 and 14.00.

---

## 5. Version B – Split the bill

A group of friends went out together for ćevapi, pizza or coffee. The app splits the bill: each person pays for what they ordered, shared items are split, and at the end everyone sees how much they pay.

### 5.1 People

1. A list of people, **from 1 to 8**. At the start there is one person ("Person 1"). An "Add person" button and a delete button next to each person. Deleting is disabled when only one person is left, and adding when there are 8.
2. The name is a TextField. The name **must not be empty** (spaces don't count) and **must not repeat** (case does not matter: "amar" and "Amar" are the same name).
3. When a person is deleted, they are also removed from every item they were on.

### 5.2 Bill items

Each item has:
- a **name** (TextField, must not be empty);
- a **unit price** (TextField);
- a **quantity** with − and + buttons, from 1 to 20;
- **who shares the item**: a choice for each person (e.g. checkbox or chip). **At least one person** must be selected.

An "Add item" button adds a new item (empty, quantity 1, no one selected), and each item has a delete button.

The **unit price** is valid if:
- it contains only digits and at most one decimal separator (**both comma and dot** are accepted: `12,5`, `12.50`, `3`);
- it has at most two decimals;
- it is greater than 0 and at most 1000.00 KM.

An invalid price shows an error **below that field** (e.g. for `12,5a`: "Invalid price"). Reading the price from text is a separate function in the logic, and its result (valid price or error) is a **sealed type**.

### 5.3 Options

- **Tip**: a choice of 0%, 5%, 10%, 15% or 20%. The default is 10%.
- **Round up** (Switch): each person's amount is rounded up to the nearest 0.50 KM.

### 5.4 Calculation rules

For each item:
1. `item amount = price × quantity`.
2. The amount is split **into equal parts** among the people sharing the item. If it cannot be split exactly in fening, **the remaining fening go one each to people in the order they appear in the list of people** (first, then second…). Example: 25.00 KM among three gives 8.34 + 8.33 + 8.33.

For each person:
3. `base` = the sum of their shares of all items.
4. `tip = base × percentage` (rounded to the fening, half up).
5. `total = base + tip`.
6. If "Round up" is on: `final` = `total` rounded up to the next multiple of 0.50 KM (if it is already a multiple, it stays the same). Otherwise `final = total`.

Overall:
- bill = the sum of all item amounts;
- tip = **the sum of the people's tips** (not calculated separately on the whole bill);
- rounding = the sum of (`final` − `total`) over all people;
- **total to pay = the sum of everyone's `final`.**

### 5.5 Errors

If there is **any error** (empty or repeated name, empty item name, invalid price, item with no person), **the result is not shown**. Instead, a list of what needs fixing is shown, e.g.:
- "Person 3: the name is repeated"
- "Item 2 (Cola): invalid price"
- "Item 3: no person selected"

The error is also shown at the field itself.

### 5.6 Result display

- For each person: base, tip, total, and final (if rounding is on).
- Overall: bill, tip, rounding, **total to pay**.

### 5.7 Reference examples (version B)

In all examples the people are, in this order: **Amar, Lejla, Dino.**

**Items for B1 and B2:**

| Item | Price | Qty | Shared by |
|---|---|---|---|
| Ćevapi 10 | 12.00 | 1 | Amar |
| Burger (pljeskavica) | 10.50 | 1 | Lejla |
| Coffee | 2.00 | 3 | Amar, Lejla, Dino |
| Cola | 3.50 | 1 | Dino |

**B1:** tip 10%, rounding **off**

| Person | Base | Tip | Total |
|---|---|---|---|
| Amar | 14.00 | 1.40 | 15.40 |
| Lejla | 12.50 | 1.25 | 13.75 |
| Dino | 5.50 | 0.55 | 6.05 |

Overall: bill 32.00 · tip 3.20 · rounding 0.00 · **total 35.20 KM**

**B2:** tip 10%, rounding **on**

| Person | Base | Tip | Total | Final |
|---|---|---|---|---|
| Amar | 14.00 | 1.40 | 15.40 | 15.50 |
| Lejla | 12.50 | 1.25 | 13.75 | 14.00 |
| Dino | 5.50 | 0.55 | 6.05 | 6.50 |

Overall: bill 32.00 · tip 3.20 · rounding 0.80 · **total 36.00 KM**

**Items for B3 and B4:**

| Item | Price | Qty | Shared by |
|---|---|---|---|
| Pizza | 25.00 | 1 | Amar, Lejla, Dino |
| Juice | 3.10 | 1 | Lejla, Dino |

**B3:** tip 15%, rounding **off**

| Person | Base | Tip | Total |
|---|---|---|---|
| Amar | 8.34 | 1.25 | 9.59 |
| Lejla | 9.88 | 1.48 | 11.36 |
| Dino | 9.88 | 1.48 | 11.36 |

Overall: bill 28.10 · tip 4.21 · rounding 0.00 · **total 32.31 KM**

**B4:** tip 15%, rounding **on**

| Person | Base | Tip | Total | Final |
|---|---|---|---|---|
| Amar | 8.34 | 1.25 | 9.59 | 10.00 |
| Lejla | 9.88 | 1.48 | 11.36 | 11.50 |
| Dino | 9.88 | 1.48 | 11.36 | 11.50 |

Overall: bill 28.10 · tip 4.21 · rounding 0.69 · **total 33.00 KM**

**B5 (errors):** the same people and items as B1, but the price of Cola is `3,5a`, Coffee has no person selected, and the third person is named `amar`. The result is not shown. The error list contains: the repeated name (for both people with that name), the item with no person (Coffee) and the invalid price (Cola).

---

## 6. What to submit

In the repository (on the `main` branch, by the deadline):
- the Android Studio project in the root of the repository;
- `README.md` with your homework version, your full name, and **answers to the questions** in the README. Write the answers in your own words, 2–5 sentences each. You will be asked about them at the defense.

## 7. Grading

- **Version A** is worth **6 points**. The half-and-half bonus, described in section **4.3**, is worth an additional **2 points** (8 at most).
- **Version B** is worth **8 points**.

Points are awarded **only after a successful defense**. The defense includes questions about the code and an extra task you do yourself, without AI.
