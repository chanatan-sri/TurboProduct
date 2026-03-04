# Capability: Contact Compliance

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Ensure all customer interactions comply with Thai debt collection regulations regarding contact frequency, timing, and methods — by consuming compliance signals from DaVinci and enforcing them in the work queue and task execution flow. Also verify that recorded contact outcomes are genuine via 3CX call log cross-reference.

## ⚠️ Critical Boundary Note

**This capability does NOT own contact compliance data.**

- The contact log, frequency limits, and cross-product aggregation are owned by **DaVinci** (`collection-contact-compliance` capability).
- Sensei is a **consumer** of DaVinci compliance events. When DaVinci says "limit reached," Sensei enforces the skip.
- If a future product (automated SMS campaigns, call center system) also needs compliance enforcement, it consumes the same DaVinci events — not Sensei events.
- Sensei feeds contact outcomes back to DaVinci (ContactRecorded event) so the centralized count stays accurate.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| DaVinci Event Subscriber | Draft | Subscribe to ContactLimitReached, ContactLimitApproaching, ContactWindowClosed events |
| Contact Limit Enforcement | Draft | Auto-skip contact tasks when ContactLimitReached; surface in supervisor exception panel |
| Contact Limit Warning Badge | Draft | Show "⚠️ เหลือ 1 ครั้ง" warning badge on customer task card when ContactLimitApproaching |
| Contact Window Enforcement | Draft | Do not present contact tasks outside business hours (ContactWindowClosed event) |
| ContactRecorded Feedback | Draft | Publish ContactRecorded event to DaVinci after every contact task completion |
| 3CX Call Log Verification | Draft | Cross-reference recorded Call outcomes against 3CX call logs after task completion |
| Verification Status Display | Draft | Show verification status on each Call task: ✅ Verified / ⚠️ Unverified / ❌ Mismatch |
| Supervisor Mismatch Panel | Draft | Surface Unverified and Mismatch tasks in supervisor exception panel for review |

---

## Business Rules

### DaVinci Event Consumption Rules

| Event | Sensei Response |
|-------|----------------|
| `ContactLimitReached` | Auto-skip all remaining contact tasks for that customer today; surface in supervisor exception panel |
| `ContactLimitApproaching` | Show warning badge on customer's task card: "⚠️ เหลือ X ครั้ง" |
| `ContactWindowClosed` | Do not present contact tasks outside business hours; CO sees "Contact window closed" indicator |

### ContactRecorded Feedback Rule

When Sensei records a contact outcome (CO completes a Call or Visit task), it **must** publish a `ContactRecorded` event to DaVinci so the centralized contact count stays accurate.

```
ContactRecorded {
  customer_id       // DaVinci customer ID
  channel           // call | visit | sms | email
  subsidiary_id     // originating subsidiary
  timestamp         // when contact was made
  outcome           // CO-recorded outcome
  task_id           // Sensei task ID for cross-reference
}
```

### 3CX Call Log Verification Rules (Trust-but-Verify)

| Verification Check | Criteria |
|-------------------|---------|
| Call placed | Outbound call record exists in 3CX log |
| Duration > 0 | Call connected and lasted > 0 seconds |
| Caller ID match | CO's phone number matches logged caller ID |
| Timestamp match | Call timestamp within ±5 minutes of task completion time |

| Verification Status | Meaning | Supervisor Action |
|--------------------|---------|--------------------|
| ✅ Verified | All criteria met — call log matches | No action needed |
| ⚠️ Unverified | No matching log found | Surface in exception panel for review |
| ❌ Mismatch | Log contradicts outcome (e.g., "PTP" recorded but call duration was 0 seconds) | Surface in exception panel; may require investigation |

### Trust-but-Verify Design Rule

3CX verification does **not** block CO from recording outcomes in real-time — it performs after-the-fact verification. This maintains throughput while enabling accountability. Mismatches are flagged for supervisor review, not auto-reversed.

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| DaVinci event latency tolerance | Contact limit enforcement must be effective within 30 seconds of DaVinci event emission |
| No data ownership | Sensei must not maintain its own contact frequency count — always consume from DaVinci |
| ContactRecorded always published | On every contact task closure, ContactRecorded must be published to DaVinci (not optional) |
| Non-blocking verification | 3CX verification must not block CO workflow; runs asynchronously after task closure |

---

## Open Questions

- What is the exact API contract for querying DaVinci contact frequency before a call task is displayed? (REST polling vs. real-time event-driven approach)
- What happens if 3CX API is unavailable? Should missing verification default to "Unverified" or be left blank?
