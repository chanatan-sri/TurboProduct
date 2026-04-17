# UI Mockup: Cash Confirmation Screen (`ConfirmationCash`)

**Feature**: [Cash Confirmation Gate](../../capabilities/cash-disbursement/features/FEATURE_cash-confirmation-gate.md)
**State**: `ConfirmationCash`
**Actor**: Loan Officer
**Created**: 2026-03-31

---

## Screen Mockup

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  ← Back          ONIGIRI — Loan Origination                        🔔  👤 สมชาย │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  Application #LOS-2026-048291                          Campaign: CarTitle PRO   │
│  นายธนกร วงษ์สุวรรณ  •  ID: 1-1009-12345-67-8         Branch: สาขาลาดพร้าว    │
│                                                                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ⚠️  ยืนยันก่อนเบิกจ่ายเงินสด                                                  │
│  Cash Disbursement Confirmation Required                                        │
│                                                                                 │
│  ข้อมูลใบสมัครมีการเปลี่ยนแปลงหลังจากผ่านการอนุมัติแล้ว                        │
│  The following fields were changed after approval. Please review                │
│  carefully before releasing funds — cash disbursement is irreversible.          │
│                                                                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  CHANGED FIELDS                                           Approved → Now        │
│  ──────────────────────────────────────────────────────────────────────────     │
│                                                                                 │
│  💰 Disbursement Amount        ฿ 320,000.00  →  ฿ 285,000.00   ▼ ฿35,000      │
│                                                                                 │
│  📅 Loan Term                  24 months     →  18 months                      │
│                                                                                 │
│  🏦 Disbursement Branch        สาขาลาดพร้าว  →  สาขารามคำแหง                  │
│                                                                                 │
│  3 field(s) changed                                                             │
│                                                                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  LOAN SUMMARY (Current)                                                         │
│  ──────────────────────────────────────────────────────────────────────────     │
│  Borrower          นายธนกร วงษ์สุวรรณ                                          │
│  Product           สินเชื่อทะเบียนรถยนต์ (Car Title Loan)                      │
│  Collateral        Toyota Fortuner 2022 (ทะเบียน กข-1234 กรุงเทพฯ)            │
│  Approved Amount   ฿ 320,000.00                                                 │
│  Disbursement Amt  ฿ 285,000.00     ← amount to be disbursed                   │
│  Interest Rate     18% p.a.                                                     │
│  Loan Term         18 months                                                    │
│  Monthly Payment   ฿ 17,833.00 / month                                         │
│  Disbursement      Cash — สาขารามคำแหง                                          │
│  Risk Level        MEDIUM                                                       │
│  Approved By       นางสาวพิมพ์ใจ รักดี  •  2026-03-28 14:32                   │
│                                                                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │ ✅ I have reviewed the changes and confirm that the revised terms        │  │
│  │    are intentional and authorised for cash disbursement.                 │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│         ┌────────────────────────────┐    ┌────────────────────────────┐       │
│         │   ✗  ปฏิเสธ / Reject       │    │   ✓  ยืนยันเบิกจ่าย        │       │
│         │   Return to Draft          │    │   Confirm & Disburse       │       │
│         └────────────────────────────┘    └────────────────────────────┘       │
│                                                (primary — filled button)        │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Reject Flow — Reason Capture

When the officer clicks **Reject**, a modal appears before returning to Draft:

```
┌──────────────────────────────────────────────────────────┐
│  ปฏิเสธการเบิกจ่าย / Reject Disbursement                  │
│  ──────────────────────────────────────────────────────── │
│                                                           │
│  เหตุผลในการปฏิเสธ / Reason for rejection:               │
│                                                           │
│  ○  จำนวนเงินไม่ถูกต้อง (Incorrect disbursement amount)  │
│  ○  ข้อมูลผิดพลาด (Data entry error — needs correction)  │
│  ○  ลูกค้าต้องการเปลี่ยนแปลง (Borrower requests change)  │
│  ○  อื่น ๆ (Other)                                        │
│                                                           │
│  หมายเหตุเพิ่มเติม / Note (optional):                    │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ ลูกค้าต้องการลดวงเงินและระยะเวลา ให้ตรวจสอบ...     │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                           │
│       [ Cancel ]          [ Confirm Rejection ]           │
└──────────────────────────────────────────────────────────┘
```

---

## Confirm Flow — Audit Confirmation Toast

After clicking **Confirm & Disburse**, a brief confirmation toast appears before the state advances:

```
┌──────────────────────────────────────────────────────────────┐
│  ✅ Disbursement confirmed                                    │
│  Officer: สมชาย ทองดี  •  2026-03-31 10:47:22               │
│  Ref: CONF-2026-048291-001                                   │
│  Advancing to Create Facility...                             │
└──────────────────────────────────────────────────────────────┘
```

---

## Mock Data Reference

| Field | Approved (Snapshot) | Current |
|-------|--------------------:|--------:|
| Disbursement Amount | ฿ 320,000.00 | ฿ 285,000.00 |
| Loan Term | 24 months | 18 months |
| Disbursement Branch | สาขาลาดพร้าว | สาขารามคำแหง |
| Interest Rate | 18% p.a. | 18% p.a. *(unchanged)* |
| Collateral | Toyota Fortuner 2022 | Toyota Fortuner 2022 *(unchanged)* |
| Risk Level | MEDIUM | MEDIUM *(unchanged)* |

> Unchanged fields are **not shown** on the confirmation screen — only the diff is surfaced to reduce cognitive load.

---

## Design Notes

| Decision | Rationale |
|----------|-----------|
| Show only changed fields, not full summary | Reduces noise; officer's attention should go to what changed, not what stayed the same |
| Full loan summary shown below the diff | Provides context — officer can verify the complete picture without switching screens |
| Acknowledgement checkbox before confirm button | Forces deliberate action; prevents accidental confirmation on an irreversible operation |
| Reject reason is structured (dropdown) + optional free text | Structured reasons enable reporting; free text captures nuance |
| Confirm button is primary (filled), Reject is secondary (outline) | Cash disbursement is the expected happy path; reject is the exception |
| Amounts shown in THB with comma formatting | Matches branch operational standards |
