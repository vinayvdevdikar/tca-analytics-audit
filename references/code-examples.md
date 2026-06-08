# TCA Analytics - Framework Code Examples

Code examples for StateProtocol, WithAssociatedValue, and trackThisView integration.

## Action Enum with StateProtocol & WithAssociatedValue

```swift
import ComposableArchitecture

struct HomeReducer: Reducer {
    enum Action: Equatable {
        // Button actions (tracked)
        case buttonLoginTapped
        case buttonSubmitTapped(FormData)
        case buttonDeleteTapped(itemId: String)
        
        // Label actions (tracked)
        case labelPrivacyPolicyTapped
        case labelTermsOfServiceTapped(String)
        
        // Checkbox actions (tracked)
        case checkboxAgreedToggled(Bool)
        case checkboxTermsAcceptedToggled
        
        // RadioButton actions (tracked)
        case radioButtonPaymentSelected(PaymentMethod)
        case radioButtonShippingSelected(String)
        
        // ListComponent actions (tracked)
        case listComponentItemTapped(ItemID)
        case listComponentItemSelected(index: Int, item: Item)
        
        // Non-tracked actions
        case viewAppeared
        case loadDataResponse(Result<[Item], Error>)
        case navigationPop
    }
    
    struct State {
        var isLoading = false
        var items: [Item] = []
    }
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            // Tracked button action
            case .buttonLoginTapped:
                // Framework automatically tracks this action
                state.isLoading = true
                return .run { send in
                    let result = try await loginService.authenticate()
                    await send(.loadDataResponse(.success([result])))
                }
                
            // Tracked button action with associated value
            case .buttonSubmitTapped(let formData):
                // Associated value provides implicit tracking data
                state.isLoading = true
                return .run { send in
                    let result = try await submitForm(formData)
                    await send(.loadDataResponse(.success(result)))
                }
                
            // Tracked label action
            case .labelPrivacyPolicyTapped:
                // Framework tracks this interaction
                return .run { send in
                    navigationService.openURL(privacyPolicyURL)
                }
                
            // Tracked checkbox action with associated value
            case .checkboxTermsAcceptedToggled:
                state.termsAccepted.toggle()
                // Framework implicitly logs the new state
                return .none
                
            // Tracked radio button action
            case .radioButtonPaymentSelected(let method):
                state.selectedPaymentMethod = method
                // Associated value provides selection context
                return .none
                
            // Tracked list component action
            case .listComponentItemTapped(let itemId):
                // Item ID becomes implicit tracking attribute
                return .run { send in
                    let item = state.items.first { $0.id == itemId }
                    navigationService.navigate(to: item)
                }
                
            // Non-tracked actions pass through without logging
            case .viewAppeared, .loadDataResponse, .navigationPop:
                return .none
            }
        }
    }
}

// Framework protocols applied
extension HomeReducer.Action: StateProtocol, WithAssociatedValue {
    // Automatic implementation
}
```

---

## Screen View Tracking with trackThisView

### Basic Screen View

```swift
struct HomeView: View {
    let store: StoreOf<HomeReducer>
    
    var body: some View {
        VStack {
            // Screen content
        }
        .onAppear {
            // Track screen view when view appears
            trackThisView(
                screenName: "home_screen",
                screenModule: "HomeModule"
            )
        }
    }
}
```

### Screen View with Tags

## Reducer with Analytics - Pattern 2: Effect-Based Logging

```swift
struct PaymentReducer: Reducer {
    @Dependency(\.analytics) var analytics
    @Dependency(\.paymentClient) var paymentClient
    
    enum Action {
        case submitPayment
        case paymentResponse(Result<PaymentResult, Error>)
    }
    
    struct State {
        var amount: Int = 0
    }
    
    func reduce(into state: inout State, action: Action) -> Effect<Action> {
        switch action {
        case .submitPayment:
            return .run { [amount = state.amount] send in
                // Log start
                self.analytics.track("payment_submitted", tags: [
                    "amount_cents": amount,
                    "timestamp": ISO8601DateFormatter().string(from: Date())
                ])
                
                do {
                    let result = try await self.paymentClient.process(amount: amount)
                    
                    // Log success
                    self.analytics.track("payment_success", tags: [
                        "transaction_id": result.id,
                        "amount_cents": amount
                    ])
                    
                    await send(.paymentResponse(.success(result)))
                } catch {
                    // Log failure
                    self.analytics.track("payment_failed", tags: [
                        "error_code": (error as NSError).code,
                        "error_description": error.localizedDescription,
                        "amount_cents": amount
                    ])
                    
                    await send(.paymentResponse(.failure(error)))
                }
            }
### Screen View with Tags

```swift
struct CheckoutView: View {
    let store: StoreOf<CheckoutReducer>
    
