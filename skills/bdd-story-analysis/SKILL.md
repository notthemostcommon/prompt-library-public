---
name: bdd-story-analysis
description: BDD story analysis and test planning for converting user stories into behavior-driven acceptance criteria with testing pyramid strategy
---

# BDD Story Analysis & Test Planning

This rule guides the transformation of user stories into Behavior-Driven Development (BDD) scenarios with appropriate test distribution following the testing pyramid.

> **Framework-Specific Implementation**: This rule defines *what* to test (scenarios) and *where* to test (pyramid level). For *how* to implement tests in React/Next.js, see `.cursor/rules/frontend-testing.mdc`.

## 1. Core Principles

### 1.1 BDD Philosophy

BDD tests describe **what** the system does, not **how** it does it:

| Principle | Description |
|-----------|-------------|
| **Behavior over implementation** | Tests verify outcomes visible to users, not internal mechanics |
| **Ubiquitous language** | Use business domain terms, not technical jargon |
| **Living documentation** | Scenarios serve as executable specifications |
| **Test-first compatible** | Scenarios can be written before any code exists |
| **Three Amigos** | Business, Dev, and QA collaborate on scenarios |

### 1.2 Implementation Decoupling Rules

Tests MUST be written so they remain valid regardless of implementation approach:

```gherkin
# ❌ BAD - Coupled to implementation (references framework internals)
Given the Redux store has user state
When the useAuth hook is called
Then the AuthContext should update

# ❌ BAD - Coupled to implementation (references code structure)
Given the UserService class is instantiated
When the validateCredentials method is called
Then it should return a User object

# ✅ GOOD - Behavior-focused (any framework could implement this)
Given I am logged in as "user@example.com"
When I navigate to the dashboard
Then I should see my account name displayed
```

**Decoupling Checklist:**
- [ ] No references to specific components, hooks, classes, or functions
- [ ] No references to state management libraries or patterns
- [ ] No references to CSS classes, DOM structure, or selectors
- [ ] No references to internal data formats or transformations
- [ ] No references to framework-specific concepts (hooks, controllers, services)
- [ ] Scenarios describe user-visible behavior or system outputs only

---

## 2. Story Analysis Process

### 2.1 Extract Behaviors from Stories

When analyzing a user story, identify:

| Element | Question to Ask | Examples |
|---------|-----------------|----------|
| **Actors** | Who uses this feature? | Customer, Admin, Guest, API consumer |
| **Actions** | What can they do? | View, submit, edit, delete, search, filter |
| **Outcomes** | What should happen? | See confirmation, data persists, state changes |
| **Constraints** | What limits exist? | Character limits, required fields, permissions |
| **Edge cases** | What could go wrong? | Empty data, boundary values, concurrent access |

### 2.2 Scenario Discovery Template

For each acceptance criterion, answer:

```markdown
## Behavior: [Name of behavior]

**Actor**: Who performs this action?
**Trigger**: What initiates this behavior?
**Preconditions**: What must be true before?
**Action**: What does the user do?
**Outcome**: What should the user observe?
**Variations**: What alternatives exist?
**Edge cases**: What boundaries should be tested?
```

### 2.3 Scenario Types

| Type | When to Use | Example |
|------|-------------|---------|
| **Happy path** | Primary success flow | User submits valid form |
| **Alternate path** | Valid but different flow | User cancels mid-process |
| **Edge case** | Boundary conditions | Input at exact character limit |
| **Error path** | Expected failure handling | Invalid input rejected |
| **Negative test** | Security/validation | Unauthorized access blocked |

---

## 3. BDD Scenario Structure

### 3.1 Feature File Format

```gherkin
Feature: [Feature name matching story title]
  [User story in "As a... I want... So that..." format]

  Background:
    [Common preconditions for all scenarios in this feature]

  @tag1 @tag2
  Scenario: [Descriptive name of specific behavior]
    Given [precondition - system state before action]
    And [additional precondition if needed]
    When [action - what the user does]
    And [additional action if needed]
    Then [outcome - observable result]
    And [additional outcome if needed]
```

### 3.2 Writing Effective Given-When-Then

**Given** - Establish context (past tense or present state):
```gherkin
# ✅ Good - describes observable state
Given I am on the product listing page
Given a product exists with price $99.99
Given the user is logged in as "admin@example.com"
Given 3 items are in my shopping cart

# ❌ Bad - describes action or is vague
Given I click the button
Given the system works
Given the database has records
```

**When** - Single action (present tense):
```gherkin
# ✅ Good - single clear action
When I submit the form
When I click the "Add to Cart" button
When I enter "laptop" in the search field
When the page finishes loading

# ❌ Bad - multiple actions or vague
When I do stuff
When I fill out the form and submit it and wait
When the API is called
```

