# TCA Analytics Audit - Quick Reference

Framework-specific cheat sheet for auditing StateProtocol/WithAssociatedValue projects with trackThisView tracking.

## Search Queries (Copy & Paste into VS Code Find)

### Find trackThisView Function Definition
```
func trackThisView\(|def trackThisView
```
**Regex**: ✓ | **Match Case**: ✗

### Find All trackThisView Calls
```
trackThisView\s*\(
```
**Regex**: ✓

### Find Action Enum Definitions
```
enum Action:|StateProtocol|WithAssociatedValue
```
**Regex**: ✓

### Find Button Actions  
```
case\s*\.button[A-Za-z]*
```
**Regex**: ✓

### Find Label Actions
```
case\s*\.label[A-Za-z]*
```
**Regex**: ✓

### Find Checkbox Actions
```
case\s*\.checkbox[A-Za-z]*
```
**Regex**: ✓

### Find RadioButton Actions
```
case\s*\.radioButton[A-Za-z]*
```
**Regex**: ✓

### Find ListComponent Actions
```
case\s*\.listComponent[A-Za-z]*
```
**Regex**: ✓

### Find trackThisView Tags
```
tags:\s*\[
```
**Regex**: ✓

---

## Quick Analysis Workflow

### 1. Find trackThisView (2 min)
```bash
Command+Shift+F (Open Find)
Search: "trackThisView\("
Regex: ✓
Note: Count results = total screen views tracked
```

### 2. Extract trackThisView Calls (5 min)
```bash
Copy all results → Paste into spreadsheet
Format: [File] [Line] [screenName] [screenModule] [Tags]
```

### 3. Find All Action Enums (3 min)
```bash
Command+Shift+F
Search: "enum Action:"
Regex: ✓
List all reducer files with actions
```

### 4. Extract Tracked Actions (10 min)
```bash
Search each pattern separately:
- button[A-Za-z]* → count results
- label[A-Za-z]* → count results
- checkbox[A-Za-z]* → count results
- radioButton[A-Za-z]* → count results
- listComponent[A-Za-z]* → count results
Total actions = sum of all results
```

### 5. Document Associated Values (5 min)
```
For each action, extract associated value types:
case .buttonDeleteTapped(id: String, confirm: Bool)
  ↓
Associated Values: id (String), confirm (Bool)
```

### 6. Check for Issues (5 min)
```bash
Search: "trackThisView" to verify call count
Search: "case \.button|label|checkbox|radioButton|listComponent"
Verify all tracked actions are accounted for
```

---

## Quick Analysis Summary Template

```
# Analytics Audit Summary

## Screen View Tracking (trackThisView)
- Total trackThisView calls: ___
- Unique screen names: ___
- Unique screen modules: ___
- Most common tags: ___

## Action Tracking
- Total button* actions: ___
- Total label* actions: ___
- Total checkbox* actions: ___
- Total radioButton* actions: ___
- Total listComponent* actions: ___
- TOTAL TRACKED ACTIONS: ___

## Associated Values
- Actions with associated values: ___
- Actions without parameters: ___

## Tag Consistency
- Unique tag keys in trackThisView: ___
- Consistent naming (snake_case): ✓/✗
- Duplicate tags: ___

## Issues Found
- [ ] Unused tracked actions
- [ ] Missing trackThisView on screens
- [ ] Inconsistent tag names
- [ ] Duplicate trackThisView calls per screen
```

---

## Common Screen Modules

Typical screen module names to look for:

| Module | Screens |
|--------|---------|
| HomeModule | home_screen, home_tab |
| ProfileModule | profile, profile_edit |
| PaymentModule | checkout, payment_confirmation |
| OnboardingModule | onboarding_1, onboarding_2 |
| SettingsModule | settings, preferences |
| SearchModule | search_results, search_filters |

---

## Common Tags in trackThisView

Standard tags used across projects:

| Tag | Purpose | Examples |
|-----|---------|----------|
| `version` | Screen version | 1.0, 2.0, 2.1 |
| `flow_type` | Flow variant | express, standard |
| `ab_test` or `variant` | A/B test variant | variant_a, control |
| `user_segment` | User type | premium, free |
| `experiment_id` | Experiment identifier | exp_123 |
| `timestamp` | When viewed | ISO8601 |

