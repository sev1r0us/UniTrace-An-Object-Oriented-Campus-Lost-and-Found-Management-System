# UniTrace

### An Object-Oriented Campus Lost and Found Management System

## About the Project

Lost belongings on campus (student IDs, phones, laptops, wallets, keys) end up at different offices, guard posts, and department desks. With no central record, students walk from place to place asking whether their item was found, recovery rates stay low, and offices fill up with unclaimed items.

**UniTrace** is a console-based Java application that centralizes this process. Admins and guards log every found item, students search the catalog and file lost reports, the system matches reports to found items automatically, and an item is only released after the claimant proves ownership.

**Who uses it**

| User | What they can do |
|---|---|
| **Owner (Student)** | Search found items (private details hidden), file lost reports, view possible matches, submit ownership proof, track claims |
| **Admin / Guard** | Log found items, review lost reports and matches, verify or reject claims, decide locked claims, release items and print receipts |

---

## Features

| Feature | Description |
|---|---|
| **Item Registration and Categorization** | Admins and guards log items with category, name, location found, date, and category-specific details |
| **Search and Cross-Matching** | Search by keyword or category. Lost reports are scored against found items by category, keywords, location, and date, with the best matches listed first |
| **Role-Based Access** | Separate Owner menu and PIN-protected Admin/Guard menu |
| **Dynamic Claim Verification** | Each item type verifies ownership its own way: student number for IDs, serial number or lock screen description for electronics, a hidden distinct feature for personal belongings |
| **Masked Search Results** | Serial numbers, student numbers, lock screen descriptions, and distinct features are shown as `********` to students so they can't be read off the screen and used to claim an item |
| **Claim Attempt Limit** | 3 failed verifications by the same student on the same item lock the claim and send it to manual admin review |
| **Status Lifecycle Manager** | `UNCLAIMED` → `PENDING_VERIFICATION` → `READY_FOR_PICKUP` → `CLAIMED`. Invalid changes are rejected, and a rejected claim returns the item to `UNCLAIMED` |
| **Release and Receipt** | The admin checks the claimant's school ID, marks the item `CLAIMED`, and prints a numbered release receipt |
| **Input Validation** | Custom exceptions and re-prompting keep bad input (wrong formats, unknown IDs, empty fields) from crashing the app |
| **Colored Console UI** | Status colors and clear menus. Set `NO_COLOR` to turn colors off |

### Item status lifecycle

```
                 correct proof / locked claim
   UNCLAIMED ───────────────────────────────▶ PENDING_VERIFICATION
       ▲                                              │
       │          claim rejected                      │ admin approves
       └──────────────────────────────────────────────┤
                                                      ▼
                         CLAIMED ◀─────────── READY_FOR_PICKUP
                                  release + receipt
```

### Match scoring

| Criterion | Points |
|---|---|
| Same category | 40 |
| Shared keywords (name and details) | up to 30 |
| Shared location words | up to 15 |
| Found shortly after the loss date | up to 10 |

Matches scoring below **55/100** are hidden to avoid weak suggestions.

---

## Sample Run

Real output of the app (colors omitted).

**Main menu**

```
=======================================
            U N I T R A C E
  Campus Lost and Found System
=======================================
  [1] Owner
  [2] Admin / Guard (PIN)
  [3] Exit
  -------------------------------------
  Enter choice: 1
  Enter owner number: 25-12345
  Welcome, Student!
```

**Owner: searching found items (private details masked)**

```
  OWNER MENU > Search Found Items
  Keyword (blank = all): laptop
  -------------------------------------
  Item ID : ITM-003
  Type    : Electronics
  Item    : Laptop (Acer)
  Found at: Library, 2nd floor
  Logged  : 2026-11-26
  Serial  : ********
  Lock scr: ********
  Status  : UNCLAIMED
  -------------------------------------
  1 item found.
  Private details are hidden. Submit a claim with proof to verify ownership.
```

**Owner: reporting a lost item and getting matches**

```
  OWNER MENU > Report Lost Item
  [1] Student ID
  [2] Electronics
  [3] Personal Belonging
  Category: 2
  Item name: Laptop
  Details (color, brand, marks): Acer laptop black
  Where did you lose it? library
  Date lost (YYYY-MM-DD, blank = today):
  Report RPT-001 filed.
  Possible matches (best first):
  -------------------------------------
  Match score: 75/100 (category, keywords, location)
  Item ID : ITM-003
  Type    : Electronics
  Item    : Laptop (Acer)
  ...
```

**Owner: submitting a claim with 3 wrong proofs (claim locked)**

