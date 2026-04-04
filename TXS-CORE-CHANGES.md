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
| 22.0.2           | 1                 | TXS-CORE-001  | Display missing unit cost in Manufacturing Order consumed items table |
| 22.0.2           | 3                 | TXS-CORE-002  | Product list performance - remove heavy GROUP BY and supplier price JOIN |
| 22.0.2           | 1                 | TXS-CORE-003  | Top menu Products link points to list page instead of index page |
| 22.0.2           | 1                 | TXS-CORE-004  | Fix missing space before INNER JOIN in productlot SQL query |
| 22.0.2           | 1                 | TXS-CORE-005  | Remove stale `fk_user_done` column reference in Future Actions box |
| 22.0.2           | 11                | TXS-CORE-006  | Move INNER JOIN `societe_commerciaux` before WHERE in 11 product stats methods |
| 22.0.2           | 1                 | TXS-CORE-007  | Add duplicate-link check in `add_object_linked()` |
| 22.0.2           | 2                 | TXS-CORE-008  | Fix broken "Billed" filter on supplier order list (Dolibarr #35434) |


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
### [2026-03-23] TXS-CORE-002 - Product List Performance Optimization
**Developer:** TXS Corp
**Dolibarr Version:** 22.0.0  
**Change Type:** Enhancement / Performance

**Files Affected:**
- `htdocs/product/list.php`

**Description:**
Commented out the heavy `GROUP BY` clause and the `LEFT JOIN` to `product_fournisseur_price`, and replaced the `MIN(pfp.unitprice)` sub-query with an empty string literal. These clauses caused significant performance degradation on the product list page when the product catalogue is large.

**Reason / Business Case:**
> The product list page was loading slowly due to an expensive `LEFT JOIN` on `product_fournisseur_price` and an associated `GROUP BY` that touched dozens of columns.  
> Removing these clauses dramatically improves page load time while the best-purchase-price column remains present (displayed as empty).

##### **Code Changes**

**Change 1 - Replace supplier price sub-query**

**Before Code (Upstream):**
*File:* `htdocs/product/list.php`  
*Line:* 497
```php
$sql .= ' MIN(pfp.unitprice) as bestpurchaseprice';
```

**After Code (TXS Customization):**
*File:* `htdocs/product/list.php`  
*Line:* 497
```php
// --- Begin Customization --- TXS Corp: Product List Performance
//$sql .= ' MIN(pfp.unitprice) as bestpurchaseprice';
$sql .= '"" as bestpurchaseprice';
// --- End Customization ---
```

---

**Change 2 - Remove supplier price LEFT JOIN**

**Before Code (Upstream):**
*File:* `htdocs/product/list.php`  
*Line:* 530
```php
$linktopfp = " LEFT JOIN ".MAIN_DB_PREFIX."product_fournisseur_price as pfp ON p.rowid = pfp.fk_product";
$sql .= $linktopfp;
```

**After Code (TXS Customization):**
*File:* `htdocs/product/list.php`  
*Line:* 530
```php
// --- Begin Customization --- TXS Corp: Product List Performance
//$linktopfp = " LEFT JOIN ".MAIN_DB_PREFIX."product_fournisseur_price as pfp ON p.rowid = pfp.fk_product";
$sql .= $linktopfp;
// --- End Customization ---
```

---

**Change 3 - Remove GROUP BY clause**

**Before Code (Upstream):**
*File:* `htdocs/product/list.php`  
*Lines:* 689-721
```php
$sql .= " GROUP BY p.rowid, p.ref, p.ref_ext, p.description, p.label, p.barcode, p.price, p.tva_tx, p.price_ttc, p.price_base_type,";
$sql .= " p.fk_product_type, p.duration, p.finished, p.tosell, p.tobuy, p.seuil_stock_alerte, p.desiredstock,";
$sql .= ' p.datec, p.tms, p.entity, p.tobatch, p.pmp, p.cost_price, p.stock,';
if (!getDolGlobalString('MAIN_PRODUCT_PERENTITY_SHARED')) {
	$sql .= " p.accountancy_code_sell, p.accountancy_code_sell_intra, p.accountancy_code_sell_export, p.accountancy_code_buy, p.accountancy_code_buy_intra, p.accountancy_code_buy_export,";
} else {
	$sql .= " ppe.accountancy_code_sell, ppe.accountancy_code_sell_intra, ppe.accountancy_code_sell_export, ppe.accountancy_code_buy, ppe.accountancy_code_buy_intra, ppe.accountancy_code_buy_export,";
}
$sql .= ' p.weight, p.weight_units, p.length, p.length_units, p.width, p.width_units, p.height, p.height_units, p.surface, p.surface_units, p.volume, p.volume_units,';
$sql .= ' p.fk_country, p.fk_state, p.stockable_product,';
$sql .= ' p.import_key';
if (getDolGlobalString('PRODUCT_USE_UNITS')) {
	$sql .= ', p.fk_unit, cu.label';
}
if (isModEnabled('workstation')) {
	$sql .= ', p.fk_default_workstation, ws.status, ws.ref';
}
if (isModEnabled('variants')) {
	$sql .= ', pac.rowid';
	$sql .= ', pac.fk_product_parent';
}
// Add fields from extrafields
if (!empty($extrafields->attributes[$object->table_element]['label'])) {
	foreach ($extrafields->attributes[$object->table_element]['label'] as $key => $val) {
		$sql .= ($extrafields->attributes[$object->table_element]['type'][$key] != 'separate' ? ", ef.".$key : '');
	}
}
// Add groupby from hooks
$parameters = array();
$reshook = $hookmanager->executeHooks('printFieldListGroupBy', $parameters, $object, $action); // Note that $action and $object may have been modified by hook
$sql .= $hookmanager->resPrint;
```

**After Code (TXS Customization):**
*File:* `htdocs/product/list.php`  
*Lines:* 689-721
```php
// --- Begin Customization --- TXS Corp: Product List Performance
/*$sql .= " GROUP BY p.rowid, p.ref, p.ref_ext, p.description, p.label, p.barcode, p.price, p.tva_tx, p.price_ttc, p.price_base_type,";
$sql .= " p.fk_product_type, p.duration, p.finished, p.tosell, p.tobuy, p.seuil_stock_alerte, p.desiredstock,";
$sql .= ' p.datec, p.tms, p.entity, p.tobatch, p.pmp, p.cost_price, p.stock,';
if (!getDolGlobalString('MAIN_PRODUCT_PERENTITY_SHARED')) {
	$sql .= " p.accountancy_code_sell, p.accountancy_code_sell_intra, p.accountancy_code_sell_export, p.accountancy_code_buy, p.accountancy_code_buy_intra, p.accountancy_code_buy_export,";
} else {
	$sql .= " ppe.accountancy_code_sell, ppe.accountancy_code_sell_intra, ppe.accountancy_code_sell_export, ppe.accountancy_code_buy, ppe.accountancy_code_buy_intra, ppe.accountancy_code_buy_export,";
}
$sql .= ' p.weight, p.weight_units, p.length, p.length_units, p.width, p.width_units, p.height, p.height_units, p.surface, p.surface_units, p.volume, p.volume_units,';
$sql .= ' p.fk_country, p.fk_state, p.stockable_product,';
$sql .= ' p.import_key';
if (getDolGlobalString('PRODUCT_USE_UNITS')) {
	$sql .= ', p.fk_unit, cu.label';
}
if (isModEnabled('workstation')) {
	$sql .= ', p.fk_default_workstation, ws.status, ws.ref';
}
if (isModEnabled('variants')) {
	$sql .= ', pac.rowid';
	$sql .= ', pac.fk_product_parent';
}
// Add fields from extrafields
if (!empty($extrafields->attributes[$object->table_element]['label'])) {
	foreach ($extrafields->attributes[$object->table_element]['label'] as $key => $val) {
		$sql .= ($extrafields->attributes[$object->table_element]['type'][$key] != 'separate' ? ", ef.".$key : '');
	}
}
// Add groupby from hooks
$parameters = array();
$reshook = $hookmanager->executeHooks('printFieldListGroupBy', $parameters, $object, $action); // Note that $action and $object may have been modified by hook
$sql .= $hookmanager->resPrint;*/
// --- End Customization ---
```

**Upgrade Notes:**
- The `bestpurchaseprice` column will display as empty. If supplier pricing visibility on the list page is needed in the future, revert Change 1 and Change 2.
- When upgrading Dolibarr, check whether `htdocs/product/list.php` upstream changes affect the `GROUP BY`, the `LEFT JOIN` on `product_fournisseur_price`, or the `bestpurchaseprice` select.
- If upstream optimises these queries in a future release, these customisations may no longer be necessary.

---


---
### [2026-03-23] TXS-CORE-003 - Top Menu Products Link to List Page
**Developer:** TXS Corp
**Dolibarr Version:** 22.0.x  
**Change Type:** Enhancement

**Files Affected:**
- `htdocs/core/menus/standard/eldy.lib.php`

**Description:**
Changed the top-menu "Products" link from the product index page (`/product/index.php`) to the product list page (`/product/list.php`) so users land directly on the searchable product list.

**Reason / Business Case:**
> The default index page provides a summary/dashboard that is less useful for day-to-day operations.  
> Pointing the top menu entry straight to the list page reduces clicks and improves workflow efficiency.

##### **Code Changes**

**Before Code (Upstream):**
*File:* `htdocs/core/menus/standard/eldy.lib.php`  
*Line:* 166
```php
'link' => '/product/index.php?mainmenu=products&amp;leftmenu=',
```

**After Code (TXS Customization):**
*File:* `htdocs/core/menus/standard/eldy.lib.php`  
*Lines:* 166-168
```php
// --- Begin Customization --- TXS Corp: Changed top menu entry to point to product list page	
'link' => '/product/list.php?mainmenu=products&amp;leftmenu=',
// --- End Customization ---
```

**Upgrade Notes:**
- If upstream changes the Products menu entry structure in `eldy.lib.php`, re-apply this single-line link change.
- No functional risk — only the landing page URL differs.

---


---
### [2026-03-23] TXS-CORE-004 - Fix Missing Space Before INNER JOIN in Product Lot Query
**Developer:** TXS Corp
**Dolibarr Version:** 22.0.x  
**Change Type:** Bugfix

**Files Affected:**
- `htdocs/product/stock/class/productlot.class.php`

**Description:**
Added a missing leading space before `INNER JOIN` in the SQL query that joins `societe_commerciaux` for sales-rep-restricted users. Without the space the previous SQL fragment and `INNER JOIN` would concatenate without a separator, producing an invalid query.

**Reason / Business Case:**
> The missing space caused a malformed SQL statement for users without the `societe->client->voir` permission, potentially breaking product lot lookups tied to manufacturing orders.

##### **Code Changes**

**Before Code (Upstream):**
*File:* `htdocs/product/stock/class/productlot.class.php`  
*Line:* 1087
```php
$sql .= "INNER JOIN ".$this->db->prefix()."societe_commerciaux as sc ON sc.fk_soc=c.fk_soc AND sc.fk_user = ".((int) $user->id);
```

**After Code (TXS Customization):**
*File:* `htdocs/product/stock/class/productlot.class.php`  
*Line:* 1087
```php
$sql .= " INNER JOIN ".$this->db->prefix()."societe_commerciaux as sc ON sc.fk_soc=c.fk_soc AND sc.fk_user = ".((int) $user->id);
```

**Upgrade Notes:**
- Trivial one-character fix. If upstream corrects this in a future release, this change can be dropped.

---


---
### [2026-03-23] TXS-CORE-005 - Remove Stale `fk_user_done` Column Reference
**Developer:** TXS Corp
**Dolibarr Version:** 22.0.x  
**Change Type:** Bugfix

**Files Affected:**
- `htdocs/core/boxes/box_actions_future.php`

**Description:**
Removed the reference to the non-existent `fk_user_done` column in the "Future Actions" dashboard box SQL query. The column was renamed to `fk_user_action` in the Dolibarr v3.6 migration, but this file still referenced the old name, causing SQL errors for non-admin users without `agenda > allactions > read` permission.

**Reason / Business Case:**
> The "Future Actions" dashboard box produced SQL errors for restricted users because the query referenced a column (`fk_user_done`) that no longer exists in the `llx_actioncomm` table. This broke the dashboard for any user without full agenda read permissions.

##### **Code Changes**

**Before Code (Upstream):**
*File:* `htdocs/core/boxes/box_actions_future.php`  
*Line:* 110
```php
$sql .= " AND (a.fk_user_author = ".((int) $user->id)." OR a.fk_user_action = ".((int) $user->id)." OR a.fk_user_done = ".((int) $user->id).")";
```

**After Code (TXS Customization):**
*File:* `htdocs/core/boxes/box_actions_future.php`  
*Line:* 110
```php
// --- Begin Customization --- TXS Corp: Remove stale fk_user_done column reference (renamed to fk_user_action in v3.6)
$sql .= " AND (a.fk_user_author = ".((int) $user->id)." OR a.fk_user_action = ".((int) $user->id).")";
// --- End Customization ---
```

**Upgrade Notes:**
- If upstream removes this stale reference in a future release, this change can be dropped.
- No functional change — `fk_user_action` was already present in the OR clause and covers the same data.

---


---
### [2026-03-23] TXS-CORE-006 - Fix INNER JOIN After WHERE in Product Stats Methods
**Developer:** TXS Corp
**Dolibarr Version:** 22.0.x  
**Change Type:** Bugfix

**Files Affected:**
- `htdocs/product/class/product.class.php`

**Description:**
Moved the conditional `INNER JOIN societe_commerciaux` clause from **after** the `WHERE` keyword to **before** it in 11 `load_stats_*` methods. The original code produced invalid SQL (`... WHERE x = y INNER JOIN ...`) for internal users without the `societe > client > voir` permission, breaking all product statistics pages for sales-rep-restricted users.

**Reason / Business Case:**
> Internal users restricted to their own companies (without `societe > client > voir`) could not view product statistics. The SQL was syntactically invalid because `INNER JOIN` appeared after `WHERE` conditions had already been emitted, causing database errors on the product card Statistics tab.

##### **Code Changes**

**Affected methods (11 total):**
1. `load_stats_propale()` — fk_soc alias: `p`
2. `load_stats_proposal_supplier()` — fk_soc alias: `p`
3. `load_stats_commande()` — fk_soc alias: `c`
4. `load_stats_commande_fournisseur()` — fk_soc alias: `c`
5. `load_stats_sending()` — fk_soc alias: `e`
6. `load_stats_reception()` — fk_soc alias: `cf`
7. `load_stats_inproduction()` — fk_soc alias: `m`
8. `load_stats_contrat()` — fk_soc alias: `c`
9. `load_stats_facture()` — fk_soc alias: `f`
10. `load_stats_facturerec()` — fk_soc alias: `f`
11. `load_stats_facture_fournisseur()` — fk_soc alias: `f`

**Before Code pattern (Upstream) — example from `load_stats_commande()`:**
*File:* `htdocs/product/class/product.class.php`
```php
$sql .= ", ".$this->db->prefix()."societe as s";
$sql .= " WHERE c.rowid = cd.fk_commande";
$sql .= " AND c.fk_soc = s.rowid";
$sql .= " AND c.entity IN (".getEntity('commande').")";
$sql .= " AND cd.fk_product = ".((int) $this->id);
if (empty($user->fk_soc) && !$user->hasRight('societe', 'client', 'voir') && !$forVirtualStock) {
    $sql .= " INNER JOIN ".$this->db->prefix()."societe_commerciaux as sc ON sc.fk_soc = c.fk_soc AND sc.fk_user = ".((int) $user->id);
}
```

**After Code pattern (TXS Customization):**
*File:* `htdocs/product/class/product.class.php`
```php
$sql .= ", ".$this->db->prefix()."societe as s";
// --- Begin Customization --- TXS Corp: Move INNER JOIN before WHERE clause
if (empty($user->fk_soc) && !$user->hasRight('societe', 'client', 'voir') && !$forVirtualStock) {
    $sql .= " INNER JOIN ".$this->db->prefix()."societe_commerciaux as sc ON sc.fk_soc = c.fk_soc AND sc.fk_user = ".((int) $user->id);
}
// --- End Customization ---
$sql .= " WHERE c.rowid = cd.fk_commande";
$sql .= " AND c.fk_soc = s.rowid";
$sql .= " AND c.entity IN (".getEntity('commande').")";
$sql .= " AND cd.fk_product = ".((int) $this->id);
```

**Upgrade Notes:**
- The same fix pattern was applied to all 11 methods listed above.
- `load_stats_mo()` was already correct upstream and was not modified.
- If upstream fixes this in a future release, verify all 11 methods before removing the customization markers.
- Search for `// --- Begin Customization --- TXS Corp: Move INNER JOIN before WHERE clause` to locate all patched sites.

---


---
### [2026-03-23] TXS-CORE-007 - Add Duplicate Link Check in `add_object_linked()`
**Developer:** TXS Corp
**Dolibarr Version:** 22.0.x  
**Change Type:** Bugfix

**Files Affected:**
- `htdocs/core/class/commonobject.class.php`

**Description:**
Added a `SELECT COUNT(*)` check before the `INSERT INTO llx_element_element` in `add_object_linked()` to detect and gracefully handle duplicate links. If the link already exists, the method returns `1` (success) immediately instead of attempting the INSERT, which would fail against the existing UNIQUE index and be misreported as an error.

**Reason / Business Case:**
> Repeated calls to `add_object_linked()` (from triggers, hooks, or retries) would hit the UNIQUE constraint on `llx_element_element` and return `0` (error), confusing callers into thinking the operation failed. The code-level check returns success when the link already exists, avoiding spurious error logs and rollback overhead.

##### **Code Changes**

**Before Code (Upstream):**
*File:* `htdocs/core/class/commonobject.class.php`  
*Line:* ~4292 (before INSERT)
```php
$this->db->begin();
$error = 0;

$sql = "INSERT INTO " . $this->db->prefix() . "element_element (";
```

**After Code (TXS Customization):**
*File:* `htdocs/core/class/commonobject.class.php`  
*Line:* ~4292
```php
// --- Begin Customization --- TXS Corp: Check for duplicate link before INSERT
$sqlCheck = "SELECT COUNT(*) as cnt FROM ".$this->db->prefix()."element_element";
$sqlCheck .= " WHERE fk_source = ".((int) $origin_id);
$sqlCheck .= " AND sourcetype = '".$this->db->escape($origin)."'";
$sqlCheck .= " AND fk_target = ".((int) $this->id);
$sqlCheck .= " AND targettype = '".$this->db->escape($targettype)."'";
$resCheck = $this->db->query($sqlCheck);
if ($resCheck) {
    $objCheck = $this->db->fetch_object($resCheck);
    if ($objCheck->cnt > 0) {
        dol_syslog(get_class($this)."::add_object_linked link already exists", LOG_DEBUG);
        return 1;
    }
}
// --- End Customization ---

$this->db->begin();
$error = 0;

$sql = "INSERT INTO " . $this->db->prefix() . "element_element (";
```

**Upgrade Notes:**
- The UNIQUE index `idx_element_element_idx1` on `(fk_source, sourcetype, fk_target, targettype)` already exists in `llx_element_element.key.sql` — no schema changes needed.
- If upstream adds its own duplicate-check logic, this customization can be removed.
- The SELECT-before-INSERT pattern has a theoretical TOCTOU race condition, but the existing UNIQUE index acts as a safety net for the rare case.

---



---


---
### [2026-04-03] TXS-CORE-008 - Fix Broken "Billed" Filter on Supplier Order List
**Developer:** TXS Corp
**Dolibarr Version:** 22.0.x  
**Change Type:** Bugfix

**Files Affected:**
- `htdocs/fourn/commande/list.php`
- `htdocs/fourn/card.php`

**Description:**
Fixed the "Billed" (Invoiced) filter dropdown on the supplier order list page, which was broken due to `GETPOSTINT` casting the parameter to integer. This made it impossible to distinguish "parameter absent" from "No" selected — both became `int(0)`. On PHP 8, this caused `AND cf.billed = 0` to be injected on every page load, silently defaulting to showing only unbilled orders. Also fixed the "Create Invoice for This Supplier" button on the supplier card, which sent `search_billed=0` instead of the correct parameter name `billed`.

**Reason / Business Case:**
> The supplier order list always filtered to unbilled orders on initial load (PHP 8), and the "Billed = No" dropdown appeared pre-selected. The "Create Invoice for This Supplier" button's pre-filter to unbilled orders never worked because it used the wrong parameter name. This is a confirmed upstream bug: [Dolibarr #35434](https://github.com/Dolibarr/dolibarr/issues/35434) (filed September 2025, still open with no upstream fix merged).

##### **Code Changes**

**Change 1 — Use `GETPOST` instead of `GETPOSTINT` for billed parameter**

**Before Code (Upstream):**
*File:* `htdocs/fourn/commande/list.php`  
*Line:* 137
```php
$billed = GETPOSTINT('billed');
```

**After Code (TXS Customization):**
*File:* `htdocs/fourn/commande/list.php`  
*Line:* 137
```php
// --- Begin Customization --- TXS Corp: Fix Billed filter – use GETPOST to preserve '' vs '0' distinction (Dolibarr #35434)
$billed = GETPOST('billed', 'int');
// --- End Customization ---
```

---

**Change 2 — Fix parameter name in "Create Invoice for This Supplier" button**

**Before Code (Upstream):**
*File:* `htdocs/fourn/card.php`  
*Line:* 1088
```php
print dolGetButtonAction('', $langs->trans('CreateInvoiceForThisSupplier'), 'default', DOL_URL_ROOT.'/fourn/commande/list.php?socid='.$object->id.'&amp;search_billed=0&amp;autoselectall=1', '');
```

**After Code (TXS Customization):**
*File:* `htdocs/fourn/card.php`  
*Line:* 1088
```php
// --- Begin Customization --- TXS Corp: Fix parameter name from search_billed to billed (Dolibarr #35434)
print dolGetButtonAction('', $langs->trans('CreateInvoiceForThisSupplier'), 'default', DOL_URL_ROOT.'/fourn/commande/list.php?socid='.$object->id.'&amp;billed=0&amp;autoselectall=1', '');
// --- End Customization ---
```

**Upgrade Notes:**
- If upstream merges a fix for [#35434](https://github.com/Dolibarr/dolibarr/issues/35434), compare their approach before carrying this forward.
- Search for `// --- Begin Customization --- TXS Corp: Fix Billed filter` and `// --- Begin Customization --- TXS Corp: Fix parameter name from search_billed` to locate the patched sites.
- No variable renames, SQL logic changes, or `selectyesno` form changes were made — the fix is minimal and self-contained.

---



## Notes

- **Do NOT edit Dolibarr core files without logging the change here.**
- Always create or update a `htdocs/custom/` module when possible instead of modifying core.
- This file is protected by `.gitattributes`.
