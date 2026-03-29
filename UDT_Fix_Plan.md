# UDT Glendale — Complete Fix Plan

**Date:** March 29, 2026 (Updated)
**Prepared by:** Jay (VA) + Claude
**Deadline:** March 30, 2026
**Status:** 11 of 12 issues fixed, 1 remaining (Corey Krebs calendar)

---

## What's Already Fixed (Mar 28)

| # | Fix | Status |
|---|-----|--------|
| 1 | Generated Glendale GHL API key (`pit-556eaf2f`) | Done |
| 2 | Calendar_Bookings — location ID, calendar ID, API key, timezone | Done |
| 3 | Post_Call — location ID, API key | Done |
| 4 | Make_Call — location ID, timezone | Done |
| 5 | InBound_call — location ID, API key | Done |
| 6 | Reactivation_call — location ID (2 nodes), API key | Done |
| 7 | Post_Call — hardcoded Grapevine stage ID replaced with Glendale Nurture ID | Done (Mar 29) |
| 8 | Reactivation_call — phone +18177789641 → +14804055948 | Done (Mar 29) |
| 9 | Glendale Outbound webhook — 00d8c02c → ff0ccb3b (Retell) | Done (Mar 29) |
| 10 | Glendale Inbound webhook — set to 38463130 (Retell) | Done (Mar 29) |
| 11 | Scottsdale Inbound — added all 12 analysis fields (Retell) | Done (Mar 29) |
| 12 | Glendale Lead Pipeline — added 3 missing stages: No-show, Nurture, Not Interested (GHL) | Done (Mar 29, Jay manual) |

---

## Remaining Issues — 1 Item Left

### ISSUE 1: Post_Call has hardcoded Grapevine pipeline stage ID
**Severity:** HIGH — will cause errors when activated
**Where:** `UDT_Glendale_Post_Call` → nodes `update_pipeline_stage` and `update_pipeline_stage1`
**Problem:** Both nodes try to move leads to stage `4355a5f3-f78f-4432-8cde-59f8543bfa69` which is Grapevine's "Nurture / Cold" stage. This ID doesn't exist in Glendale's Lead Pipeline.
**Root cause:** Glendale's Lead Pipeline is missing 3 stages that Grapevine has: **Nurture / Cold**, **No-Show**, **Not Interested**.
**Fix:**
1. Add 3 missing stages to Glendale's Lead Pipeline in GHL (Settings → Opportunities & Pipelines → Lead Pipeline → Add Stage)
2. Get the new stage IDs from GHL
3. Update the `update_pipeline_stage` and `update_pipeline_stage1` nodes with the correct Glendale stage ID
**Who:** Jay adds stages in GHL, Claude updates n8n via API
**Estimated time:** 15 minutes

### ISSUE 2: Reactivation_call uses Grapevine agent + phone number
**Severity:** HIGH — calls would come from wrong number
**Where:** `UDT_Glendale_Reactivation_call` → node `trigger_call`
**Problem:** The workflow triggers `agent_1cbc7d004d3ce6dd4b8cc9ce1f` (UDT_GRAPEVINE_REACTIVATION) and calls from `+18177789641` (Grapevine number). Glendale leads would get calls from a Texas number, not the Arizona number they expect.
**Fix — Option A (Create Glendale Reactivation agent):**
1. Clone UDT_GRAPEVINE_REACTIVATION in Retell
2. Name it UDT_GLENDALE_REACTIVATION
3. Assign Glendale phone number (+14804055948)
4. Update the trigger_call node with new agent ID and Glendale phone
**Fix — Option B (Use existing agent with Glendale number):**
1. Update `from_number` in trigger_call to `+14804055948` (Glendale)
2. Keep agent but pass Glendale-specific dynamic variables
**Who:** Daylon decides which option. Claude updates n8n.
**Estimated time:** 20 minutes (Option A) or 5 minutes (Option B)

### ISSUE 3: Reactivation_call Google Sheet needs verification
**Severity:** MEDIUM — may pull wrong leads
**Where:** `UDT_Glendale_Reactivation_call` → nodes `Get_Leads` and `Update Sheet Row`
**Problem:** Both nodes reference Google Sheet `1MPSK-fH7t58lQ0v2SbtaxEMwWKqSVnmH9iNAJpQPB18`. Need to verify this is actually a Glendale leads sheet, not Grapevine's.
**Fix:** Open the Google Sheet, verify it contains Glendale leads (not Grapevine). If it's Grapevine's sheet, create a new Glendale sheet with the same structure and update the sheet ID.
**Who:** Jay verifies the sheet.
**Estimated time:** 5 minutes

### ISSUE 4: Glendale Outbound agent webhook mismatch
**Severity:** HIGH — post-call data goes to wrong workflow
**Where:** Retell agent `UDT_GLENDALE_OUTBOUND` (agent_15ea3e55bb81dc8f0c7dd4a985)
**Problem:** Agent's webhook points to `00d8c02c` (Grapevine Post_Call), but Glendale has its own Post_Call at `ff0ccb3b`. After each outbound call, the data gets processed by Grapevine's workflow and written to Grapevine's CRM.
**Fix:** In Retell, update UDT_GLENDALE_OUTBOUND's webhook URL from `https://guerilla-fi.app.n8n.cloud/webhook/00d8c02c-...` to `https://guerilla-fi.app.n8n.cloud/webhook/ff0ccb3b-...`
**Who:** Claude can do via Retell API, or Jay in Retell dashboard.
**Estimated time:** 2 minutes