```
  OWNER MENU > Submit Claim Request
  Item ID : ITM-003
  Prompt : Enter the serial number or lock screen description.
  Proof (blank to cancel): blue mountain
  [X] Proof does not match.
    Failed attempts: 1 of 3
  Proof (blank to cancel): cat photo
  [X] Proof does not match.
    Failed attempts: 2 of 3
  Proof (blank to cancel): red car
  [X] Proof does not match.
    Failed attempts: 3 of 3
  [!] Claim LOCKED. Sent to admin for manual review.
  Item status: PENDING_VERIFICATION
```

**Admin: reviewing a locked claim**

```
  ADMIN MENU > Process & Verify Claims
  CLM-001  ITM-003  Laptop (Acer)      25-12345     LOCKED
  Claim ID: CLM-001
  -------------------------------------
  Item ID : ITM-003
  Type    : Electronics
  Item    : Laptop (Acer)
  Found at: Library, 2nd floor
  Logged  : 2026-11-26
  Serial  : ACR-9X2K-7731
  Lock scr: sunset beach
  Status  : PENDING_VERIFICATION
  Proof   : red car
  Claimant: 25-12345
  -------------------------------------
  Approve this claim? (y/n): y
  Claim approved. Item is READY_FOR_PICKUP.
```

**Admin: releasing the item and printing the receipt**

```
  ADMIN MENU > Release Item / Print Receipt
  CLM-001  ITM-003  Laptop (Acer)      25-12345     READY_FOR_PICKUP
  Claim ID: CLM-001
  Student number on claimant's school ID: 25-12345
  Check claimant's school ID... verified
=======================================
            RELEASE RECEIPT
=======================================
  Receipt No : RCP-0001
  Claim ID   : CLM-001
  Item       : ITM-003 Laptop (Acer)
  Claimant   : 25-12345
  Released by: Admin/Guard
  Date       : 2026-11-27
  Status     : CLAIMED
=======================================
  Item released successfully.
```

---

## Demo Logins and Data

| Item | Value |
|---|---|
| Owner login | Any student number in the format `25-12345` |
| Admin / Guard PIN | `1234` (change `ADMIN_PIN` in `ConsoleUI.java`) |
| Sample items | `ITM-001` Student ID, `ITM-002` Wallet, `ITM-003` Acer Laptop |
| Laptop proof | Serial `ACR-9X2K-7731` or lock screen `sunset beach` |
| Wallet proof | Distinct feature: `red dragon` keychain |
| Student ID proof | Student number `25-00001` |

Set the `NO_COLOR` environment variable to disable colored output.

---

## How to Use

**As an Owner**

1. Choose **[1] Owner** and enter your student number.
2. **Search Found Items** to check whether your item was turned in.
3. If it isn't there, **Report Lost Item**. The system lists possible matches, and you can recheck them later under **My Reports and Possible Matches**.
4. When you find your item, **Submit Claim Request** and enter the proof it asks for.
5. Track the result under **My Claims**, then bring your school ID to the admin or guard.

**As an Admin / Guard**

1. Choose **[2] Admin / Guard** and enter the PIN.
2. **Log Found Item** whenever someone hands in an item.
3. Check **View Lost Reports and Matches** to see which reports line up with logged items.
4. **Process and Verify Claims** to approve or reject claims, including locked ones.
5. **Release Item / Print Receipt** after checking the claimant's school ID.

---

## Class Overview

| Class | Role |
|---|---|
| `Item` (abstract, implements `Claimable`) | Shared item data, masked and full display, status lifecycle |
| `StudentIDCard` | Verifies the student number |
| `Electronics` | Verifies the serial number or lock screen description |
| `PersonalBelonging` | Verifies a hidden distinct feature |
| `User` (abstract) | Base for `Student` and `AdminGuard` |
| `LostReport` | A missing item report used for matching |
| `ClaimRequest` | One student's claim on one item, with proof and status |
| `ItemRegistry` (implements `ClaimProcessor`) | Stores all records and runs logging, search, matching, claim checks, and release |

---

## Program Flow

```
Start
  │
  ▼
Main Menu ── 1. Owner ──────────▶ Owner Menu (Search, Report, Matches, Claim, My Claims, Log Out)
  │
  ├──────── 2. Admin / Guard ───▶ PIN check ─▶ Admin Menu (Log, View, Search, Reports,
  │                                                        Process Claims, Release, All Claims, Log Out)
  └──────── 3. Exit

Every action:  Enter information ─▶ Validate input ─▶ Process request ─▶ Display result ─▶ Back to menu
```