**Then** - Observable outcome (should/should see):
```gherkin
# ✅ Good - observable by user
Then I should see "Order confirmed" message
Then the product count should show "4"
Then the submit button should be disabled
Then I should be redirected to the confirmation page

# ❌ Bad - implementation detail or not observable
Then the database should have a new record
Then the Redux state should update
Then the validateOrder function should return true
Then the OrderService.create method should be called
```

### 3.3 Parameterized Scenarios (Scenario Outline)

Use for testing multiple variations of the same behavior:

```gherkin
Scenario Outline: Product discount calculation
  Given a product with price <original_price>
  And a <discount_type> discount of <discount_value>
  When the discount is applied
  Then the final price should be <final_price>

  Examples:
    | original_price | discount_type | discount_value | final_price |
    | $100.00        | percentage    | 10%            | $90.00      |
    | $100.00        | percentage    | 25%            | $75.00      |
    | $100.00        | fixed         | $15.00         | $85.00      |
    | $50.00         | percentage    | 50%            | $25.00      |
```

```gherkin
Scenario Outline: Content truncation based on length
  Given content with <length> characters
  When the content is displayed
  Then I should see <display_behavior>
  And the "Read more" link should be <visibility>

  Examples:
    | length | display_behavior                | visibility |
    | 48     | the full content                | hidden     |
    | 250    | the full content                | hidden     |
    | 251    | truncated text at word boundary | visible    |
    | 500    | truncated text at word boundary | visible    |
```

### 3.4 Tagging Strategy

Apply tags for test filtering and organization:

| Tag Category | Examples | Purpose |
|--------------|----------|---------|
| **Actor** | `@customer`, `@admin`, `@guest` | Filter by user type |
| **Feature area** | `@search`, `@checkout`, `@auth`, `@validation` | Filter by functionality |
| **Test type** | `@happy-path`, `@edge-case`, `@error-path` | Filter by scenario type |
| **Priority** | `@critical`, `@smoke`, `@regression` | Filter for CI/CD pipelines |
| **Test level** | `@unit`, `@integration`, `@e2e` | Map to testing pyramid |
| **Sprint/Release** | `@sprint-12`, `@v2.0` | Filter by delivery scope |

---

## 4. Testing Pyramid Strategy

### 4.1 The Testing Pyramid

```
        /\
       /  \        E2E Tests (5-10%)
      /----\       - Critical user journeys only
     /      \      - Expensive, slow, flaky
    /--------\
   /          \    Integration Tests (20-30%)
  /            \   - Component interactions
 /--------------\  - API boundaries
/                \ - User-facing behavior
------------------
                   Unit Tests (60-70%)
                   - Pure functions
                   - Isolated logic
                   - Fast, cheap, stable
```

### 4.2 Test Level Decision Matrix

For each BDD scenario, determine the appropriate test level:

| Scenario Characteristic | Test Level | Rationale |
|------------------------|------------|-----------|
| Pure data transformation | **Unit** | No dependencies, fast |
| Single component rendering | **Unit** | Isolated, mockable |
| User interaction within one component | **Integration** | Tests realistic flow |
| Multiple components communicating | **Integration** | Tests integration points |
| API request/response handling | **Integration** | Tests boundaries |
| Full page with real API | **E2E** | Tests full stack |
| Cross-page navigation flow | **E2E** | Tests user journey |
| Authentication flows | **E2E** | Security-critical |

### 4.3 Distribution Guidelines

**Unit Tests** - Test in isolation (60-70% of test effort):
- Pure functions (formatting, validation, calculation, transformation)
- Business logic with no external dependencies
- Isolated module behavior
- Data parsing and serialization

**Integration Tests** - Test interactions (20-30% of test effort):
- Component/module interactions with user events
- API boundary testing (with mocked external services)
- State management flows
- Form submission with validation
- Multi-component coordination

**E2E Tests** - Test critical paths only (5-10% of test effort):
- Complete user journeys (signup → first action → outcome)
- Payment/checkout flows
- Authentication (login, logout, session management)
- Cross-browser/device compatibility
- Flows that span multiple pages/screens

### 4.4 Test Level Selection Flowchart

```
Is the behavior a pure data transformation?
├── YES → Unit Test
└── NO ↓

Does it require user interaction with UI?
├── YES ↓
│   Is it contained within a single component/module?
│   ├── YES → Integration Test
│   └── NO ↓
│       Does it span multiple pages or require real backend?
│       ├── YES → E2E Test (if critical path)
│       └── NO → Integration Test
└── NO ↓

Does it test an API/service boundary?
├── YES → Integration Test (with mocked dependencies)
└── NO → Unit Test
```

