# Partner Lifecycle

Press includes a comprehensive Partner portal that allows agencies to resell Frappe Cloud hosting, generate leads, and earn commissions.

## The Workflow: From Onboarding to Payout

### 1. Onboarding & Approval
*   **Trigger:** A user clicks "Become a Partner" in the Dashboard.
*   **DocType:** A `Team` document is updated with partner request details.
*   **Backend:** `press/api/partner.py:approve_partner_request` is called by administrators. Once approved, the `Team.erpnext_partner` flag is set to `1` and `partner_status` becomes `Active`.

### 2. Lead Generation & Customer Mapping
Partners can bring in customers. This is tracked via referral links or manual lead entry.
*   **DocTypes:** `Site Partner Lead` and `Team` (when a new tenant signs up under a partner).
*   **Dashboard:** Partners view their leads in `dashboard/src/components/partners/PartnerLeads.vue`.

### 3. MRR Calculation & Tiers
Press automatically categorizes partners into tiers (e.g., Bronze, Silver, Gold) based on their contribution (Monthly Recurring Revenue - MRR).
*   **Calculation:** `press/api/partner.py:get_partner_mrr` aggregates the invoice totals of all customer sites mapped to the partner.
*   **Discount/Commission Map:** Tiers dictate the commission percentage (e.g., Bronze = 15%, Gold = 25%).

### 4. Payout Generation
Commissions are distributed via Payout Orders. This process runs automatically via scheduled background jobs.

*   **Trigger:** A monthly cron job defined in `hooks.py` runs `create_marketplace_payout_orders()`.
*   **DocType:** `Payout Order` (`press/press/doctype/payout_order/payout_order.py`).
*   **Execution Flow:**
    1.  The system finds all `Invoice Item`s associated with the partner's mapped customers that haven't been accounted for.
    2.  It calculates the partner's cut based on their active Tier discount percentage.
    3.  A `Payout Order` document is created with child rows (`Payout Order Item`) detailing the exact breakdown.
*   **Dashboard View:** Partners view their Payout Orders via the "Partner Payout" tab (`dashboard/src/components/billing/mpesa/PartnerPaymentPayout.vue`).

### 5. Transferring Commissions
Partners can transfer their earned commissions as credits to their own customers.
*   **API Endpoint:** `press.api.partner.transfer_credits`.
*   **Logic:** Validates the partner has sufficient balance, applies the partner tier discount, and creates accounting ledger entries transferring credits from the Partner's `Team` balance to the Customer's `Team` balance.