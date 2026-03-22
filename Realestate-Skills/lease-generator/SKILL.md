---
name: lease-generator
version: 1.0.0
description: >
  Generate residential and commercial lease agreement drafts with state-specific
  legal clauses, landlord-tenant law caveats, and customizable terms. Use this
  skill whenever the user wants to draft a lease, create a rental agreement,
  generate an addendum, write a move-in checklist, or review lease terms.
  Trigger on phrases like "write a lease", "rental agreement", "lease template",
  "month-to-month lease", "commercial lease", "lease addendum", "pet addendum",
  "lease renewal", "eviction notice", "security deposit rules", or
  "landlord-tenant law". Always includes jurisdiction-specific legal caveats.
  Note: generated leases are drafts only — always recommend attorney review.
---

# Lease Generator Skill

Draft residential and commercial lease agreements with state-specific clauses.
Always caveat that outputs are starting templates, not legal advice.

---

## ⚠️ Legal disclaimer (include in every generated lease)

> **Important:** This lease is an AI-generated draft for informational purposes only
> and does not constitute legal advice. Landlord-tenant laws vary significantly by
> state, county, and city. Always have a licensed attorney review any lease before
> signing. This template may not reflect recent legislative changes.

---

## Input collection

Before drafting, gather:

**Essential:**
- Property address (determines jurisdiction)
- Lease type: residential / commercial / room rental / vacation
- Tenancy type: fixed-term (specify dates) / month-to-month
- Monthly rent amount
- Security deposit amount
- Landlord and tenant names

**Optional but important:**
- Pets allowed? (fee / deposit / restrictions)
- Utilities included or tenant's responsibility?
- Parking (included / additional fee)
- Late fee structure
- Notice to vacate requirements
- Smoking policy
- Maintenance responsibilities
- Early termination clause

---

## Standard residential lease template

```markdown
RESIDENTIAL LEASE AGREEMENT

This Lease Agreement ("Agreement") is entered into as of [DATE], between:

LANDLORD: [LANDLORD NAME], residing at [LANDLORD ADDRESS]
TENANT(S): [TENANT NAME(S)]

PROPERTY: [FULL PROPERTY ADDRESS] ("the Premises")

═══════════════════════════════════════════════════════════

1. LEASE TERM
   This Agreement shall commence on [START DATE] and terminate on [END DATE],
   unless renewed or terminated earlier in accordance with this Agreement.
   [OR: This is a month-to-month tenancy beginning [START DATE], terminable
   by either party with [30/60] days written notice.]

2. RENT
   Tenant agrees to pay $[AMOUNT] per month, due on the [1st/15th] day of
   each month. Rent shall be paid to [LANDLORD NAME] at [ADDRESS/METHOD].

   Late Fee: Rent not received by the [5th] day of the month is subject to
   a late fee of $[AMOUNT OR %]. [STATE CAVEAT — see below]

3. SECURITY DEPOSIT
   Tenant shall deposit $[AMOUNT] as security prior to occupancy.
   [STATE CAVEAT on maximum deposit and return timeline — see below]

4. UTILITIES & SERVICES
   □ Included in rent: [list]
   □ Tenant responsible for: [list]

5. OCCUPANTS
   The Premises shall be occupied only by the named Tenant(s) and the
   following approved occupants: [NAMES]. Guests staying longer than
   [14] consecutive days require Landlord written approval.

6. PETS
   □ No pets permitted.
   □ Pets permitted with prior written approval. Non-refundable pet fee:
     $[AMOUNT]. Monthly pet rent: $[AMOUNT/0]. Restrictions: [BREED/WEIGHT].

7. MAINTENANCE & REPAIRS
   Tenant shall keep the Premises in clean and sanitary condition. Tenant
   shall promptly notify Landlord of any damage or needed repairs.
   Landlord shall maintain the Premises in habitable condition as required
   by applicable law.

8. PROHIBITED USES
   Tenant shall not: use the Premises for any unlawful purpose; make
   alterations without written consent; sublet without written consent;
   smoke [tobacco/marijuana] on the Premises; run a business from the
   Premises without written consent.

9. ENTRY BY LANDLORD
   Landlord may enter the Premises with [24/48] hours written notice except
   in case of emergency. [STATE CAVEAT — see below]

10. TERMINATION & NOTICE
    [Fixed-term]: This lease terminates automatically on the end date.
    A renewal or holdover requires written agreement.
    [Month-to-month]: Either party may terminate with [30/60] days written
    notice. [STATE CAVEAT — see below]

11. DEFAULT
    If Tenant fails to pay rent when due, Landlord may serve a [3/5/7]-day
    Pay or Quit notice as required by [STATE] law. Tenant in default of
    other provisions shall receive a [3/10/30]-day cure notice.

12. LEAD PAINT DISCLOSURE (required for pre-1978 housing)
    □ N/A — property built after 1978.
    □ See attached Lead Paint Disclosure form (federally required).

13. ENTIRE AGREEMENT
    This Agreement constitutes the entire agreement between the parties.
    Any modifications must be in writing and signed by both parties.

14. GOVERNING LAW
    This Agreement shall be governed by the laws of [STATE].

LANDLORD SIGNATURE: _________________________ Date: ___________
TENANT SIGNATURE:   _________________________ Date: ___________
```