---

## 5. Applying BDD to a Story

### 5.1 Analysis Output

**IMPORTANT**: When performing BDD analysis, **append the results directly to the story.md file**. Do not create separate files or output to chat only.

The BDD analysis becomes part of the story's living documentation, ensuring:
- Scenarios evolve with the story
- Test distribution is visible to all team members
- Single source of truth for acceptance criteria

When analyzing a story, append the following sections to the end of `story.md`:

```markdown
---

## BDD Analysis: [Story Title]

### Actors & Personas
- [Actor 1]: [Description and goals]
- [Actor 2]: [Description and goals]

### Behavior Summary
| Behavior | Actor | Trigger | Outcome |
|----------|-------|---------|---------|
| [Name] | [Who] | [Action] | [Result] |

### BDD Scenarios

#### Feature: [Feature Name]

[Gherkin scenarios here]

### Test Distribution

#### Unit Tests
| Scenario/Behavior | What to Test | Why Unit |
|-------------------|--------------|----------|
| [Behavior] | [Specific test] | [Rationale] |

#### Integration Tests
| Scenario/Behavior | What to Test | Why Integration |
|-------------------|--------------|-----------------|
| [Behavior] | [Specific test] | [Rationale] |

#### E2E Tests
| Scenario/Behavior | What to Test | Why E2E |
|-------------------|--------------|---------|
| [Behavior] | [Specific test] | [Rationale] |

### Test-First Implementation Notes
[Guidance on what tests to write before implementation]
```

### 5.2 Example: Applying to a Text Truncation Feature

**Story Acceptance Criterion:**
> "Display the first 250 characters of content with '… Read More' link"

**BDD Analysis:**

```markdown
### Behavior Summary
| Behavior | Actor | Trigger | Outcome |
|----------|-------|---------|---------|
| View truncated content | User | Page load | See max 250 chars with "Read more" |
| View full short content | User | Page load | See full text, no "Read more" |

### BDD Scenarios

@truncation @unit
Scenario: Short content displays in full
  Given content with 48 characters
  When the content is displayed
  Then I should see the complete text
  And no "Read more" indicator should appear

@truncation @unit  
Scenario: Long content is truncated at word boundary
  Given content with 280 characters
  When the content is displayed
  Then I should see text ending at a complete word
  And "... Read more" should be appended
  And total displayed length should not exceed 250 characters

@truncation @edge-case
Scenario: Content at exactly the limit displays without truncation
  Given content with exactly 250 characters
  When the content is displayed
  Then I should see the complete text
  And no "Read more" indicator should appear

### Test Distribution

#### Unit Tests
| Scenario | Test | Rationale |
|----------|------|-----------|
| Truncation logic | Pure truncation function | No dependencies, pure input→output |
| Word boundary detection | Edge cases for word breaks | Isolated string manipulation |
| Boundary values | Exact limit (250, 251 chars) | Boundary value analysis |

#### Integration Tests  
| Scenario | Test | Rationale |
|----------|------|-----------|
| Content card rendering | Component displays truncated text | Tests UI + utility integration |
| Read more visibility | Conditional rendering based on length | Tests UI state logic |

#### E2E Tests
| Scenario | Test | Rationale |
|----------|------|-----------|
| (None for truncation) | - | Pure UI logic, no system integration needed |
```

### 5.3 Example: Applying to a Search Feature

**Story Acceptance Criteria:**
> "User can search products by name and see matching results"

**BDD Analysis:**

```markdown
### Behavior Summary
| Behavior | Actor | Trigger | Outcome |
|----------|-------|---------|---------|
| Search products | Customer | Enter search term | See matching results |
| No results | Customer | Search for non-existent item | See "no results" message |
| Clear search | Customer | Clear search field | See all products again |

### BDD Scenarios

@search @integration
Scenario: User finds products by name
  Given products exist in the catalog
  When I search for "laptop"
  Then I should see products with "laptop" in the name
  And I should not see products without "laptop" in the name

@search @integration
Scenario: User sees message when no products match
  Given products exist in the catalog
  When I search for "xyznonexistent"
  Then I should see a "No products found" message
  And I should see a suggestion to try different keywords

@search @integration
Scenario: User clears search to see all products
  Given I have searched for "laptop"
  And I see filtered results
  When I clear the search field
  Then I should see all products in the catalog

### Test Distribution

#### Unit Tests
| Scenario | Test | Rationale |
|----------|------|-----------|
| Search matching logic | Filter function with various inputs | Pure function, no dependencies |
| Case insensitivity | "Laptop" matches "laptop" | Isolated string logic |
| Partial matching | "lap" matches "laptop" | Isolated string logic |

#### Integration Tests
| Scenario | Test | Rationale |
|----------|------|-----------|
| Search input + results | Component updates results on input | Tests user interaction flow |
| Empty state rendering | "No results" UI appears correctly | Tests conditional UI |
| API integration | Search calls API with correct params | Tests component + API boundary |

#### E2E Tests
| Scenario | Test | Rationale |
|----------|------|-----------|
| Full search journey | Search → results → clear → all | Critical user flow, tests full stack |
```

