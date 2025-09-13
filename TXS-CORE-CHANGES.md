# TXS Core Change Log

![](https://docs.txscorp.com/img/TXS_Logo-Small.png)

> **Purpose:**  
> This document tracks all changes made by **TXS Corp** to the Dolibarr _core codebase_  
> (`htdocs/` and other non-custom directories).
> 
> **This ensures:**
> - Clear history of modifications outside of `htdocs/custom/` modules.
> - Easier upgrades when merging future Dolibarr releases.
> - Audit trail for debugging and compliance.


---

## Table of Contents

- [Change Log Format](#change-log-format)
- [Summary by Version](#summary-by-version)
- [Detailed Changes](#detailed-changes)


---

## Change Log Format

Each change entry **must include**:

| Field                      | Description                                             |
| -------------------------- | ------------------------------------------------------- |
| **Date**                   | YYYY-MM-DD of when the change was made                  |
| **TXS Change ID**          | Unique ID, e.g., `TXS-CORE-001`                         |
| **Developer**              | Who made the change                                     |
| **Dolibarr Version**       | Dolibarr core version at the time of change             |
| **Files Affected**         | List of modified core files (relative paths)            |
| **Change Type**            | Bugfix / Enhancement / Custom Feature / Temporary Patch |
| **Description**            | What was changed and why                                |
| **Reason / Business Case** | Why this modification was necessary                     |
| **Before Code**            | The original upstream Dolibarr code (snippet)           |
| **After Code**             | The new TXS-modified code (snippet)                     |
| **Upgrade Notes**          | Special handling required when upgrading Dolibarr       |


---

## Summary by Version

| Dolibarr Version | TXS Changes Count | **Change ID** | Notes                                                                 |
| ---------------- | ----------------- | ------------- | --------------------------------------------------------------------- |
| 22.0.x           | 0                 |               | Initial v22 integration                                               |
| 22.0.x           | 1                 | TXS-CORE-001  | Display missing unit cost in Manufacturing Order consumed items table |


---

## Detailed Changes


---
### [2025-09-13] TXS-CORE-001 - Display Unit Cost in Manufacturing Orders
**Developer:** Accellier
**Dolibarr Version:** 22.0.0  
**Change Type:** Bugfix / Custom Enhancement

**Files Affected:**
- `htdocs/mrp/mo_production.php`

**Description:**
Added logic to display the **unit cost** of consumed stock movements in the Manufacturing Order view.

**Reason / Business Case:**
> Users were unable to see unit costs for consumed products in manufacturing orders, leading to gaps in production cost auditing and verification.  
> This enhancement improves transparency and financial reporting accuracy.

##### **Code Changes**

**Before Code (Upstream):**
*File:* `htdocs/mrp/mo_production.php`  
*Lines:* 1792-1795
```php
// Cost price
if ($permissiontoupdatecost) {
        print '<td></td>';
}
```

**After Code (TXS Customization):**
*File:* `htdocs/mrp/mo_production.php`  
*Lines:* 1792-1799
```php
// --- Begin Customization --- TXS Corp/Accellier: Fix the Unit Price not visible in Manufacturing Order
// Cost price
if ($permissiontoupdatecost) {
    $stockmovement = new MouvementStock($db);
    $stockmovement->fetch($line2['fk_stock_movement']);
    print '<td>'.($stockmovement->price ? price2num($stockmovement->price) : 0).'</td>';
}
// --- End Customization ---
```

**Upgrade Notes:**
- Verify `$permissiontoupdatecost` logic during future Dolibarr upgrades.
- If upstream adds similar functionality, evaluate before carrying this customization forward.
- Check for possible conflicts in `mrp/mo_production.php` when merging new versions.

---



---


## Notes

- **Do NOT edit Dolibarr core files without logging the change here.**
- Always create or update a `htdocs/custom/` module when possible instead of modifying core.
- This file is protected by `.gitattributes`.
