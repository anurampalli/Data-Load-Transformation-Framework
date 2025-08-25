# Data-Load-Transformation-Framework
Built data transformation framework for seamless HR-to-ServiceNow integration.
---

## Project Overview

This project implements a robust framework for importing, transforming, and validating HR employee data into ServiceNow (`sys_user` table) using Import Sets, Transform Maps, and Scheduled Jobs. It ensures data integrity, avoids duplicates, and automates nightly imports.

---

## Modules & Components

### 1. Import Sets
- **Purpose:** Load HR employee records from CSV/XML files into ServiceNow staging tables.
- **Sample Data Sources:**
  - `users_for_Importset.csv`
  - `user_forScheduledJob.csv`
  - `users_for_TestingDuplicates.csv`

### 2. Transform Maps
- **Purpose:** Map data from import tables to target tables (`sys_user`) with normalization and enrichment.
- **Import Set Tables & Transform Maps:**
  - `u_is_scheduled_user` → Transform Map: `TM HR User Dump`
  
- **Transform Script Implemented:**
  
  **onStart Script**
  ```javascript
  (function runTransformScript(source, map, log, target) {
      // Build dictionary of Manager Emails → Sys_id
      this.managerMap = {};
      var grManagerUser = new GlideRecord("sys_user");
      grManagerUser.query();
      while (grManagerUser.next()) {
          managerMap[grManagerUser.email.toString().toLowerCase()] = grManagerUser.sys_id.toString();
      }
  })(source, map, log, target);


 **onBefore Script**
   ```javascript
  (function runTransformScript(source, map, log, target) {
    // Split full name into first and last names
    if (source.u_full_name) {
        var parts = source.u_full_name.trim().split(/\s+/);
        target.first_name = parts[0];
        if (parts.length > 1) {
            target.last_name = parts.slice(1).join(" ");
        }
    }

    // Normalize email
    if (source.u_email) {
        target.email = source.u_email.toString().toLowerCase();
    }

    // Map manager
    var managerMap = this.managerMap;
    gs.info('manager map: ' + managerMap);
    if (source.u_manager_email && managerMap) {
        var managerEmail = source.u_manager_email.toString().toLowerCase();
        var managerSysId = managerMap[managerEmail];
        gs.info('TM HR User Dump managerSysId: ' + managerSysId);

        if (managerSysId) {
            target.manager = managerSysId;
        } else {
            log.warn("Manager email not found: " + managerEmail + " for user " + source.u_full_name);
            target.manager = ""; // optional: blank manager instead of error
        }
    }
})(source, map, log, target);
```
### 3. Scheduled Jobs

* **Purpose:** Automate nightly imports and log results.
* **Status:**

  * Nightly job works for file attachments.
  * FTP import not functional; REST integration may be implemented later.

---

## Data Integrity & Validation

* Business Rule to prevent duplicate users replaced with **coalesce condition** in Transform Map.
* Normalizations applied:

  * Lowercase email
  * Trim whitespace
  * Split full name into first and last name
  * Map manager using email-to-`sys_id` dictionary

---

## Deliverables

* Fully automated HR data import with integrity checks.
* Sample CSV dataset.
* Transform Map XML for `TM HR User Dump`.

---

## Notes

* Framework is extendable to support REST-based imports.
* Duplicate prevention handled via coalesce; can be adjusted for custom business rules.
* Logging included for manager mapping and import issues.

```