### ISSUE 5: Glendale Inbound agent has no webhook
**Severity:** MEDIUM — no inbound call data captured
**Where:** Retell agent `UDT_Glendale_Inbound` (agent_76baa06b3e89fd15f2cdde7c68)
**Problem:** No webhook URL configured. When someone calls the Glendale number, the bot talks to them, but after the call ends, nothing happens — no CRM update, no follow-up.
**Fix:** Get the webhook URL from `UDT_Glendale_InBound_call` workflow (already fixed) and set it as the agent's webhook in Retell.
**Who:** Claude can do via Retell API.
**Estimated time:** 2 minutes

### ISSUE 6: Corey Krebs missing from Facility Tours calendar
**Severity:** LOW — round-robin only distributes to 2 people instead of 3
**Where:** GHL Glendale → Calendars → Facility Tours (H1XBRM6gSwL85unlTpy9)
**Problem:** Calendar round-robin has Sam Hirschman and Kimball Conklin, but Corey Krebs is not added.
**Fix:** In GHL Glendale, go to Calendars → Facility Tours → Team Members → Add Corey Krebs.
**Who:** Jay in GHL dashboard.
**Estimated time:** 2 minutes

### ISSUE 7: Scottsdale Inbound — no webhook + missing analysis fields
**Severity:** MEDIUM — Scottsdale inbound data not captured
**Where:** Retell agent `UDT SCOTTSDALE INBOUND` (agent_e65f36db6ddc7680b748d86745)
**Problem:** (a) No webhook configured — call data goes nowhere. (b) Only 3 basic extraction fields vs Grapevine Inbound's 12 full analysis fields.
**Fix:**
1. Configure webhook to point to `UDT_Post_Call_Analysis` workflow
2. Add 12 analysis fields matching Grapevine Inbound (lead_qualification_status, training_session_datetime, event_id, training_background, firearms_ownership_status, self_assessed_skill_level, dynamic_training_experience, training_motivation, objection_type, objection_overcome, preferred_callback_time, call_outcome_summary)
**Who:** Claude can do via Retell API.
**Estimated time:** 10 minutes

---

## Structural Issues (Not Blocking but Need Attention)

### Glendale GHL Missing Pipelines
Glendale has 5 pipelines. Scottsdale/Grapevine have 7 each. Missing:
- **1. Enrollment Pipeline** (13 stages)
- **B_F Lead** (4 stages)
- **Corporate Membership** (10 stages)

Decision needed: Does Glendale need these? If yes, create them in GHL with matching stages.

### Duplicate A. Sales Pipeline
Glendale has 2 "A. Sales Pipeline" entries (IDs: `Mf8aAqZC0PiX1lVTX9cS` and `8rsXkYPADNlFA7ZFpMh2`). One should be deleted to avoid confusion.

### Unverified Retell API Keys in Workflows
Two different Retell API keys are used across Glendale workflows:
- `key_504662...` (in Post_Call and Make_Call) — verified: sees 22 UDT agents ✅
- `key_889962...` (in Reactivation_call) — verified: sees 386 agents (parent workspace) ⚠️

Both work, but `key_889962` appears to be a master workspace key with access to 386 agents beyond UDT. May want to standardize to one key for security.

---

## Fix Order (Recommended)

| Step | Issue | Time | Blocker? |
|------|-------|------|----------|
| 1 | Add 3 missing stages to Glendale Lead Pipeline | 5 min | Yes — blocks Post_Call |
| 2 | Fix Post_Call hardcoded stage ID | 5 min | Must do after Step 1 |
| 3 | Fix Outbound agent webhook (Retell) | 2 min | No |
| 4 | Fix Inbound agent webhook (Retell) | 2 min | No |
| 5 | Verify/fix Reactivation Google Sheet | 5 min | No |
| 6 | Fix Reactivation agent + phone | 5-20 min | Needs Daylon decision |
| 7 | Add Corey Krebs to calendar | 2 min | No |
| 8 | Fix Scottsdale Inbound | 10 min | No |
| **Total** | | **~35-50 min** | |

---

## Audit Tool

A reusable audit script has been built at `udt-audit/scripts/udt_audit.py`. Run it anytime to scan all 3 platforms (n8n, Retell, GHL) and automatically flag misconfigurations:

```bash
python3 udt-audit/scripts/udt_audit.py
```

**What it checks:** Cross-location ID contamination in n8n workflows, Retell webhook routing, missing analysis fields, GHL pipeline structure gaps (missing stages/pipelines/duplicates), phone number mismatches, timezone errors, and workflow activation status.

**Last run (Mar 29):** Found 14 issues — 2 CRITICAL (hardcoded Grapevine stage IDs), 4 HIGH, 7 MEDIUM, 1 LOW. All match the fix plan above.

**Additional finding:** Glendale has stage "New" while Grapevine has "New Lead" — consider renaming for consistency.

**Also found:** Scottsdale's UDT_Make_Call has `America/Chicago` timezone in the "Check Business Hours" node (should be `America/Phoenix`). Grapevine's UDT_GV_POST_CALL has `America/Phoenix` in 2 nodes (should be `America/Chicago`). These are pre-existing issues not in the original fix scope but worth noting.