---

## 6. Test-First Development Flow

### 6.1 Writing Tests Before Implementation

BDD scenarios enable test-first development (Red-Green-Refactor):

1. **Write the scenario** (business language - framework agnostic)
2. **Create failing test** (maps scenario to test code in your framework)
3. **Implement minimum code** (make test pass)
4. **Refactor** (improve without changing behavior)
5. **Repeat** for next scenario

### 6.2 Scenario-to-Test Mapping (Conceptual)

The BDD scenario defines **WHAT** to test. The test implementation defines **HOW**.

```gherkin
# BDD Scenario (framework-agnostic)
Scenario: Long content is truncated at word boundary
  Given content with 280 characters
  When the content is displayed
  Then I should see text ending at a complete word
  And "... Read more" should be appended
```

**Unit Test Pattern** (language-agnostic pseudocode):
```
TEST "truncates long content at word boundary with suffix"
  // Arrange
  input = "A" repeated 50 times + " " + "B" repeated 230 times  // 281 chars
  
  // Act
  result = truncate(input)
  
  // Assert
  VERIFY result ends with "... Read more"
  VERIFY result.length <= 250
  VERIFY result does not end with " ... Read more"  // No trailing space
```

**Integration Test Pattern** (language-agnostic pseudocode):
```
TEST "displays truncated content with Read more indicator"
  // Arrange
  content = { title: "Example", body: "A" repeated 300 times }
  
  // Act  
  render ContentCard with content
  
  // Assert
  VERIFY user can see "... Read more" text
  VERIFY user can see the title
```

> **React/Next.js Implementation**: See `.cursor/rules/frontend-testing.mdc` for:
> - Section 3.1: Component rendering patterns
> - Section 3.2: Query selection (getByRole, getByText, etc.)
> - Section 3.3: User interaction with userEvent
> - Section 7.1: AAA pattern with specific syntax

### 6.3 What NOT to Specify in Tests

Keep tests implementation-agnostic by testing **observable outcomes**:

```
❌ DON'T test implementation details:
- Internal state values (component.state.isTruncated)
- Method/function calls (useTruncation was called)
- CSS classes or DOM structure (.truncated-text exists)
- Framework internals (hook was invoked)

✅ DO test observable behavior:
- User can see the "... Read more" text
- User cannot see the full original text
- Clicking button shows confirmation message
- Form submission displays success state
```

### 6.4 Test Independence Principle

Tests written from BDD scenarios should remain valid even if:
- You refactor component structure
- You change state management approach
- You rename internal functions
- You switch CSS frameworks
- You update framework versions

**If a refactor breaks your test but not the behavior, your test is coupled to implementation.**

---

## 7. Checklist for Story Analysis

Before marking BDD analysis complete, verify:

### Scenario Coverage
- [ ] All acceptance criteria have corresponding scenarios
- [ ] Happy path scenarios exist for each primary behavior
- [ ] Edge cases identified (boundaries, empty states, limits)
- [ ] Error paths covered where applicable
- [ ] Scenarios use business language, not technical terms

### Test Distribution
- [ ] Each scenario assigned to appropriate test level
- [ ] Unit tests cover isolated logic (60-70% of test effort)
- [ ] Integration tests cover component interactions (20-30%)
- [ ] E2E tests limited to critical user journeys (5-10%)
- [ ] No redundant testing across levels

### Implementation Independence
- [ ] No scenarios reference specific components or functions
- [ ] No scenarios reference state management approach
- [ ] No scenarios reference CSS or DOM structure
- [ ] Tests could pass with any valid implementation
- [ ] Scenarios focus on user-observable outcomes

### Test-First Readiness
- [ ] Tests can be written without seeing implementation
- [ ] Assertions based on inputs and outputs, not internals
- [ ] Mock boundaries clearly identified (APIs, external services)

---

## 8. Quick Reference

### Scenario Writing Checklist

