# 🏥 COMPLETE SYSTEM LOGIC GUIDE — Prompt for Backend AI

**INSTRUCTIONS TO AI:** Read this entire document carefully. It explains the EXACT logic, math, and data flow for EVERY screen in EVERY portal of the Hospital Meal Planner system. Before making ANY changes, first CHECK if the feature described below already exists and works in the current codebase. If it does, skip it. If it doesn't, implement it. Do NOT blindly rebuild things that are already working.

---

## 🏥 PORTAL 1: ADMISSIONS

### `PatientRegistration.vue` — The Starting Point
- **What it does:** The Admissions Officer registers a new patient (Name, Age, Sex, Ward, Room, Bed, Admission Date, Contact Info, Allergies).
- **Critical field:** The **Allergies** field. This gets stored in the database and is later used by the Dietitian screens to trigger allergy warnings when assigning meals. Allergies should be stored as a comma-separated string or JSON array (e.g., `"Chicken, Shrimp, Peanuts"`).
- **Data flow:** When saved → the patient record must appear in the **Doctor's Portal** and **Dietitian's Portal** instantly via the same database table.
- **Ward field:** The hospital has exactly **10 Wards**. The Ward assignment is critical because Food Servers scan QR codes per ward to see which patients need food delivery.

---

## 👨‍⚕️ PORTAL 2: DOCTOR

### `PatientPrescriptionsScreen.vue` — The Diet Order
- **What it does:** The Doctor sees the list of admitted (Active) patients. They click a patient and write a **Diet Prescription**.
- **Critical fields that MUST be saved:**
  - `dietType` — e.g., "Normal Diet (Regular Diet)", "Diabetic Diet", "Low-Sodium Diet", "Renal Diet". A patient can have COMPOSITE prescriptions like "Diabetic + Low-Sodium".
  - `calorieTarget` — e.g., 1500 kcal per day. This is used later by the Dietitian to validate that assigned meals don't exceed or fall too far below this target.
  - `feedingInstructions` — Free text, e.g., "NGT feeding only", "Soft diet", "No cold liquids".
  - `status` — "Active" or "Completed". Only Active prescriptions are used by the Dietitian.
- **Data flow:** When saved → this prescription flows into the Dietitian's portal. The `dietType` is the KEY that determines which **Diet Group** the patient falls into in `DailyProduction.vue`. If a patient has no prescription yet, the system defaults them to "Normal Diet (Regular Diet)".
- **Notification:** Saving a prescription should trigger a notification to the Dietitian role: "New diet prescription for [Patient Name]".

---

## 🍽️ PORTAL 3: DIETITIAN (The Most Complex Portal — Read Carefully!)

### Screen 1: `DishMenu.vue` — The Recipe Dictionary (Master Library)

**Purpose:** This is the master library of ALL foods the hospital can potentially cook. Every other screen in the entire system pulls dish data from here. If a dish doesn't exist in DishMenu, it cannot be assigned, cooked, or purchased.

**What the Dietitian does when adding a dish:**
1. Enters the dish name (e.g., "Chicken Tinola").
2. Assigns classification tags:
   - `mealType`: Breakfast / Lunch / Dinner (which meal slot this dish belongs to)
   - `componentType`: Carbohydrate / Protein-Viand / Side-Fruit (what food group component this dish fills)
   - `dietaryTags`: An ARRAY of diet types this dish is safe for. A single dish can have MULTIPLE tags! Example: Chicken Tinola might be tagged as [Normal Diet, Low-Sodium Diet]. This means it will appear in the dropdowns for BOTH Normal and Low-Sodium patient groups.
3. Sets the **cost per serving** (₱) — This is manually entered because food prices in Manticao's local market change daily. The Dietitian estimates the cost. Later, the Purchasing Officer logs the ACTUAL price from the receipt.
4. **Maps the raw ingredients (BOM — Bill of Materials):**
   - Example: Chicken Tinola requires: 150g Chicken + 100g Sayote + 10g Ginger + 5g Salt
   - **CRITICAL:** Each ingredient quantity is for exactly **1 serving (1 patient)**. This is the foundation of all math in the system.