---

## State-specific law caveats

Include the relevant section after drafting. Flag the state based on property address.

### Security deposit rules (key variations)

| State | Max deposit | Return deadline | Interest required |
|-------|------------|-----------------|-------------------|
| California | 2 months (unfurnished) | 21 days | No |
| New York | 1 month | 14 days (itemized) | Yes (banks) |
| Texas | No limit | 30 days | No |
| Florida | No limit | 15–60 days | Yes if held > 12mo |
| Illinois | No limit | 30 days | Yes (5+ units) |
| Washington | No limit | 21 days | No |

### Late fee limits

| State | Maximum late fee |
|-------|-----------------|
| California | Lesser of $25 or 6% of rent (per court precedent) |
| New York | 5% of monthly rent (max $50) |
| Texas | 12% (4+ units) or 10% (3 or fewer) |
| Florida | Not regulated (must be reasonable) |
| DC | 5% or $10 (whichever is greater) |

### Notice to enter

| State | Minimum notice |
|-------|---------------|
| California | 24 hours |
| New York | Reasonable notice (no statute) |
| Texas | 24 hours preferred (no minimum) |
| Florida | 12 hours |
| Washington | 2 days |

### Notice to terminate month-to-month

| State | Landlord notice | Tenant notice |
|-------|----------------|---------------|
| California | 60 days (tenant > 1yr) / 30 days | 30 days |
| New York | 30 days | 30 days |
| Texas | 30 days | 30 days |
| Florida | 15 days | 15 days |
| Illinois | 30 days | 30 days |

---

## Addenda templates

### Pet addendum

```markdown
PET ADDENDUM to Lease Agreement dated [DATE]
Property: [ADDRESS]

Tenant is approved to keep the following pet(s):
Name: [PET NAME]  Type/Breed: [BREED]  Weight: [LBS]

Pet deposit (refundable): $[AMOUNT]
Monthly pet rent: $[AMOUNT]
Non-refundable pet fee: $[AMOUNT]

Tenant agrees to: carry renter's insurance covering pet liability; clean
up after pet immediately; not allow pet in [common areas]; repair all
pet-related damage; remove pet if it causes nuisance complaints.
```

### Move-in / move-out checklist

Generate room-by-room inspection checklist with condition ratings (Excellent / Good / Fair / Poor / Damaged) for:
- Living room, dining room, kitchen, each bedroom, each bathroom
- Note: walls, floors, ceilings, windows, doors, fixtures, appliances

Both parties sign and date at move-in; repeat at move-out to document deductions.

---

## Commercial lease — key differences

Commercial leases are far less regulated than residential. Key clauses to include:

- **Lease type**: Gross / Net / NNN / Modified Gross — explain to user
- **Use clause**: Specific permitted use (overly broad = risk to both parties)
- **CAM charges**: Common Area Maintenance — detail what's included, cap increases
- **Personal guarantee**: Common for LLCs/corps; landlord often requires
- **Assignment/subletting**: More negotiable in commercial
- **Build-out allowance (TI)**: Tenant improvement allowance — specify $ and scope
- **Renewal options**: Rate formula for option periods (CPI, fixed %, or market)
- **Exclusivity clause**: Prevents landlord from leasing nearby to a competitor

---

## Document output

When outputting leases:
1. Generate the full draft in markdown
2. Offer to create a `.docx` version via the **docx skill** if available
3. Highlight any sections that need attorney review with `[REVIEW]` markers
4. List the top 3 state-specific items the user must verify before signing
5. Always end with: "This draft should be reviewed by a licensed real estate attorney in [STATE] before use."