```
✓ Feature describes the capability
✓ Scenario name describes the specific behavior
✓ Given establishes preconditions (state, not actions)
✓ When describes a single user action
✓ Then describes observable outcomes
✓ Tags applied for filtering (@actor, @feature, @type, @level)
✓ No implementation details in any step
```

### Test Level Quick Decision

```
Is it a pure function with no dependencies?
  → Unit Test

Does it involve user interaction with UI?
  → Integration Test

Does it span multiple pages or require real backend?
  → E2E Test (only if critical path)
```

### Anti-Patterns to Avoid

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Ice cream cone | Too many E2E, few unit | Invert pyramid |
| Testing implementation | Brittle tests | Test behavior |
| Duplicate coverage | Same test at multiple levels | Choose one level |
| Scenario explosion | Too many scenarios | Use Scenario Outline |
| Vague assertions | "System works correctly" | Specific observable outcomes |
| Framework coupling in scenarios | Scenarios break on refactor | Use business language only |

---

## 9. Framework-Specific Implementation

This BDD rule defines **what** to test and **where** (pyramid level). For **how** to implement tests, refer to framework-specific testing rules.

### 9.1 React / Next.js Projects

**Reference**: `.cursor/rules/frontend-testing.mdc`

| BDD Concept | Implementation Reference |
|-------------|-------------------------|
| Integration test queries | Section 3.2: Query Selection Guide (getByRole, getByText hierarchy) |
| User interactions | Section 3.3: userEvent library patterns |
| Async behavior testing | Section 3.4: find*, waitFor patterns |
| Server component testing | Section 4.1: Next.js App Router patterns |
| Client component testing | Section 4.2: State and effects testing |
| Form testing | Section 4.3: Submission and validation |
| API mocking | Section 5.1: MSW (Mock Service Worker) setup |
| E2E tests | Section 5.2: Playwright patterns |
| Accessibility assertions | Section 6.1: jest-axe integration |
| Test structure | Section 7.1: AAA pattern with specific syntax |

**Key Testing Library Principles** (from frontend-testing.mdc):
- Use `screen` for all queries
- Use `userEvent` over `fireEvent` for user interactions
- Use `getBy*` for elements that should exist, `queryBy*` only for non-existence
- Use `findBy*` for async elements
- Query by accessibility attributes (role, label) first

### 9.2 Other Frameworks

For non-React projects, apply the same BDD principles with your framework's testing tools:

| Framework | Unit Testing | Integration Testing | E2E Testing |
|-----------|--------------|---------------------|-------------|
| **Vue.js** | Vitest | Vue Testing Library | Playwright/Cypress |
| **Angular** | Jasmine/Jest | Angular Testing Library | Playwright/Cypress |
| **Svelte** | Vitest | Svelte Testing Library | Playwright |
| **Backend (Node)** | Jest/Vitest | Supertest | Playwright |
| **Backend (Python)** | pytest | pytest + httpx | pytest + Playwright |
| **Backend (.NET)** | xUnit/NUnit | Integration test fixtures | Playwright |

**Universal Principles**:
1. Query by user-visible attributes, not implementation details
2. Simulate real user interactions
3. Assert on observable outcomes
4. Mock external dependencies at integration boundaries
5. Reserve E2E for critical paths only

---

## 10. BDD Analysis Output Checklist

When completing a BDD analysis for a story:

1. **Append results to the story.md file** (do not create separate files)
2. Add a horizontal rule (`---`) to separate from existing content
3. Include all sections below

```markdown
---

## BDD Analysis: [Story Title]

### ✅ Actors & Personas
- [ ] All user types identified
- [ ] Goals and motivations documented

### ✅ Behavior Summary Table
- [ ] All acceptance criteria mapped to behaviors
- [ ] Triggers and outcomes clearly stated

### ✅ BDD Scenarios (Gherkin)
- [ ] Happy path scenarios for each behavior
- [ ] Edge cases identified (boundaries, empty states)
- [ ] Error paths covered (if in scope)
- [ ] Scenarios use business language (no implementation terms)
- [ ] Appropriate tags applied (@actor, @feature, @level)

### ✅ Test Distribution
- [ ] Each scenario assigned to unit/integration/E2E
- [ ] Rationale provided for each assignment
- [ ] Pyramid proportions respected (60-70% unit, 20-30% integration, 5-10% E2E)
- [ ] No duplicate coverage across levels

### ✅ Test-First Readiness
- [ ] Tests can be written without implementation code
- [ ] Assertions focus on inputs/outputs only
- [ ] Mock boundaries identified (APIs, external services)
```

**File Location**: `specifications/[story-folder]/story.md`