**The Calorie Auto-Calculation Math:**
- The `ingredients` table in the database stores nutritional facts per gram for each raw ingredient. Example: Chicken = 1.65 calories per gram, Rice = 1.30 calories per gram.
- When the Dietitian adds "150g Chicken" to the recipe, the system calculates: `150g × 1.65 cal/g = 247.5 calories`.
- The system sums ALL ingredient calories to give the total dish calories automatically.
- The Dietitian CAN override this auto-calculated number if they have a more accurate value.

**Why this screen is critical:** Because every other screen reads from this dictionary. The dropdowns in DailyProduction, MealAssignment, and MealCalendar all query this table. The Purchasing Officer's market list is calculated from these ingredient mappings. The Kitchen's prep sheet quantities are multiplied from these 1-serving amounts.

---

### Screen 2: `MealCalendar.vue` — The Weekly Cycle Planner

**Purpose:** The Dietitian pre-plans a rotating weekly menu as a TEMPLATE. Example: "Every Monday: Lunch = Tinola, Dinner = Adobo". This is for advance planning convenience only.

**What the Dietitian does:**
1. Picks a date on the calendar.
2. Assigns dishes from the Dish Menu to Breakfast / Lunch / Dinner slots for that date.
3. This creates a **suggestion template** — it does NOT assign food to any patient yet! It does NOT trigger purchasing or kitchen actions.

**Data flow:** When the Dietitian opens `DailyProduction.vue` and clicks the **"Load Cycle Menu"** button, the system reads the MealCalendar entries for today's date and auto-fills the dropdown selections. If the Dietitian doesn't like the suggestion (e.g., because market prices changed that morning), they can manually change the dropdown selections to different dishes. The calendar is a convenience tool, not a binding contract.

**Storage:** This can be stored in localStorage on the frontend OR in a `menu_calendar_entries` backend table. Either approach works because this data is purely a suggestion template and does not affect actual patient meal records.

---

### Screen 3: `MealAssignmentScreen.vue` — Individual Special Cases ONLY

**Purpose:** This screen is used ONLY for the small minority of patients who need completely custom, individual meal plans. Examples: A patient with severe multiple allergies, a patient on NGT tube feeding, a patient whose doctor ordered a very specific calorie-restricted diet. The vast majority of patients (Normal Diet) are handled in DailyProduction instead.

**What the Dietitian does:**
1. Searches for a specific patient by name.
2. Picks a date.
3. For each meal slot (Breakfast, Lunch, Dinner), selects 3 components:
   - Carb (e.g., Rice)
   - Viand/Protein (e.g., Scrambled Egg)
   - Side/Fruit (e.g., Banana)
4. The dropdowns use SearchableSelect (typeahead combobox) and ONLY show dishes that match the patient's `dietType` from their Doctor's prescription.
5. Can adjust portion multiplier (0.5x, 0.8x, 1.0x, 1.2x) for each meal.

**The 3 Rules That MUST Be Enforced on This Screen:**

