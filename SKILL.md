---
name: tca-analytics-audit
description: 'Audit TCA-based iOS projects using StateProtocol & WithAssociatedValue to identify all analytics events. Extracts tracked actions (button*, label*, listComponent*, checkbox*, radioButton*) and screen views via trackThisView. Use when: analyzing analytics coverage, discovering all tracked UI interactions, generating event reports, validating tracking consistency.'
argument-hint: 'Provide the workspace root path'
user-invocable: true
---

# TCA Analytics Audit Skill

Specialized analysis tool for iOS projects using **SwiftUI**, **Combine**, and **The Composable Architecture (TCA)** with **StateProtocol** and **WithAssociatedValue** pattern. Identifies and reports all analytics tracking for UI interactions and screen views.

## When to Use

- **Audit existing implementation**: Find all analytics events in your codebase
- **Generate analytics documentation**: Create comprehensive event catalog with tags
- **Validate tracking coverage**: Ensure important user actions are tracked
- **Identify gaps or duplicates**: Find missing or redundantly logged events
- **Onboard new features**: Understand analytics patterns before adding new tracking
- **Compliance reviews**: Document all user-facing events for privacy/GDPR audits

## How It Works

The skill performs a multi-stage analysis:

1. **Codebase Scanning**: Searches for analytics logging patterns in TCA actions, effects, and reducers
2. **Event Extraction**: Identifies all tracked events, their tags, parameters, and context
3. **Flow Analysis**: Maps UI actions → TCA actions → analytics events
4. **Report Generation**: Creates categorized documentation with event taxonomy

## Procedure

### Step 1: Prepare Your Workspace

Ensure your TCA project is open in VS Code. The analysis works best with projects structured as:

```
MyApp/
├── Sources/
│   ├── Features/           # Feature modules with TCA
│   │   ├── Feature1/
│   │   │   ├── Feature1Reducer.swift
│   │   │   ├── Feature1View.swift
│   │   │   └── Feature1State.swift
│   │   └── Feature2/
│   ├── Analytics/          # Analytics service
│   │   ├── AnalyticsService.swift
│   │   └── AnalyticsEvents.swift
│   └── App/
└── Package.swift
```

### Step 2: Start the Audit

Use the analysis script to scan your codebase for analytics patterns:

[TCA Analytics Pattern Reference](./references/tca-patterns.md) — Common patterns used for logging in TCA
[Analysis Checklist](./references/analysis-checklist.md) — What to look for during manual review

### Step 3: Review Analysis Results

The audit will generate a report containing:

- **Events Catalog**: All tracked analytics events organized by feature
- **Action Flow Map**: How UI actions trigger TCA actions and analytics
- **Tag Registry**: All tags/event names with frequencies
- **Parameter Analysis**: Data sent with each event
- **Gaps and Recommendations**: Areas that need better coverage

## Key Framework Patterns

### Tracked Action Types

This audit **only extracts** actions matching these prefixes:
- `button*` - Button tap interactions
- `label*` - Label/text interactions
- `listComponent*` - List item/collection interactions
- `checkbox*` - Checkbox state changes
- `radioButton*` - Radio button selections

### Pattern 1: StateProtocol with WithAssociatedValue Actions

```swift
enum Action: Equatable {
    case buttonTapped(ButtonAction)
    case labelClicked(String)
    case checkboxToggled(Bool)
    case radioButtonSelected(String)
}

extension Action: StateProtocol, WithAssociatedValue {
    // Framework automatically applies these protocols
}
```

**Audit Focus**: All cases starting with `button`, `label`, `checkbox`, `radioButton`

### Pattern 2: Screen View Tracking via trackThisView

```swift
.onAppear {
    trackThisView(
        screenName: "home_screen",
        screenModule: "HomeModule",
        tags: ["version": "2.1"]
    )
}
```

**Audit Focus**: All `trackThisView()` calls to capture screen view events

### Pattern 3: Action with Analytics Inside Reducer

```swift
case .buttonTapped(let action):
    analytics.track(
        event: "button_tapped",
        tags: ["button_id": action.id, "screen": state.currentScreen]
    )
    return .none
```

### Pattern 4: Reducer with Multiple Action Cases

```swift
switch action {
case .buttonLoginTapped:
    // Implicit tracking via action name convention
    return performLogin()
    
case .checkboxTermsAccepted:
    state.acceptedTerms = true
    return .none
    
case .radioButtonPaymentSelected(let method):
    state.selectedPayment = method
    return .none
}
```

## What the Report Includes

### 1. **Events Summary**
- Total tracked actions (button*, label*, checkbox*, radioButton*, listComponent*)
- Total screen views via `trackThisView()`
- Events per feature
- Event frequency distribution

### 2. **Screen View Events**
For each `trackThisView()` call:
- Screen name
- Screen module
- Tags passed
- File location
- Associated reducer

### 3. **Action Events**
For each tracked action:
- Action case name
- Action type (button, label, checkbox, etc.)
- Feature module
- Associated parameters via WithAssociatedValue
- File location
- Reducer context

### 4. **Tags Taxonomy**
- Complete list of all tags used in actions
- Complete list of all tags used in screen views
- Tag values and types
- Tag consistency check

### 5. **Flow Mapping**
- UI Action → StateProtocol enum case → Analytics tags
- Screen navigation flows with trackThisView tracking

### 6. **Gaps & Recommendations**
- Missing action tracking (untracked user interactions)
- Missing screen view tracking
- Inconsistent tag naming
- Duplicate tracking

## Manual Review Steps

After the automated analysis:

1. **Verify Event Names**: Check for typos, inconsistencies in naming
2. **Validate Parameters**: Ensure sensitive data (PII) is not being logged
3. **Check Conditionals**: Verify analytics calls aren't accidentally in debug-only code
4. **Review Timing**: Confirm events fire when intended (not before errors occur)
5. **Test Coverage**: Manually verify tracked events appear in your analytics dashboard

## Output Formats

The audit produces:

- **Markdown Report**: Human-readable documentation with tables and sections
- **JSON Export**: Machine-readable event catalog for tooling integration
- **CSV Tags List**: For spreadsheet analysis and tracking

## Example Output Structure

```
# Analytics Audit Report
Generated: 2026-06-06

## Summary
- Total Events: 47
- Features Audited: 8
- Coverage: 89%

## Events by Feature

### HomeFeature
- **view_loaded** (HomeFeature/HomeReducer.swift:42)
  - Tags: [screen, feature_name]
  - Parameters: [duration, timestamp]
  - Flow: HomeView loads → HomeReducer.init → onTask effect

- **search_performed** (HomeFeature/SearchReducer.swift:158)
  - Tags: [interaction, search]
  - Parameters: [query_length, result_count, time_ms]
  - Flow: SearchView → searchAction → API call with analytics

## Tags Registry
- screen (used in 12 events)
- feature_name (used in 8 events)
- interaction (used in 23 events)
- error (used in 5 events)
...
```

## Tips for Best Results

- **Run after code changes**: Re-audit after major refactoring
- **Compare reports**: Track analytics coverage across releases
- **Combine with linting**: Integrate with code review processes
- **Document patterns**: Share the report with your team's analytics stakeholders
- **Link to tracking plan**: Cross-reference with your product's analytics plan

## References

- [TCA Patterns Guide](./references/tca-patterns.md) — Common analytics patterns in TCA
- [Analysis Checklist](./references/analysis-checklist.md) — Systematic review steps
- [Sample Report Template](./references/sample-report.md) — What to expect in output
