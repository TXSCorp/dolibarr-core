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


## Notes

- **Do NOT edit Dolibarr core files without logging the change here.**
- Always create or update a `htdocs/custom/` module when possible instead of modifying core.
- This file is protected by `.gitattributes`.
