1. Questions for the Client / BA Team

Section 7.1 (Branches & Staff): When a regional director temporarily takes over a branch after a manager resigns, is the regional director stored as an Employee in the system with a temporary TakeoverDate, or should we keep a historical log of branch managers?

Section 7.1 (Other Staff): Are accountants, drivers, and cleaners modeled as separate subtypes in the database, or do they remain solely in the superclass EMPLOYEE since they have no specialized attributes?

Section 7.3 (Pricing): How is the rental base price calculated if a category's daily rate changes mid-rental? Does the rate at pickup apply for the entire rental duration, or is it calculated day-by-day?

Section 7.10 (Loyalty Program): What is the exact conversion formula between EGP spent on a rental and loyalty points earned/redeemed?

Section 7.16 (Payments & Corporate Billing): Should individual corporate rentals be explicitly linked to a specific monthly CORPORATE_INVOICE line-by-line, or is an invoice simply billed against the corporate customer for a given billing month?

2. Design Assumptions

Section 7.1 (Airport 24/7 Branches): Added a boolean flag Is24Hours on BRANCH; when Is24Hours = 1, OpenTime and CloseTime are allowed to be NULL.

Section 7.1 (Staff ISA Hierarchy): Modeled RENTAL_AGENT and MECHANIC as a Disjoint, Partial specialization of EMPLOYEE because an employee cannot be both, while other staff (drivers, accountants, cleaners) exist only in EMPLOYEE.

Section 7.2 (Training Retakes): Added AttemptNo as part of the composite key of EMPLOYEE_TRAINING so an employee can enroll in the same course multiple times after failing without violating primary key uniqueness.

Section 7.3 (Vehicle Status Audit): Added StatusUpdatedBy (FK to EMPLOYEE) and StatusUpdatedAt directly on VEHICLE to satisfy the audit requirement for vehicle status changes.

Section 7.7 & Section 8 (Corporate Driver vs Individual Customer): As noted by Ms. Rana Adel, an authorized corporate driver may also rent as an individual customer. They are kept as separate entities (AUTHORIZED_DRIVER weak to CORPORATE_CUSTOMER, and INDIVIDUAL_CUSTOMER subtype of CUSTOMER) to avoid coupling corporate employment lifecycle with personal accounts.

Section 7.10 (Loyalty Tier & Points): PointsBalance and LoyaltyTier are modeled as derived attributes calculated from LOYALTY_TRANSACTION records to prevent manual tampering or data loss.

Section 7.14 (Damage Photos): Modeled PhotoLinks as a multivalued attribute on DAMAGE_REPORT (mapped to a separate DAMAGE_PHOTO table) because multiple photo URLs are stored per report.

3. Contradictions Identified in the SRS

Section 7.1 vs Section 10 (Scope vs Employee Attributes): Section 1.3 and Section 10 state that "HR payroll calculations are out of scope for this phase", whereas Section 7.1 explicitly requires storing Salary for every employee and CommissionRate for rental agents (and NFR-3 specifies hiding salaries from agents). Resolution: Stored Salary and CommissionRate as static attributes in EMPLOYEE / RENTAL_AGENT without building payroll calculation tables.

Section 7.3 vs Section 7.3 COO Note (Category Pricing vs Vehicle-Specific Pricing): Section 7.3 states that "Pricing is set per category as a daily rate", immediately followed by the COO's note that certain luxury vehicles (e.g., Mercedes S-Class) have their own individual price. Resolution: Added an optional CustomDailyRate (NULL) on VEHICLE that overrides CATEGORY_RATE.DailyRate when present.

Section 7.16 vs Section 8 (Payments per Rental vs Corporate Monthly Invoicing): Section 7.16 initially describes payments as belonging to a rental, while the Finance note in Section 7.16 states corporate clients pay via a single end-of-month invoice. Resolution: Linked PAYMENT optionally to either RENTAL or CORPORATE_INVOICE (with a CHECK constraint ensuring at least one FK is populated).