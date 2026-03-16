---
name: edge-case-heuristics
description: Edge case testing heuristics for validating story analysis completeness based on Ministry of Testing Test Heuristics Cheat Sheet
---

# Edge Case Testing Heuristics

This rule provides systematic heuristics for validating that BDD story analysis has comprehensive edge case coverage. Use this as a checklist after completing initial BDD analysis to identify gaps.

> **Source**: Adapted from [Ministry of Testing - Test Heuristics Cheat Sheet](https://www.ministryoftesting.com/articles/test-heuristics-cheat-sheet)
> **Companion Skill**: Use with `skills/bdd-story-analysis/SKILL.md` for complete story analysis

---

## 1. Data Type Attack Vectors

For each data input in your feature, systematically consider these attack vectors:

### 1.1 String Inputs

| Heuristic | Test Scenario | Example |
|-----------|---------------|---------|
| **Long strings** | 255, 256, 257, 1000, 1024, 2000, 2048+ chars | Name field with 1000 characters |
| **Empty/blank** | Empty string, single space, multiple spaces | Submit form with blank required field |
| **Leading/trailing spaces** | " value", "value ", " value " | Search with leading spaces |
| **Special characters** | `" ' \` | / \ , ; : & < > ^ * ?` | Name with apostrophe: "O'Brien" |
| **Delimiters** | Tab characters, newlines, carriage returns | Multiline input in single-line field |
| **Unicode/Accented** | àáâãäåçèéêëìíîðñòôõöö | International customer names |
| **Asian characters** | 漢字, emoji 🎉 | CJK character support |
| **SQL injection** | `'; DROP TABLE users; --` | Search field with SQL syntax |
| **HTML/Script injection** | `<script>alert('xss')</script>` | Comment field with HTML |

**Edge Case Checklist:**
- [ ] What happens with empty input?
- [ ] What happens at maximum length?
- [ ] What happens with special characters in the domain context?
- [ ] What happens with international characters?
- [ ] What happens with copy/pasted text vs typed text?

### 1.2 Numeric Inputs

| Heuristic | Test Values | Rationale |
|-----------|-------------|-----------|
| **Zero** | 0 | Division, empty collection, null quantity |
| **Negative** | -1, -0.001 | Invalid quantities, dates before epoch |
| **Boundaries (16-bit)** | 32768, 32769, 65536, 65537 | Integer overflow |
| **Boundaries (32-bit)** | 2147483648, 2147483649, 4294967296 | Large number handling |
| **Floating point** | 0.0001, 0.1 + 0.2 | Precision issues |
| **Scientific notation** | 1E-16, 1E+10 | Parsing edge cases |
| **Formatted numbers** | 1,234,567 vs 1.234.567,89 | Locale-specific formats |

**Edge Case Checklist:**
- [ ] What happens with zero?
- [ ] What happens with negative numbers (if not allowed)?
- [ ] What happens at boundary values (min/max)?
- [ ] What happens with decimal precision beyond expected?
- [ ] What happens with locale-specific number formats?

### 1.3 Date and Time Inputs

| Heuristic | Test Scenario | Example |
|-----------|---------------|---------|
| **Invalid dates** | Feb 30, Sept 31 | Calendar selection edge |
| **Leap year** | Feb 29 in non-leap year | 2023-02-29 |
| **Format variations** | DD/MM/YYYY vs MM/DD/YYYY | 06/05/2024 ambiguity |
| **Timezone crossing** | UTC vs local, DST changeover | Event at midnight UTC |
| **Time boundaries** | 23:59:59 → 00:00:00 | Day rollover |
| **Past/future limits** | Very old dates, far future | DOB of 1800-01-01 |
| **Timeouts** | Long-running operations | Session expires during action |

**Edge Case Checklist:**
- [ ] What happens at date boundaries (month-end, year-end)?
- [ ] What happens with timezone differences?
- [ ] What happens during daylight savings transition?
- [ ] What happens with dates far in the past or future?
- [ ] What happens if operation takes longer than expected?

### 1.4 File Inputs

| Heuristic | Test Scenario | Example |
|-----------|---------------|---------|
| **Long filename** | >255 characters | Very long filename upload |
| **Special chars in name** | `file name (1).pdf` | Spaces, parentheses, brackets |
| **No extension** | `document` | File without extension |
| **Wrong extension** | `image.pdf` (actually JPEG) | Mismatched type |
| **Zero-byte file** | Empty file | Upload empty document |
| **Large file** | Beyond limit | 100MB upload with 10MB limit |
| **Corrupted file** | Invalid structure | Truncated or damaged file |

**Edge Case Checklist:**
- [ ] What happens with unsupported file types?
- [ ] What happens at size limits?
- [ ] What happens with empty files?
- [ ] What happens with corrupted files?
- [ ] What happens with special characters in filenames?

---

## 2. Behavioral Heuristics

### 2.1 CRUD Operations

For any data entity, ensure all operations are covered:

| Operation | Edge Cases to Consider |
|-----------|----------------------|
| **Create** | Duplicate entry, maximum count reached, required fields missing |
| **Read** | Non-existent record, deleted record, permission denied |
| **Update** | Concurrent updates, optimistic locking, stale data |
| **Delete** | Delete with dependencies, soft vs hard delete, undo capability |

**Edge Case Checklist:**
- [ ] What happens creating duplicate records?
- [ ] What happens reading non-existent data?
- [ ] What happens with concurrent updates?
- [ ] What happens deleting records with dependencies?

### 2.2 Goldilocks Principle

Test values at three key points:

| Point | Description | Example |
|-------|-------------|---------|
| **Too Small** | Below minimum, zero, negative | 0 items in cart |
| **Just Right** | Valid middle value | 5 items in cart |
| **Too Big** | Above maximum, boundary | 1000 items in cart |

### 2.3 Boundaries

| Position | Test At | Example (limit: 250 chars) |
|----------|---------|---------------------------|
| **Below boundary** | n-1 | 249 characters |
| **At boundary** | n | 250 characters |
| **Above boundary** | n+1 | 251 characters |

**Edge Case Checklist:**
- [ ] Have you tested one below the limit?
- [ ] Have you tested exactly at the limit?
- [ ] Have you tested one above the limit?

### 2.4 Count Heuristic (Zero, One, Many)

| Count | Test Scenario | Example |
|-------|---------------|---------|
| **Zero** | Empty collection, no items | Search returns 0 results |
| **One** | Single item, singular vs plural | 1 item in cart |
| **Many** | Multiple items, pagination | 1000+ search results |

**Edge Case Checklist:**
- [ ] What does the UI show with zero items?
- [ ] Is singular/plural grammar correct for one item?
- [ ] Does pagination/scrolling work with many items?

### 2.5 State Transitions

| Heuristic | Test Scenario |
|-----------|---------------|
| **Happy path** | Normal state progression |
| **Skip states** | Jump from state A to state C |
| **Reverse states** | Go backward in workflow |
| **Repeat states** | Enter same state multiple times |
| **Invalid transitions** | Attempt forbidden state change |

**Edge Case Checklist:**
- [ ] What happens if user navigates backward?
- [ ] What happens if state is changed externally?
- [ ] What happens with concurrent state changes?

### 2.6 Conditional Logic & Fallback Chains

| Heuristic | Test Scenario | Example |
|-----------|---------------|---------|
| **OR conditions** | Test each operand failing independently | `!code \|\| !state` → test missing code, test missing state |
| **AND conditions** | Test each operand true/false independently | `isAdmin && isActive` → test admin+inactive, non-admin+active |
| **Fallback chains** | Test each fallback step activating | `name \|\| username \|\| email \|\| 'Default'` → test with only name, only username, only email, none |
| **Optional chaining** | Test null at each level | `session.auth?.user?.id` → test null auth, test null user |
| **Ternary cascades** | Test each condition branch | Nested ternaries need each path exercised |

**Edge Case Checklist:**
- [ ] For each `||` / `&&`, is every operand tested as the deciding factor?
- [ ] For fallback chains, does a test exist where each fallback is the one that provides the value?
- [ ] For optional chaining, is there a test with null/undefined at each level?
- [ ] For components with contextual dependencies (pathname, locale, role), are non-default values tested?

---

## 3. Web-Specific Heuristics

### 3.1 Navigation

| Heuristic | Test Scenario |
|-----------|---------------|
| **Back button** | Press back after form submission (double-post?) |
| **Refresh** | F5 on confirmation page |
| **Bookmark** | Bookmark mid-flow, return later |
| **URL manipulation** | Change query params, remove IDs |
| **Multiple tabs** | Same user, same action in two tabs |
| **Deep linking** | Direct URL to page requiring auth |

**Edge Case Checklist:**
- [ ] What happens if user presses back after submit?
- [ ] What happens if user refreshes a transactional page?
- [ ] What happens if user bookmarks and returns when logged out?
- [ ] What happens with URL parameter manipulation?

### 3.2 Interruptions

| Heuristic | Test Scenario |
|-----------|---------------|
| **Network loss** | Disconnect during operation |
| **Timeout** | Session expires mid-action |
| **Cancel** | Cancel long-running operation |
| **Close tab** | Close browser during action |
| **Device sleep** | Mobile goes to sleep mid-flow |

**Edge Case Checklist:**
- [ ] What happens if network is lost during operation?
- [ ] What happens if session expires mid-flow?
- [ ] Is there proper error handling for timeouts?
- [ ] Can users safely cancel operations?

### 3.3 Input Methods

| Method | Considerations |
|--------|----------------|
| **Typing** | Character-by-character, autocomplete |
| **Copy/Paste** | Formatted text, hidden characters |
| **Drag/Drop** | File drops, text drops |
| **Import** | CSV, JSON, bulk data |
| **API** | Direct API vs UI behavior differences |

**Edge Case Checklist:**
- [ ] Does copy/paste behave the same as typing?
- [ ] What happens with drag/drop operations?
- [ ] Does API input have same validation as UI?

---

## 4. API-Specific Heuristics

### 4.1 BINMEN Mnemonic

| Letter | Heuristic | Test Scenario |
|--------|-----------|---------------|
| **B** | Boundary | Min/max values, length limits |
| **I** | Invalid | Malformed JSON, wrong types |
| **N** | NULL | Null values, missing fields |
| **M** | Method | Wrong HTTP verb (GET vs POST) |
| **E** | Empty | Empty arrays, empty objects |
| **N** | Negative | Negative IDs, negative quantities |

### 4.2 POISED Mnemonic

| Letter | Heuristic | Test Scenario |
|--------|-----------|---------------|
| **P** | Parameters | Missing, extra, malformed params |
| **O** | Output | Correct format, pagination, sorting |
| **I** | Interop | Integration with other services |
| **S** | Security | Auth, permissions, rate limits |
| **E** | Errors | Error codes, messages, recovery |
| **D** | Data | Validation, persistence, consistency |

### 4.3 VADER Mnemonic

| Letter | Heuristic | Test Scenario |
|--------|-----------|---------------|
| **V** | Verbs | All HTTP methods tested |
| **A** | Auth | Authentication and authorization |
| **D** | Data | Request/response data handling |
| **E** | Errors | Error scenarios and responses |
| **R** | Responsiveness | Performance under load |

**API Edge Case Checklist:**
- [ ] What happens with missing required fields?
- [ ] What happens with wrong HTTP method?
- [ ] What happens with invalid authentication?
- [ ] What happens with malformed request body?
- [ ] What happens under high load?

---

## 5. Accessibility Heuristics

### 5.1 WCAG Quick Checks

| Area | Heuristic | Test Scenario |
|------|-----------|---------------|
| **Keyboard** | Full navigation without mouse | Tab through all interactive elements |
| **Focus** | Visible focus indicator | Can user see where focus is? |
| **Skip link** | First tab goes to "Skip to content" | Can users bypass navigation? |
| **Labels** | All inputs have accessible labels | Screen reader announces field purpose |
| **Alt text** | Images have descriptive alt text | Non-sighted users understand images |
| **Contrast** | 4.5:1 for normal text, 3:1 for large | Text readable for low vision |
| **Zoom** | Usable at 200% zoom | No horizontal scrolling, content intact |

**Accessibility Edge Case Checklist:**
- [ ] Can all features be accessed via keyboard only?
- [ ] Are focus indicators visible?
- [ ] Do all form inputs have labels?
- [ ] Is the page usable at 200% zoom?
- [ ] Does the screen reader announce important changes?

---

## 6. Sequence and Timing Heuristics

### 6.1 Order Variations

| Variation | Test Scenario |
|-----------|---------------|
| **Forward** | Normal expected sequence |
| **Reverse** | Operations in opposite order |
| **Skip** | Skip optional steps |
| **Repeat** | Perform same action multiple times |
| **Interleave** | Mix different operations |
| **Simultaneous** | Multiple actions at once |

### 6.2 Timing Scenarios

| Scenario | Test Consideration |
|----------|-------------------|
| **Fast user** | Rapid clicking, submitting twice |
| **Slow user** | Session timeout, data staleness |
| **Concurrent** | Two users, same data, same time |
| **Race conditions** | Operations complete out of order |

**Sequence Edge Case Checklist:**
- [ ] What happens if user clicks submit twice quickly?
- [ ] What happens if two users update same record simultaneously?
- [ ] What happens if operations complete out of expected order?
- [ ] What happens if user performs steps out of sequence?

---

## 7. Sorting and Filtering Heuristics

| Heuristic | Test Scenario |
|-----------|---------------|
| **Alpha vs Numeric** | "2" before "10"? | Sort by name with numbers |
| **Case sensitivity** | "Apple" vs "apple" | Alphabetical sorting |
| **Special chars** | Ñ, Ö, accented letters | International sort order |
| **Empty/null values** | Where do they appear? | Sort with missing data |
| **Multiple pages** | Sort across pagination | Does sort persist across pages? |
| **Combined filters** | Multiple active filters | Filter by status AND date |
| **Clear filters** | Return to unfiltered state | Reset button behavior |

**Sorting/Filtering Edge Case Checklist:**
- [ ] Does numeric sorting work correctly (2 vs 10)?
- [ ] Where do null/empty values appear when sorted?
- [ ] Does sorting persist across pagination?
- [ ] Do combined filters work correctly?

---

## 8. Configuration and Environment Heuristics

### 8.1 Browser/Device Variations

| Variation | Test Consideration |
|-----------|-------------------|
| **Browser** | Chrome, Firefox, Safari, Edge |
| **Screen size** | Desktop, tablet, mobile |
| **Orientation** | Portrait vs landscape |
| **Zoom level** | 100%, 125%, 150%, 200% |
| **Font size** | User's browser font settings |

### 8.2 User Preferences

| Preference | Test Consideration |
|------------|-------------------|
| **JavaScript disabled** | Graceful degradation |
| **Cookies disabled** | Session handling |
| **Ad blockers** | Feature functionality |
| **Dark mode** | Color scheme adaptation |
| **Reduced motion** | Animation preferences |

---

## 9. Integration Checklist

After completing BDD analysis, run through this consolidated checklist:

### Data Input Coverage
- [ ] **Strings**: Empty, max length, special chars, unicode, injection
- [ ] **Numbers**: Zero, negative, boundaries, precision, formats
- [ ] **Dates**: Invalid dates, timezone, DST, boundaries
- [ ] **Files**: Empty, large, wrong type, corrupted

### Behavioral Coverage  
- [ ] **CRUD**: Create duplicates, read missing, update conflicts, delete dependencies
- [ ] **Boundaries**: n-1, n, n+1 for all limits
- [ ] **Counts**: Zero, one, many for all collections
- [ ] **States**: Forward, backward, skip, repeat, invalid transitions

### Web/UI Coverage
- [ ] **Navigation**: Back, refresh, bookmark, URL hack, multi-tab
- [ ] **Interruptions**: Network loss, timeout, cancel, close
- [ ] **Input methods**: Type, paste, drag, import, API

### API Coverage
- [ ] **BINMEN**: Boundary, Invalid, Null, Method, Empty, Negative
- [ ] **Auth**: Missing, invalid, expired, insufficient permissions
- [ ] **Errors**: All error codes documented and handled

### Accessibility Coverage
- [ ] **Keyboard**: Full navigation, no traps, visible focus
- [ ] **Screen reader**: Labels, announcements, structure
- [ ] **Visual**: Contrast, zoom, responsive

### Sequence Coverage
- [ ] **Timing**: Double-click, timeout, concurrent access
- [ ] **Order**: Reverse, skip, repeat, interleave

---

## 10. Usage: Applying to Story Analysis

After completing initial BDD analysis, append this validation section to the story:

```markdown
---

## Edge Case Validation

### Heuristics Applied

| Category | Heuristics Reviewed | Edge Cases Added |
|----------|--------------------|--------------------|
| String Inputs | Long, empty, special chars | [List scenarios added] |
| Numeric Inputs | Zero, boundary, negative | [List scenarios added] |
| Date/Time | Timezone, DST, invalid dates | [List scenarios added] |
| CRUD Operations | Duplicates, missing, conflicts | [List scenarios added] |
| Boundaries | n-1, n, n+1 | [List scenarios added] |
| Navigation | Back, refresh, URL manipulation | [List scenarios added] |
| Interruptions | Network, timeout, cancel | [List scenarios added] |
| Accessibility | Keyboard, screen reader, zoom | [List scenarios added] |

### Gaps Identified

| Gap | Decision | Rationale |
|-----|----------|-----------|
| [Missing edge case] | Include / Exclude / Defer | [Why] |

### Confidence Assessment

- [ ] All relevant data type attacks considered
- [ ] All CRUD edge cases covered
- [ ] Boundary values tested
- [ ] Navigation edge cases addressed
- [ ] Error scenarios documented
- [ ] Accessibility requirements verified
```

---

## 11. Quick Reference Cards

### String Testing Quick Card
```
EMPTY → SPACE → SHORT → VALID → BOUNDARY → OVER
Special: ' " < > & / \ ; : Tab Newline
Unicode: éàü 漢字 🎉
Injection: <script> '; DROP TABLE
```

### Number Testing Quick Card  
```
NEGATIVE → ZERO → ONE → VALID → BOUNDARY → OVER
Special: 0.1+0.2, 1E-16, 1,234.56
Boundaries: 255/256, 32767/32768, 2^31-1/2^31
```

### Boundary Testing Quick Card
```
n-1 → n → n+1
Below minimum → At minimum → Above minimum
Below maximum → At maximum → Above maximum
```

### State Testing Quick Card
```
A → B → C (normal)
C → B → A (reverse)
A → C (skip)
A → A (repeat)
A → X (invalid)
```
