# Analytics Audit Checklist - Framework Specific

Systematic guide for auditing TCA analytics with StateProtocol, WithAssociatedValue, and trackThisView.

## Phase 1: Framework Components Discovery

- [ ] Find trackThisView function definition
  - Location: _______________
  - Parameters: screenName, screenModule, tags
  - Return type: _______________
  
- [ ] Locate all Action enums
  - Verify StateProtocol conformance: ✓/✗
  - Verify WithAssociatedValue conformance: ✓/✗
  - Count reducers with actions: _______________
  
- [ ] Identify action naming pattern
  - Verify button* actions exist: ✓/✗
  - Verify label* actions exist: ✓/✗
  - Verify checkbox* actions exist: ✓/✗
  - Verify radioButton* actions exist: ✓/✗
  - Verify listComponent* actions exist: ✓/✗

## Phase 2: Screen View Tracking Audit

For each View in the project:

- [ ] **View Name**: _______________
  - [ ] Has .onAppear modifier: ✓/✗
  - [ ] Calls trackThisView: ✓/✗
  - [ ] screenName parameter: _______________ (descriptive: ✓/✗)
  - [ ] screenModule parameter: _______________ (matches module: ✓/✗)
  - [ ] Tags included: ✓/✗
    - [ ] Tag keys documented: _______________
    - [ ] Tag values consistent: ✓/✗
  - [ ] .onDisappear tracking: ✓/✗ (if needed)

## Phase 3: Tracked Action Analysis

For each action starting with button/label/checkbox/radioButton/listComponent:

- [ ] **Action Name**: _______________ 
  - [ ] Type: (button/label/checkbox/radioButton/listComponent)
  - [ ] Has associated values: ✓/✗
    - [ ] Associated value types documented: _______________
  - [ ] Used in reducer: ✓/✗
  - [ ] Action appears in dispatch: ✓/✗
  - [ ] Context is meaningful for tracking: ✓/✗

### Specific Type Checks

**Button Actions**
- [ ] button*Tapped naming pattern used
- [ ] Associated values carry action context
- [ ] All significant buttons tracked

**Label Actions**
- [ ] label* prefix used for clickable labels
- [ ] Associated values contain link/context info
- [ ] All tappable labels tracked

**Checkbox Actions**
- [ ] checkbox* prefix used
- [ ] Boolean toggle captured in action
- [ ] All checkboxes tracked

**RadioButton Actions**
- [ ] radioButton* prefix used
- [ ] Selected option captured as associated value
- [ ] All radio button groups tracked

**ListComponent Actions**
- [ ] listComponent* prefix used
- [ ] Item/index captured in associated values
- [ ] All tappable list items tracked

## Phase 4: Associated Values Verification

## Phase 4: Associated Values Verification

For each tracked action with associated values:

- [ ] **Action**: .button/label/checkbox/radioButton/listComponent___
  - [ ] Parameter names documented: _______________
  - [ ] Parameter types identified: _______________
  - [ ] Values contain tracking context: ✓/✗
  - [ ] No PII in parameters: ✓/✗
  - [ ] Consistent across similar actions: ✓/✗

## Phase 5: Tag Consistency Check

- [ ] **screenName tag**
  - [ ] Used consistently: ✓/✗
  - [ ] Descriptive naming: ✓/✗
  - [ ] Same name for same screen: ✓/✗
  - [ ] Examples: _______________
  
- [ ] **screenModule tag**
  - [ ] Matches feature module names: ✓/✗
  - [ ] Consistent naming: ✓/✗
  - [ ] All screens have module: ✓/✗
  - [ ] Examples: _______________
  
- [ ] **Custom tags**
  - [ ] Snake_case naming: ✓/✗
  - [ ] Values normalized: ✓/✗
  - [ ] No duplicates with different names: ✓/✗
  - [ ] Documented tag meanings: ✓/✗

## Phase 6: Gap Analysis

**Screens Missing trackThisView**
- [ ] List screens without tracking: _______________
- [ ] Reason not tracked (hidden, internal, etc): _______________
- [ ] Should be tracked: ✓/✗

