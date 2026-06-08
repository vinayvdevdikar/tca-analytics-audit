# TCA Analytics Patterns - Framework Reference

Patterns specific to projects using **StateProtocol**, **WithAssociatedValue**, and **trackThisView()** API.

## What Gets Audited

### Action Types (ONLY these are extracted):
- ✅ `button*` - Actions starting with "button"
- ✅ `label*` - Actions starting with "label"
- ✅ `checkbox*` - Actions starting with "checkbox"
- ✅ `radioButton*` - Actions starting with "radioButton"
- ✅ `listComponent*` - Actions starting with "listComponent"

### Screen View Events (ALL are extracted):
- ✅ `trackThisView(...)` function calls in view lifecycle

### NOT Audited:
- ❌ Other action types (navigation, data, etc.)
- ❌ Direct analytics.track() calls (rely on action naming)
- ❌ Non-tracking function calls

---

## Pattern Recognition Guide

### Pattern 1: StateProtocol & WithAssociatedValue Actions

**Signature**: Enum conforming to StateProtocol and WithAssociatedValue

```swift
enum Action: Equatable {
    case buttonLoginTapped
    case buttonSubmitTapped(FormData)
    case labelSettingsTapped
    case checkboxAgreedTapped(Bool)
    case radioButtonPaymentSelected(PaymentMethod)
    case listComponentItemTapped(ItemID)
}

extension Action: StateProtocol, WithAssociatedValue {
    // Framework protocols applied
}
```

**Search terms**:
- `enum Action:` followed by action cases
- Case names starting with: `button`, `label`, `checkbox`, `radioButton`, `listComponent`
- `StateProtocol`, `WithAssociatedValue` conformance

### Pattern 2: Screen View Tracking with trackThisView

**Signature**: `trackThisView()` call in view's lifecycle method

```swift
struct HomeView: View {
    let store: StoreOf<HomeReducer>
    
    var body: some View {
        VStack { ... }
            .onAppear {
                trackThisView(
                    screenName: "home_screen",
                    screenModule: "HomeModule",
                    tags: ["version": "2.0", "experiment": "variant_a"]
                )
            }
    }
}
```

**Search terms**:
- `trackThisView(`
- Parameters: `screenName`, `screenModule`, `tags`
- Located in `.onAppear`, `.onDisappear`, or view initialization

### Pattern 3: Action Case in Reducer with Associated Values

**Signature**: Action case with WithAssociatedValue containing tracking context

```swift
case .buttonSubmitTapped(let formData):
    // Associated value contains context for implicit tracking
    state.lastSubmittedData = formData
    return .run { send in
        let result = await api.submit(formData)
        await send(.response(result))
    }
```

**Search terms**:
- `case .button*` (extract these cases)
- `case .label*` 
- `case .checkbox*`
- `case .radioButton*`
- `case .listComponent*`
- Associated values via `WithAssociatedValue`

### Pattern 4: Action Type Extraction from Enum Cases

The audit extracts action cases using this logic:

```swift
// Extract these action types from Action enum
case .buttonLoginTapped              ✅ Extract
case .buttonSubmitTapped(data)       ✅ Extract with associated value
case .labelTermsClicked              ✅ Extract
case .checkboxAgreedToggled(Bool)    ✅ Extract
case .radioButtonPaymentSelected     ✅ Extract
case .listComponentRowTapped(id)     ✅ Extract

// DON'T extract (different prefix)
case .viewAppeared                   ❌ Skip
case .loadDataResponse               ❌ Skip
case .navigationPop                  ❌ Skip
case .apiError(Error)                ❌ Skip
```

---

## Screen View Tracking with trackThisView()

### Basic trackThisView Call

```swift
.onAppear {
    trackThisView(
        screenName: "product_detail",
        screenModule: "ProductModule"
    )
}
```

### trackThisView with Tags

```swift
.onAppear {
    trackThisView(
        screenName: "checkout",
        screenModule: "PaymentModule",
        tags: [
            "flow_type": "express",
            "user_segment": "premium",
            "ab_test": "variant_b"
        ]
    )
}
```

**Search terms**:
- `trackThisView(`
- Extract: `screenName`, `screenModule`, and all `tags`

---

## Data Extraction from WithAssociatedValue

The framework allows associated values in actions:

```swift
// Audit extracts the associated value context
case .buttonDeleteTapped(itemId: String, shouldConfirm: Bool)

// This context becomes implicit tracking data
case .checkboxPurchaseTermsTapped(accepted: Bool)

// Extract these values as part of event parameters
case .listComponentItemSelected(index: Int, itemType: String)
```

**Search terms**:
- Action cases with `(` and `)` parameters
- Extract parameter names and types
- These become implicit event attributes

---

## Action Naming Convention

### Button Actions
```
buttonTapped                    ✅ Generic tap
buttonLoginTapped              ✅ Specific action
buttonSubmitFormTapped         ✅ Multi-word action
button<ActionName>Tapped       ✅ Pattern
```

### Label Actions
```
labelClicked                   ✅ Generic click
labelPrivacyPolicyTapped       ✅ Specific link
labelTermsOfServiceTapped      ✅ Specific link
label<ActionName>Tapped        ✅ Pattern
```

### Checkbox Actions
```
checkboxToggled                ✅ Generic toggle
checkboxTermsAcceptedToggled   ✅ Specific checkbox
checkbox<OptionName>Toggled    ✅ Pattern
```

### RadioButton Actions
```
radioButtonSelected            ✅ Generic selection
radioButtonPaymentMethodSelected
radioButton<OptionName>Selected ✅ Pattern
```

### ListComponent Actions
```
listComponentTapped            ✅ Generic tap
listComponentItemTapped        ✅ Item tap
listComponentSectionExpanded   ✅ Section action
listComponent<ActionName>       ✅ Pattern
```

---

## Tag Key Recommendations for Actions

When using tracked actions, use consistent tags:

| Tag Key | Purpose | Example |
|---------|---------|---------|
| `action_type` | Type of interaction | button, label, checkbox, radio, list |
| `action_name` | Specific action identifier | login_button, submit_form |
| `screen_name` | Current screen context | home, profile, checkout |
| `component_id` | Specific component identifier | button_id_42 |
| `user_segment` | User classification | premium, free, trial |
| `ab_variant` | A/B test assignment | variant_a, control |
| `timestamp` | When action occurred | ISO8601 string |
| `interaction_time_ms` | Time spent before action | 1234 |

---

## Data Flow: Action → Tracking

```
User taps button
    ↓
.buttonSubmitTapped(formData) action sent
    ↓
Reducer receives action
    ↓
Framework implicitly tracks:
  - action_type: "button"
  - action_name: "submit_tapped"
  - associated_values: {formData properties}
    ↓
Event logged to analytics
```

---

## Common Issues to Flag

1. **Wrong Action Prefix**: Action tracked but doesn't start with button/label/checkbox/radioButton/listComponent
2. **Missing trackThisView**: Screen appears but no trackThisView call
3. **Duplicate Actions**: Same action case in multiple reducers
4. **Empty Associated Values**: Action with parameters but values not documented
5. **Screen View Duplication**: trackThisView called multiple times per screen
6. **Tag Consistency**: screenModule named differently in different calls
