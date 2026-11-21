Part D — Business Rules Beyond Standard Relational Constraints

#

SRS Section

Business Rule

Why Constraints Are Not Enough

How to Enforce It

1

Section 7.7

Blacklisted Customer Booking Block: A customer with IsBlacklisted = 1 cannot create a new reservation or initiate a walk-in rental.

A CHECK constraint on RESERVATION or RENTAL cannot query another table (CUSTOMER.IsBlacklisted) to inspect the customer's status at insertion time.

Database Trigger (BEFORE INSERT on RESERVATION and RENTAL) or Stored Procedure that checks CUSTOMER.IsBlacklisted and raises an error if true, reinforced by application-layer validation.

2

Section 7.9

Promo Code Usage & Validity Enforcement: A promo code can only be applied if the reservation date falls within [ValidFrom, ValidTo] and the total count of reservations using that code has not exceeded MaxUses.

SQL constraints cannot aggregate row counts across RESERVATION (COUNT(*) → MaxUses) or compare dates across two separate tables (RESERVATION.CreatedDT vs PROMO_CODE.ValidFrom/ValidTo).

Stored Procedure / Application Logic inside a database transaction (with row-level locking on PROMO_CODE) or a BEFORE INSERT Trigger on RESERVATION verifying date range and usage count.

3

Section 7.12

Corporate Rental Main Driver Ownership: For a corporate rental, MainDriverSeqNo must be provided and must belong to the exact CORPORATE_CUSTOMER making the rental; for individual rentals, it must be NULL.

A standard Foreign Key can ensure (CorpCustNo, MainDriverSeqNo) exists in AUTHORIZED_DRIVER, but cannot ensure that CorpCustNo matches RENTAL.CustomerNo or check CUSTOMER.CustomerType across tables.

BEFORE INSERT/UPDATE Trigger on RENTAL that queries CUSTOMER.CustomerType and verifies CorpCustNo = CustomerNo for corporate clients (and ensures both driver fields are NULL for individual clients).

4

Section 7.1 & 7.12

Role & Disjoint Subtype Enforcement: Only employees registered in RENTAL_AGENT can be assigned as HandoverAgentID / ReceiveAgentID on a rental; only MECHANIC employees can be assigned to JOB_MECHANIC, and an employee cannot exist in both subtype tables.

Foreign keys point to EMPLOYEE.EmpID (or subtype PKs), but standard declarative constraints cannot enforce disjointness across two separate subtype tables (RENTAL_AGENT vs MECHANIC).

Triggers on RENTAL_AGENT and MECHANIC checking that the inserted EmpID does not already exist in the sibling subtype table, combined with FKs pointing directly to subtype PKs.

5

Section 7.3 & 7.6

Vehicle Transfer Branch Continuity: When a VEHICLE_TRANSFER is recorded, FromBranchCode must match the vehicle's current physical location, and the transfer's OdometerReading must be >= VEHICLE.CurrentOdometer.

Cross-table state validation and sequential history checks (comparing the new transfer against the vehicle's current branch or current odometer) cannot be expressed in a single-row CHECK constraint.

Stored Procedure (sp_TransferVehicle) or AFTER INSERT Trigger on VEHICLE_TRANSFER that validates the vehicle's current branch and odometer, logs the transfer, and updates VEHICLE.CurrentBranchCode and VEHICLE.CurrentOdometer.