---

## Action Type Prefixes

These are the ONLY prefixes extracted:

```
✅ button*          ❌ view*
✅ label*           ❌ screen*
✅ checkbox*        ❌ load*
✅ radioButton*     ❌ navigation*
✅ listComponent*   ❌ api*
```

---

## Spreadsheet Template

Create this to track all findings:

```
SCREEN VIEWS (trackThisView)
| screenName | screenModule | tags | file | line |
|------------|--------------|------|------|------|
| home_screen | HomeModule | version:2.0 | HomeView.swift | 42 |

TRACKED ACTIONS
| actionName | type | file | reducer | associatedValues |
|------------|------|------|---------|-----------------|
| buttonLoginTapped | button | HomeReducer.swift | HomeReducer | - |
| buttonSubmitTapped | button | FormReducer.swift | FormReducer | (FormData) |
| labelTermsTapped | label | OnboardingView.swift | OnboardingReducer | (String) |
```

---

## Checklist: What to Verify

**Screen Views (trackThisView)**
- [ ] All major screens have trackThisView
- [ ] screenName is descriptive
- [ ] screenModule matches feature module
- [ ] Tags are consistent across calls
- [ ] No duplicate calls per screen

**Action Tracking**
- [ ] button actions properly named (button*)
- [ ] label actions properly named (label*)
- [ ] checkbox actions properly named (checkbox*)
- [ ] radioButton actions properly named (radioButton*)
- [ ] listComponent actions properly named (listComponent*)

**Associated Values**
- [ ] Associated values documented
- [ ] Parameter types identified
- [ ] Values used appropriately

---

## Critical Issues to Flag

| Issue | Impact | Example |
|-------|--------|---------|
| **No trackThisView** | 🔴 High | Screen appears but not tracked |
| **Wrong action prefix** | 🔴 High | Action tracked but not button/label/etc |
| **Duplicate trackThisView** | 🟡 Medium | Same screen tracked twice per appear |
| **Missing associated values** | 🟡 Medium | Action has params but not used |
| **Inconsistent tag naming** | 🟡 Medium | screenModule vs screenModule |
| **Unused action** | 🟢 Low | Action defined but never sent |

---

## Report Statistics

Calculate these for your summary:

```
Total Screen Views: [COUNT all unique trackThisView screenNames]
Total Screen Modules: [COUNT all unique screenModules]
Total Tracked Actions: [SUM of button+label+checkbox+radioButton+listComponent]
Coverage: [ESTIMATE: % of user interactions tracked]

Health:
🟢 Good: 80%+
🟡 Fair: 60-79%
🔴 Poor: <60%
```

---

## Quick Commands

### Count total trackThisView calls
```bash
grep -r "trackThisView(" Sources/ --include="*.swift" | wc -l
```

### Find all screen names
```bash
grep -r "trackThisView(" Sources/ --include="*.swift" | grep -o 'screenName:\s*"[^"]*"' | sort -u
```

### Find all tracked actions
```bash
grep -r "case \.button\|case \.label\|case \.checkbox\|case \.radioButton\|case \.listComponent" Sources/ --include="*.swift" | wc -l
```

### Find duplicate trackThisView per file
```bash
grep -r "trackThisView(" Sources/ --include="*.swift" | cut -d: -f1 | sort | uniq -d
```

### Extract all tag keys from trackThisView
```bash
grep -r "tags:" Sources/ --include="*.swift" -A2 | grep -o '"[^"]*":' | sort -u
```

---

## Reference Files

| File | Use For |
|------|---------|
| [TCA Patterns](./tca-patterns.md) | Understanding framework patterns |
| [Analysis Checklist](./analysis-checklist.md) | Systematic review steps |
| [Code Examples](./code-examples.md) | Implementation & testing |
| [Sample Report](./sample-report.md) | Expected output format |
| [How to Analyze](./how-to-analyze.md) | Step-by-step walkthrough |

---

## Common Mistakes to Avoid

❌ **DON'T**
- Track non-button actions with buttonXxx prefix
- Call trackThisView multiple times per screen appearance
- Log PII in trackThisView tags
- Use inconsistent screenModule names
- Skip tracking interactions due to complexity