    var body: some View {
        VStack {
            // Checkout flow
        }
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
    }
}
```

### Multiple Screen Modules

```swift
// HomeModule
struct HomeView: View {
    var body: some View {
        VStack {}
            .onAppear {
                trackThisView(
                    screenName: "home_screen",
                    screenModule: "HomeModule",
                    tags: ["version": "2.1"]
                )
            }
    }
}

// ProfileModule
struct ProfileView: View {
    var body: some View {
        VStack {}
            .onAppear {
                trackThisView(
                    screenName: "profile",
                    screenModule: "ProfileModule",
                    tags: ["edit_mode": false]
                )
            }
    }
}

// PaymentModule
struct PaymentView: View {
    var body: some View {
        VStack {}
            .onAppear {
                trackThisView(
                    screenName: "payment",
                    screenModule: "PaymentModule",
                    tags: ["flow_type": "checkout"]
                )
            }
    }
}
```

---

## Testing Tracked Actions

### Test for Button Action

```swift
import XCTest
import ComposableArchitecture

final class HomeReducerTests: XCTestCase {
    
    func testButtonLoginTappedAction() async {
        let store = TestStore(
            initialState: HomeReducer.State(),
            reducer: { HomeReducer() }
        )
        
        // Send tracked action
        await store.send(.buttonLoginTapped) { state in
            state.isLoading = true
        }
    }
    
    func testButtonSubmitTappedWithAssociatedValue() async {
        let formData = FormData(email: "test@example.com", password: "pass")
        
        let store = TestStore(
            initialState: HomeReducer.State(),
            reducer: { HomeReducer() }
        )
        
        // Send action with associated value (provides implicit tracking context)
        await store.send(.buttonSubmitTapped(formData)) { state in
            state.isLoading = true
        }
    }
}
```

### Test for Label Action

```swift
func testLabelPrivacyPolicyTapped() async {
    let store = TestStore(
        initialState: HomeReducer.State(),
        reducer: { HomeReducer() }
    )
    
    // Label tap action (tracked)
    await store.send(.labelPrivacyPolicyTapped)
    
    // Verify navigation or state change
}
```

### Test for Checkbox Action

```swift
func testCheckboxAgreedToggled() async {
    let store = TestStore(
        initialState: HomeReducer.State(termsAccepted: false),
        reducer: { HomeReducer() }
    )
    
    // Toggle action (tracked with state change)
    await store.send(.checkboxAgreedToggled) { state in
        state.termsAccepted = true
    }
}
```

### Test for RadioButton Action

```swift
func testRadioButtonPaymentSelected() async {
    let method = PaymentMethod.creditCard
    
    let store = TestStore(
        initialState: HomeReducer.State(),
        reducer: { HomeReducer() }
    )
    
    // Selection action with associated value (provides selection context for tracking)
    await store.send(.radioButtonPaymentSelected(method)) { state in
        state.selectedPaymentMethod = method
    }
}
```

### Test for ListComponent Action

```swift
func testListComponentItemTapped() async {
    let itemId = "item_123"
    let initialState = HomeReducer.State(items: [
        Item(id: itemId, name: "Item 1")
    ])
    
    let store = TestStore(
        initialState: initialState,
        reducer: { HomeReducer() }
    )
    
    // List item tap (tracked with item ID context)
    await store.send(.listComponentItemTapped(itemId))
}
```

---

## Testing Screen View Tracking

### Verify trackThisView Calls

```swift
func testScreenViewTracking() {
    // Create a mock to capture trackThisView calls
    var capturedScreenNames: [String] = []
    var capturedModules: [String] = []
    
    // Mock trackThisView
    let mockTrackView: (String, String, [String: String]?) -> Void = { screen, module, tags in
        capturedScreenNames.append(screen)
        capturedModules.append(module)
    }
    
    // When HomeView appears
    let view = HomeView(store: testStore)
    view.onAppear { 
        mockTrackView("home_screen", "HomeModule", ["version": "2.1"])
    }
    
    // Verify tracking was called
    XCTAssertEqual(capturedScreenNames, ["home_screen"])
    XCTAssertEqual(capturedModules, ["HomeModule"])
}
```

---

## Extracting Associated Value Data for Tracking

```swift
// Define structure that contains tracking context
struct FormData: Equatable {
    let email: String
    let password: String
    let termsAccepted: Bool
}

// Action with associated value
case .buttonSubmitTapped(FormData)

// When action is sent, the FormData becomes tracking context:
// Implicit tracking attributes:
// - email: value
// - password: (redacted/hashed)
// - termsAccepted: true/false

// In reducer, extract for explicit tracking if needed
case .buttonSubmitTapped(let formData):
    // Associated value provides implicit tracking
    // Can also manually track if more detail needed:
    // analytics?.track(
    //     event: "form_submitted",
    //     tags: [
    //         "email_domain": extractDomain(formData.email),
    //         "terms_accepted": formData.termsAccepted
    //     ]
    // )
    return submitForm(formData)