**Actions Missing Tracking**
- [ ] User interactions without action names: _______________
- [ ] Interactions not starting with button/label/checkbox/radioButton/listComponent: _______________
- [ ] Important flows without tracking: _______________

**Incomplete Tracking**
- [ ] Screen appears but trackThisView missing tags: ✓/✗
- [ ] Actions with associated values not using them: ✓/✗
- [ ] Duplicate trackThisView calls per screen: ✓/✗

## Phase 7: Quality & Consistency Review

- [ ] **Action Naming**
  - [ ] All button actions follow pattern: ✓/✗
  - [ ] All label actions follow pattern: ✓/✗
  - [ ] No typos or inconsistencies: ✓/✗
  - [ ] Examples of good naming: _______________
  - [ ] Examples of naming issues: _______________

- [ ] **Screen View Naming**
  - [ ] screenName values are descriptive: ✓/✗
  - [ ] screenModule values match modules: ✓/✗
  - [ ] No duplicate screen names: ✓/✗
  - [ ] Examples: _______________

- [ ] **Tag Consistency**
  - [ ] Tag keys use snake_case: ✓/✗
  - [ ] Tag values are consistent type: ✓/✗
  - [ ] No repeated tags with different spelling: ✓/✗
  - [ ] All required tags present: ✓/✗

## Phase 8: PII & Security Audit

- [ ] **Associated Values Check**
  - [ ] No email addresses: ✓/✗
  - [ ] No phone numbers: ✓/✗
  - [ ] No user IDs (unencrypted): ✓/✗
  - [ ] No passwords or tokens: ✓/✗
  - [ ] No credit card data: ✓/✗
  - [ ] No sensitive dates: ✓/✗

- [ ] **trackThisView Tags Check**
  - [ ] No PII in tag keys: ✓/✗
  - [ ] No PII in tag values: ✓/✗
  - [ ] No locations or precise data: ✓/✗

## Phase 9: Testing & Verification

## Phase 9: Testing & Verification

- [ ] **Unit Tests Exist**
  - [ ] Test trackThisView calls in views
  - [ ] Test tracked action cases in reducers
  - [ ] Verify associated values passed correctly
  - [ ] Mock trackThisView for view tests
  
- [ ] **Manual Testing**
  - [ ] Open each screen and verify trackThisView called
  - [ ] Tap each button/label and verify action sent
  - [ ] Toggle each checkbox and verify state change
  - [ ] Select each radio option and verify state change
  - [ ] Tap list items and verify action sent
  - [ ] Check network/analytics tool for events
  
- [ ] **Analytics Dashboard**
  - [ ] All trackThisView events appear: ✓/✗
  - [ ] All tracked actions appear: ✓/✗
  - [ ] Event data is complete: ✓/✗
  - [ ] Tag values are correct: ✓/✗

## Phase 10: Scoring & Recommendations

### Coverage Assessment

| Metric | Score | Status |
|--------|-------|--------|
| trackThisView coverage | __/100 | 🟢/🟡/🔴 |
| Tracked action coverage | __/100 | 🟢/🟡/🔴 |
| Associated value coverage | __/100 | 🟢/🟡/🔴 |
| Tag consistency | __/100 | 🟢/🟡/🔴 |
| PII compliance | __/100 | 🟢/🟡/🔴 |
| **Overall Health** | __/100 | 🟢/🟡/🔴 |

### Interpretation

- 🟢 Good (85%+): Solid coverage, minor improvements only
- 🟡 Fair (70-84%): Address missing tracking systematically
- 🔴 Poor (<70%): Significant work needed

### Priority Recommendations

**High Priority** (Blocks release):
1. _________________________________________________
2. _________________________________________________
3. _________________________________________________

**Medium Priority** (Next sprint):
1. _________________________________________________
2. _________________________________________________
3. _________________________________________________

**Low Priority** (Backlog):
1. _________________________________________________
2. _________________________________________________
3. _________________________________________________

## Notes Section

### Key Findings
- 

### Patterns Observed
- 

### Issues Found
- 

### Team Recommendations
- 

### Follow-up Actions
- Date to re-audit: _______________
- Owner: _______________
- Success criteria: _______________ 