✅ **DO**
- Use proper action prefixes (button*, label*, etc)
- Call trackThisView once in .onAppear
- Keep tags minimal and consistent
- Document associated value meanings
- Track all major user interactions

---

## Time Estimates

| Activity | Time |
|----------|------|
| Find trackThisView definition | 2 min |
| Count all trackThisView calls | 2 min |
| Extract screen tracking data | 10 min |
| Count tracked actions | 5 min |
| Extract action data | 15 min |
| Analyze associated values | 10 min |
| Check for issues | 10 min |
| Create report | 15 min |
| **Total Quick Audit** | **~70 min** |

---

## Help & Support

**In Copilot Chat**: Ask me to "audit my TCA project for analytics"
**With this skill**: Type `/` and search for "tca-analytics-audit"

**Key Questions**:
- "Where are all my screen views tracked?"
- "Which actions are being tracked?"
- "Are my tag names consistent?"
- "What interactions are missing tracking?"

---

## Quick Analysis Workflow

### 1. Find Service (2 min)
```bash
Command+Shift+F (Open Find)
Search: "AnalyticsService\|TrackingService"
Region: *.swift files
```

### 2. Find All Events (5 min)
```bash
Command+Shift+F
Search: "analytics\.track\("
Regex: ✓
Note: Count results = total events tracked
```

### 3. Export Event List (10 min)
```bash
Copy all search results → Paste into spreadsheet
Format: [File] [Line] [Event Name] [Tags]
```

### 4. Identify Gaps (15 min)
- Review features not in results
- Check error paths
- Look for conditional logic without tracking

### 5. Check for Issues (10 min)
```bash
Search: "password\|token\|email" (in results above)
Verify: No PII in analytics tags
```

---

## Common Events to Look For

### Authentication
- `login_successful`, `login_failed`
- `logout`, `session_ended`
- `password_reset`, `mfa_verified`

### User Actions
- `button_tapped`, `link_clicked`
- `form_submitted`, `form_abandoned`
- `filter_applied`, `sort_changed`

### Navigation
- `screen_viewed`, `tab_switched`
- `back_pressed`, `menu_opened`

### Data Operations
- `data_loaded`, `refresh_completed`
- `item_added`, `item_deleted`
- `search_performed`, `results_shown`

### Errors & Exceptions
- `error_occurred`, `network_failed`
- `api_timeout`, `validation_failed`
- `permission_denied`, `session_expired`

### Completions
- `checkout_completed`, `signup_completed`
- `video_watched`, `article_read`
- `purchase_confirmed`, `booking_made`

---

## Tag Naming Convention

| Tag Name | Format | Example | ✓ / ✗ |
|----------|--------|---------|-------|
| screen_name | snake_case | home, profile | ✓ |
| feature_id | snake_case | payment_v2 | ✓ |
| user_id | snake_case (hashed) | abc123... | ✓ |
| action_type | snake_case | tap, swipe | ✓ |
| time_ms | snake_case | 1234 | ✓ |
| userId | camelCase | ✗ (inconsistent) | ✗ |
| USER_ID | UPPER_CASE | ✗ (inconsistent) | ✗ |

---

## Event Naming Convention

| Pattern | Example | ✓ / ✗ |
|---------|---------|-------|
| `verb_noun` | button_tapped | ✓ |
| `screen_action` | home_search_started | ✓ |
| `feature_event` | payment_completed | ✓ |
| `state_change` | user_authenticated | ✓ |
| `action` | tap (too vague) | ✗ |
| `MyEvent` (PascalCase) | LoginSuccess | ✗ |

---

## Spreadsheet Template

Create this in Excel/Sheets to track findings:

```
| Event | File | Line | Reducer | Tags | Parameters | Notes |
|-------|------|------|---------|------|------------|-------|
| home_viewed | HomeReducer.swift | 47 | HomeReducer | screen,feature | timestamp | OK |
| payment_failed | PaymentReducer.swift | 256 | PaymentReducer | error,feature | error_code | MISSING: error_message |
```

---

## Checklist: What to Verify

**Happy Path**
- [ ] Screen/view loaded event
- [ ] Primary user action event
- [ ] Success completion event