1. **₱150 Daily Budget Rule:** The system adds up the cost of Breakfast + Lunch + Dinner for that patient for that day. If the total exceeds ₱150.00, the UI shows a RED warning banner and the budget progress bar turns red. The Dietitian is warned but can still save (it's a soft warning, not a hard block).

2. **Allergy Validation Rule:** The system reads the patient's `allergies` field from their Admissions record. It cross-references the allergens against the ingredient list (BOM) of the selected dish. If a patient has "Chicken" listed as an allergy and the Dietitian selects "Chicken Tinola," the system triggers a critical warning: "⚠️ ALLERGEN DETECTED: This dish contains Chicken, which is listed as an allergy for this patient."

3. **Dietary Prescription Filter:** The dropdowns ONLY display dishes whose `dietaryTags` array includes the patient's prescribed `dietType`. A Diabetic patient will never see a sugar-heavy dessert in their dropdown because that dessert was not tagged as "Diabetic Diet" in the Dish Menu.

---

### Screen 4: `DailyProduction.vue` — The Bulk Planner (MOST IMPORTANT SCREEN)

**Purpose:** This is where 90% of all meal planning happens. Instead of assigning meals patient-by-patient (which would be insane for 100+ patients), the Dietitian assigns meals to entire diet groups at once.

**How it works step-by-step:**
1. The system reads ALL currently admitted (Active) patients from the database.
2. It groups them by their `dietType` from the Doctor's prescription:
   - Example: Normal Diet (85 patients), Diabetic (10 patients), Renal (5 patients).
3. The left sidebar shows these groups with patient counts.
4. The Dietitian clicks a group (e.g., "Normal Diet — 85 patients").
5. The right panel shows 3 meal slots (Breakfast, Lunch, Dinner), each with 3 component dropdowns (Carb, Viand, Side).
6. The dropdowns use **SearchableSelect** (typeahead combobox) and only show dishes matching that group's diet type.
7. The Dietitian can click **"Load Cycle Menu"** to auto-fill from the MealCalendar, OR manually search and select dishes.
8. The ₱150 budget bar at the bottom shows the average cost per patient across all selected meals.
9. The Dietitian clicks **"Dispatch & Preview"** which opens a confirmation modal.

**What Happens on "Confirm & Dispatch" (The Critical Backend Math):**

Step A — **Create Bulk Meal Assignments:**
The system creates meal assignment records for ALL patients in that group in a SINGLE bulk API call (`POST /api/meal-assignments/bulk`). If there are 85 patients in the Normal Diet group, it creates 85 meal_assignment rows, all with the same dishes but different patient_ids.

Step B — **Calculate Aggregated Ingredient Requirements:**
The system reads the BOM (Bill of Materials / ingredient list) for each selected dish and MULTIPLIES by the number of patients:
- Chicken Tinola needs 150g Chicken per serving × 85 patients = **12,750g (12.75 kg) of Chicken needed**
- Rice needs 200g per serving × 85 patients = **17,000g (17 kg) of Rice needed**
- This is done for EVERY ingredient in EVERY selected dish.

Step C — **Aggregate Across ALL Diet Groups:**
After the Dietitian dispatches multiple groups, the system combines all ingredient needs:
- Normal Diet group needs 12.75kg Chicken + Diabetic group needs 1.5kg Chicken = **Total: 14.25kg Chicken needed today**
- This final aggregated list is what flows to the Purchasing Officer's Market List.

Step D — **Generate Kitchen Prep Sheet:**
The dispatch also generates the data that appears on the Kitchen Staff's `ProductionSchedule.vue`:
- "Cook Chicken Tinola: 85 servings (Normal) + 10 servings (Diabetic) = 95 total servings"
- Shows total raw ingredient quantities needed per dish.

---

## 🛒 PORTAL 4: PURCHASING OFFICER

### `PurchasingOfficerDashboard.vue` — The Daily Market List

**How do ingredients appear on this list?**
1. The Dietitian dispatches meals from `DailyProduction.vue`.
2. The backend reads the BOM of every dispatched dish, multiplies each ingredient by the number of servings, and AGGREGATES all identical ingredients across all diet groups into a single total.
3. This aggregated list appears as the "Daily Market List" on the Purchasing Officer's dashboard.
4. Each row shows: Ingredient Name, Total Quantity Needed, Unit (g/kg/pcs), Estimated Unit Price, Estimated Total Cost.
5. The Purchasing Officer prints this list and takes it to the Manticao market.

### "Log Purchase" Button
After buying items at the market, the Purchasing Officer returns and clicks "Log Purchase" for each item:
- **Fields:** Ingredient name, Actual quantity bought, Actual unit price paid (which may differ from the Dietitian's estimate), Supplier name (optional), Receipt reference number (optional).
- **What happens in the database:**
  1. A new record is inserted into the `purchase_history` table (for auditing).
  2. A POSITIVE stock entry is logged in the `stock_movement_log` table (e.g., "+15kg Chicken").
  3. The `ingredients` table `quantity` field is INCREASED by the purchased amount.

### "Add Unplanned Purchase" Button
**What it's for:** Sometimes the Purchasing Officer buys something that was NOT on the daily market list.
- **Use case 1:** Eggs were too expensive at the market (price tripled), so they bought Pork instead as a cheaper substitute. The Pork was not on the list, so they use "Add Unplanned Purchase" to log it.
- **Use case 2:** They found fresh Mangoes on sale and bought them for tomorrow's dessert even though it wasn't planned.
- **What happens:** This button does the EXACT same 3 database actions as "Log Purchase" (insert purchase_history, log positive stock_movement, increase ingredients.quantity). The only difference is the UI — it lets them pick any ingredient, not just ones on today's list.

### `PurchaseHistory.vue` — Receipt Archive
**What it is:** A searchable, filterable, chronological list of ALL past purchases ever made. Each row shows: Date, Ingredient Name, Quantity, Unit, Unit Price, Total Cost, Supplier, Receipt #. This is used for DOH compliance auditing and cost analysis. This data comes from the `purchase_history` database table and must NOT be static/hardcoded.

### `StockMovementLog.vue` — The Inventory Ledger (Like a Bank Statement)
**What it is:** A chronological history log of EVERY stock change that has ever happened to any ingredient. Think of it exactly like a bank account statement:
- **Positive entries (+):** "Jul 22 — Purchased 15kg Chicken (+15kg)" → from Log Purchase / Unplanned Purchase
- **Negative entries (-):** "Jul 22 — Used 12.75kg Chicken for Tinola ×85 servings (-12.75kg)" → from Kitchen Backflush when cooking is marked complete
- **Running balance:** Each row should show the running total, just like a bank statement.
- **This must NOT be static data.** It must be populated automatically from the `stock_movement_log` database table every time a purchase is logged (positive) or a dish is cooked (negative backflush).

---

## 👨‍🍳 PORTAL 5: KITCHEN STAFF

### `ProductionSchedule.vue` — The Kitchen Prep Sheet (Today's Cooking Orders)

**How do foods appear on this screen?**
1. The Dietitian dispatches meals from `DailyProduction.vue`.
2. The Kitchen Staff opens their portal and sees today's **Prep Sheet** — a list of dishes that need to be cooked today.
3. Each dish shows:
   - Dish name (e.g., "Chicken Tinola")
   - Total number of servings to cook (e.g., "95 servings" = 85 Normal + 10 Diabetic)
   - The complete raw ingredient list with MULTIPLIED quantities:
     - Chicken: 150g × 95 = 14,250g (14.25 kg)
     - Sayote: 100g × 95 = 9,500g (9.5 kg)
     - Ginger: 10g × 95 = 950g
   - Status: "Pending" → "In Progress" → "Cooked"

**"Mark as Cooked" Button — Triggers Inventory Backflush:**
When the Kitchen Staff finishes cooking a dish and clicks this button, the backend performs the **Inventory Backflush**:
1. Reads the dish's BOM (ingredient list from Dish Menu).
2. Multiplies each ingredient quantity by the number of servings cooked.
3. **SUBTRACTS** those amounts from the `ingredients` table `quantity` field.
4. Logs a NEGATIVE entry in `stock_movement_log`: "Used 14.25kg Chicken for Tinola ×95 servings".
5. If any ingredient stock falls below a critical threshold after deduction, a notification is triggered to the Purchasing Officer: "⚠️ LOW STOCK: Chicken is now at 2kg remaining."
6. The dish status changes to "Cooked" / "Ready for Serving" which the Food Server can now see.

### `ProductionHistory.vue` — Kitchen Historical Log
**What it is:** A historical archive of everything the kitchen has ever cooked. Each row shows:
- Date, Dish Name, Number of Servings, Total Ingredients Used, Status, Who marked it as cooked (staff name).
- This is for accountability, auditing, and tracking kitchen performance over time.
- This data comes from completed production records in the database — it must NOT be hardcoded.

---

## 🚚 PORTAL 6: FOOD SERVER

### `MobileDistribution.vue` / `DistributionList.vue` — Meal Delivery Tracker

**How it works with QR codes (10 Wards):**
1. The hospital has exactly **10 Wards**. Each ward has a permanent QR code sticker at its entrance.
2. The QR code contains simple text identifying the ward: "WARD-01", "WARD-02", ..., "WARD-10".
3. The Food Server (there are 5 Food Servers total) opens the app on their phone browser.
4. They scan the QR code at a ward entrance.
5. The system reads "WARD-03" from the QR code and queries: "Show me all patients currently admitted in Ward 3 AND their assigned meals for today."
6. The Food Server sees a checklist showing each patient's name, room, bed, and their Breakfast/Lunch/Dinner dishes.
7. After physically delivering each tray to the patient, the Food Server taps **"Mark as Served"** next to that patient.
8. This updates the meal_assignment status to "Served" in the database.

**Data flow after marking as served:**
- The served status flows to the Dietitian's **`MealServiceHistory.vue`** so they can track delivery completion.
- The served count feeds into **`DohReport.vue`** for compliance reporting (how many patients were successfully fed today).

---

## 🔔 NOTIFICATIONS (Cross-Portal — Must Be Automatic)

Notifications should fire automatically when these specific events happen. Each notification targets a specific role:

| Event | Who Gets Notified | Message |
|---|---|---|
| Doctor writes a new diet prescription | Dietitian | "New diet prescription for [Patient Name]: [Diet Type]" |
| Dietitian dispatches daily production | Purchasing Officer | "New Daily Market List available for [Date]" |
| Dietitian dispatches daily production | Kitchen Staff | "New production schedule available for [Date]" |
| Purchasing Officer logs a purchase | Kitchen Staff | "Ingredients received: [Item Name] — Ready to cook" |
| Kitchen Staff marks dish as cooked | Food Server | "Meals ready for delivery: [Dish Name] ×[Servings]" |
| Ingredient stock falls below critical level | Purchasing Officer + Dietitian | "⚠️ LOW STOCK: [Ingredient] is at [Qty] remaining" |
| Patient is discharged | Dietitian | "Patient [Name] discharged — future meals cancelled" |

---

## 📊 DOH COMPLIANCE REPORT (`DohReport.vue`)

**What it generates:** A summary report for Department of Health auditing. The report should compile:
1. **Daily Meal Summary:** Total patients fed, total meals served, meals per diet type.
2. **Cost Summary:** Total food cost for the period, average cost per patient per day, budget adherence (% of patients within ₱150 limit).
3. **Purchase Summary:** Total purchases, itemized ingredient costs, supplier breakdown.
4. **Inventory Summary:** Current stock levels, stock movement history, wastage (if any).

**Export formats:** PDF (for printing) and CSV/Excel (for spreadsheet analysis).

> **NOTE:** We do not have the final DOH report template from the client hospital yet. The current implementation is a placeholder. The real template will be connected on Friday. The backend just needs to be able to query and aggregate the data — the exact visual layout will be updated later.

---

## ⚠️ CRITICAL MATH FORMULAS (Reference for Backend)

### Calorie Auto-Calculation (Per Dish)
```
dish_calories = SUM(ingredient_quantity_grams × ingredient_calories_per_gram) for all ingredients in the dish
```

### Daily Patient Budget Check
```
daily_cost = breakfast_cost + lunch_cost + dinner_cost
IF daily_cost > 150.00 THEN trigger budget warning
```

### Ingredient Aggregation for Market List
```
FOR each dispatched diet_group:
  FOR each dish in the group's meal plan:
    FOR each ingredient in the dish's BOM:
      total_needed[ingredient] += ingredient_qty_per_serving × number_of_patients_in_group
```

### Inventory Backflush (When Kitchen Marks as Cooked)
```
FOR each ingredient in the cooked dish's BOM:
  deduction = ingredient_qty_per_serving × total_servings_cooked
  ingredients_table[ingredient].quantity -= deduction
  INSERT INTO stock_movement_log (type: 'DEDUCTION', quantity: -deduction, reason: 'Backflush')
  IF ingredients_table[ingredient].quantity < critical_threshold:
    CREATE notification for Purchasing Officer: "LOW STOCK WARNING"
```

---

**FINAL INSTRUCTION TO AI:** Before implementing anything above, scan the current codebase to check which of these features already exist and are working. Only implement features that are missing or broken. Do NOT rebuild working features. Work in small, testable chunks — implement one screen's logic at a time, test it, then move to the next.
