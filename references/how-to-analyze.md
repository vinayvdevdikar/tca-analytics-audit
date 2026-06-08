# Analytics Audit - Framework-Specific Analysis

Guide for auditing TCA projects using StateProtocol, WithAssociatedValue, and trackThisView.

## Prerequisites

- Open your TCA iOS project in VS Code
- Have access to all source files (typically in `Sources/` or `Packages/`)
- Project uses StateProtocol and WithAssociatedValue for actions
- Project has trackThisView function for screen tracking

## Step 1: Find trackThisView Function Definition

Start by locating the trackThisView function and understanding its signature.

### Search Pattern 1: trackThisView Definition

Use VS Code search to find:
```
SearchTerm: func trackThisView|trackThisView\(
Type: Swift files (.swift)
Match: Regex
```

Look for:
- Function signature
- Parameters (screenName, screenModule, tags)
- File location
- Any configuration or dependencies

### What to document:
```
trackThisView Location: ________________
Parameters:
  - screenName: _____________
  - screenModule: _____________
  - tags: _____________
```

## Step 2: Find All Action Enum Definitions

Locate all Action enums that use StateProtocol and WithAssociatedValue.

### Search Pattern 2: Action Enums

Use VS Code search:
```
SearchTerm: enum Action:|enum.*Action.*:
Type: Swift files
Match: Regex pattern
```

Look for:
- `StateProtocol` conformance
- `WithAssociatedValue` conformance
- Action cases starting with: button, label, checkbox, radioButton, listComponent

### Document:
```
Action Enums Found:
  File: ________________
  Reducer: ________________
  StateProtocol: ✓/✗
  WithAssociatedValue: ✓/✗
  
  Tracked Action Cases:
  - .button* cases: __________ 
  - .label* cases: __________
  - .checkbox* cases: __________
  - .radioButton* cases: __________
  - .listComponent* cases: __________
```

---

## Step 3: Extract All Tracked Action Cases

Find every action case matching the tracked prefixes.

### Search Pattern 3: Button Actions

```
SearchTerm: case \s*\.button[A-Za-z]*
Type: Swift files
Match: Regex
```

### Search Pattern 4: Label Actions

```
SearchTerm: case \s*\.label[A-Za-z]*
Type: Swift files
Match: Regex
```

### Search Pattern 5: Checkbox Actions

```
SearchTerm: case \s*\.checkbox[A-Za-z]*
Type: Swift files
Match: Regex
```

### Search Pattern 6: RadioButton Actions

```
SearchTerm: case \s*\.radioButton[A-Za-z]*
Type: Swift files
Match: Regex
```

### Search Pattern 7: ListComponent Actions

```
SearchTerm: case \s*\.listComponent[A-Za-z]*
Type: Swift files
Match: Regex
```

### Create Spreadsheet

Build a table of all tracked actions:

| Action Name | File | Reducer | Associated Values | Notes |
|------------|------|---------|-------------------|-------|
| buttonLoginTapped | HomeReducer.swift | HomeReducer | - | Standard tap |
| buttonSubmitTapped | FormReducer.swift | FormReducer | (FormData) | Carries form context |
| labelPrivacyTapped | OnboardingReducer.swift | OnboardingReducer | - | Link tap |
| [continue...] | | | | |

---

## Step 4: Extract All trackThisView Calls

Find every screen tracking call.

### Search Pattern 8: trackThisView Calls

```
SearchTerm: trackThisView\s*\(
Type: Swift files
Match: Regex
```

For each match, document:
- **File & Line**: Where it's called
- **Screen Name**: screenName parameter value
- **Screen Module**: screenModule parameter value  
- **Tags**: All tags passed in the dictionary
- **View**: Which View contains this call

### Create Screen Tracking Table

| Screen Name | Module | Tags | File | View | Line |
|------------|--------|------|------|------|------|
| home_screen | HomeModule | version, experiment | HomeView.swift | HomeView | 42 |
| checkout | PaymentModule | flow_type, variant | CheckoutView.swift | CheckoutView | 89 |
| [continue...] | | | | | |

---

## Step 5: Map Action-to-Tracking Data Flow

For each tracked action, document its associated values (WithAssociatedValue).

### Analysis

```
Extract the associated value types from action definitions:

case .buttonDeleteTapped(itemId: String, shouldConfirm: Bool)
  ↓
Associated Values for Tracking:
  - itemId: String
  - shouldConfirm: Bool
  
These become implicit tracking attributes:
  {
    "action_type": "button",
    "action_name": "delete_tapped",
    "item_id": <value>,
    "should_confirm": <value>
  }
```

### Document:
```
Action: .buttonSubmitTapped
Associated Values:
  - formData: FormData (contains fields: email, password, terms)
  
Implicit Tracking Attributes:
  - form_data_is_valid: Bool
  - form_field_count: Int
```

---

## Step 6: Analyze Tag Consistency

Extract all tags used in trackThisView calls.

### Search Pattern 9: Extract Tag Keys

```
SearchTerm: tags:\s*\[
Type: Swift files
Match: Regex
```

### Create Tag Registry

Document every tag used:

| Tag Key | Used In | Example Values | Consistency |
|---------|---------|-----------------|-------------|
| screenModule | trackThisView | HomeModule, PaymentModule | ✓ Consistent |
| version | trackThisView | 2.0, 2.1 | ✓ Consistent |
| flow_type | trackThisView | express, standard | ✓ Consistent |
| variant | trackThisView | variant_a, control | ⚠️ Check: variant vs ab_test |

### Check for:
- [ ] Naming consistency (snake_case vs camelCase)
- [ ] Duplicate tag keys with different names
- [ ] Unused tag keys
- [ ] Tag values that should be enumerated