**Error Path**
- [ ] Error occurred event
- [ ] Error code/type included
- [ ] User-facing error shown

**Edge Cases**
- [ ] Cancellation event
- [ ] Back button press event
- [ ] Timeout event
- [ ] Permission denied event

**Parameters**
- [ ] No PII (email, phone, SSN)
- [ ] No auth tokens or passwords
- [ ] Normalized values (consistency)
- [ ] Sufficient detail for debugging

---

## Critical Issues to Flag

| Issue | Impact | Example |
|-------|--------|---------|
| **No error tracking** | 🔴 High | Error path has no analytics.track() |
| **PII in tags** | 🔴 High | email or phone number logged |
| **Duplicate events** | 🟡 Medium | Same event fired twice for one action |
| **Missing context** | 🟡 Medium | Event with no tags/parameters |
| **Inconsistent naming** | 🟡 Medium | user_id vs userId in different places |
| **Dead code** | 🟢 Low | Analytics call never reached |

---

## Report Statistics

Calculate these for your executive summary:

```
Total Unique Events: [COUNT all unique event names]
Features Covered: [COUNT reducers with analytics]
Coverage Score: [ESTIMATE: % of user flows tracked]
PII Issues: [COUNT sensitive data found]
Naming Issues: [COUNT inconsistencies]
Missing Events: [COUNT gaps identified]

Health:
🟢 Good: 85%+
🟡 Fair: 70-84%
🔴 Poor: <70%
```

---

## Quick Commands

### Count total analytics calls
```bash
grep -r "analytics\.track(" Sources/ --include="*.swift" | wc -l
```

### Find event names
```bash
grep -r "analytics\.track(" Sources/ --include="*.swift" | grep -o '"[^"]*"' | sort -u
```

### Find duplicate events
```bash
grep -r "analytics\.track(" Sources/ --include="*.swift" | grep -o '"[^"]*"' | sort | uniq -d
```

### Find PII patterns
```bash
grep -r "email\|phone\|password\|token" Sources/ --include="*.swift"
```

### List by reducer
```bash
grep -r "analytics\.track(" Sources/ --include="*.swift" | cut -d: -f1 | sort -u
```

---

## Reference Files

| File | Use For |
|------|---------|
| [TCA Patterns](./tca-patterns.md) | Understanding logging patterns |
| [Analysis Checklist](./analysis-checklist.md) | Systematic review steps |
| [Code Examples](./code-examples.md) | Implementation & testing |
| [Sample Report](./sample-report.md) | Expected output format |
| [How to Analyze](./how-to-analyze.md) | Detailed walkthrough |

---

## Common Mistakes to Avoid

❌ **DON'T**
- Log before validating state
- Include PII in event tags
- Use inconsistent event naming
- Fire same event twice per action
- Skip error path tracking
- Hardcode analytics (not dependency-injected)
- Log sensitive info (passwords, tokens)

✅ **DO**
- Update state first, then log
- Use anonymized/hashed IDs
- Follow naming conventions
- Test analytics in unit tests
- Track both success and failure
- Inject analytics as dependency
- Validate data before logging

---

## Time Estimates

| Activity | Time |
|----------|------|
| Find service definition | 5 min |
| Extract all events | 15 min |
| Review event coverage | 30 min |
| Check for PII/issues | 15 min |
| Create event catalog | 30 min |
| Identify gaps | 30 min |
| Write recommendations | 30 min |
| **Total Audit** | **~2.5 hours** |

---

## Next Steps After Audit

1. **Share Report**: Send to product/analytics team
2. **Prioritize Fixes**: Identify high-priority gaps
3. **Create Issues**: Add implementation tasks to backlog
4. **Schedule Implementation**: Plan with team
5. **Re-audit**: Schedule 3-month follow-up

---

## Help & Support

**In Copilot Chat**: Ask me to "analyze analytics in my TCA project"
**With this skill**: Type `/` and search for "tca-analytics-audit"

**Questions**?
- Check [How to Analyze](./how-to-analyze.md) for step-by-step guide
- Review [Code Examples](./code-examples.md) for patterns
- Compare against [Sample Report](./sample-report.md)