```

---

## Action Naming Best Practices

```swift
enum Action: Equatable {
    // ✅ Good: Clear action name with tracking prefix
    case buttonLoginTapped
    case buttonSignupTapped(SignupData)
    case labelTermsClicked
    case checkboxMarketingToggled(Bool)
    case radioButtonShippingSelected(ShippingMethod)
    case listComponentItemTapped(ItemID)
    
    // ❌ Poor: Non-standard naming won't be tracked
    case buttonTap  // Too generic
    case submit     // Missing prefix
    case onClick    // Not a case name
    case button_login_tapped  // Wrong format (should be camelCase)
}

extension Action: StateProtocol, WithAssociatedValue {
    // Framework provides implementation
}
```

---

## Complete Feature Example

```swift
struct OnboardingReducer: Reducer {
    enum Action: Equatable {
        // Tracked actions only
        case buttonNextTapped
        case buttonSkipTapped
        case checkboxTermsAcceptedToggled(Bool)
        case radioButtonMethodSelected(OnboardingMethod)
        
        // Non-tracked actions
        case viewAppeared
        case stepCompleted(Int)
    }
    
    struct State {
        var currentStep: Int = 0
        var termsAccepted: Bool = false
        var selectedMethod: OnboardingMethod = .standard
    }
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            // Tracked: button action
            case .buttonNextTapped:
                state.currentStep += 1
                return .none
                
            // Tracked: button action
            case .buttonSkipTapped:
                // Skip is tracked with implicit step context
                return .none
                
            // Tracked: checkbox toggle with boolean state
            case .checkboxTermsAcceptedToggled(let isAccepted):
                state.termsAccepted = isAccepted
                // Boolean becomes tracking attribute
                return .none
                
            // Tracked: radio button selection with method
            case .radioButtonMethodSelected(let method):
                state.selectedMethod = method
                // Selected method becomes tracking attribute
                return .none
                
            case .viewAppeared, .stepCompleted:
                return .none
            }
        }
    }
}

extension OnboardingReducer.Action: StateProtocol, WithAssociatedValue {
}

struct OnboardingView: View {
    let store: StoreOf<OnboardingReducer>
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Step \(store.currentStep)")
            
            // Tracked button
            Button("Next") {
                store.send(.buttonNextTapped)
            }
            
            // Tracked button
            Button("Skip") {
                store.send(.buttonSkipTapped)
            }
            
            // Tracked checkbox
            Toggle("I accept terms", isOn: Binding(
                get: { store.termsAccepted },
                set: { store.send(.checkboxTermsAcceptedToggled($0)) }
            ))
            
            // Tracked radio buttons
            Picker("Method", selection: Binding(
                get: { store.selectedMethod },
                set: { store.send(.radioButtonMethodSelected($0)) }
            )) {
                Text("Standard").tag(OnboardingMethod.standard)
                Text("Premium").tag(OnboardingMethod.premium)
            }
        }
        .onAppear {
            store.send(.viewAppeared)
            
            // Screen view tracking
            trackThisView(
                screenName: "onboarding_step_\(store.currentStep)",
                screenModule: "OnboardingModule",
                tags: [
                    "step": "\(store.currentStep)",
                    "total_steps": "5"
                ]
            )
        }
    }
}
```

---

## Data Extraction Guidelines

When auditing, extract:

```
FROM ACTION ENUM:
✅ .button* cases → extract as "button" actions
✅ .label* cases → extract as "label" actions
✅ .checkbox* cases → extract as "checkbox" actions
✅ .radioButton* cases → extract as "radio_button" actions
✅ .listComponent* cases → extract as "list_component" actions
✅ Associated values: (paramName: Type) → extract parameter info

FROM VIEW:
✅ trackThisView calls → extract screenName, screenModule, tags
✅ Call location (file, view name)
✅ All tag key-value pairs

DON'T EXTRACT:
❌ Non-prefixed actions (.viewAppeared, .loadData)
❌ Direct analytics.track() calls (rely on action framework)
❌ Navigation or data actions
```
            initialState: PaymentReducer.State(amount: 5000),
            reducer: { PaymentReducer() }
        )
        
        store.dependencies.analytics = analytics
        store.dependencies.paymentClient = paymentClient
        
        await store.send(.submitPayment)
        await store.finish()
        
        // Verify error event
        XCTAssertTrue(analytics.hasRecorded("payment_failed"))
    }
}
```

## Checklist for Framework Setup

When implementing analytics with StateProtocol and WithAssociatedValue:

- [ ] Define Action enum with StateProtocol, WithAssociatedValue conformance
- [ ] Use tracking prefixes: button*, label*, checkbox*, radioButton*, listComponent*
- [ ] Add trackThisView calls in .onAppear for each screen
- [ ] Include associated values for tracking context
- [ ] Test tracked actions fire correctly
- [ ] Verify trackThisView calls with correct parameters
- [ ] Validate tag consistency across screens
- [ ] Document action naming conventions for team
