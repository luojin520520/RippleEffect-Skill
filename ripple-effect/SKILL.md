---
name: "ripple-effect"
version: "3.0"
description: "Systematically prevent dependency chain breakage from code changes, covering code references, data flows, API contracts, configuration dependencies, and frontend-backend consistency to ensure all affected layers are synchronized when modifying A."
author: "wechat: Anonymvs1234"
---

# RippleEffect Skill

## Activation Conditions

This skill activates automatically when:
- Modifying any code file (frontend, backend, configuration, type definitions)
- Modifying data structures, type definitions, interface contracts
- Modifying API endpoints, request/response formats
- Modifying configuration files, environment variables
- Modifying data processing logic, calculation rules
- Modifying frontend-backend communication code

## Core Principles

### Principle1: Multi-Dimensional Dependency Analysis
Single-dimensional analysis is insufficient; must cover:
- **Code Reference Dependencies** - Who calls this function/class
- **Data Flow Dependencies** - Who consumes this data's output
- **Contract Dependencies** - Who depends on this interface format
- **Configuration Dependencies** - Who uses this configuration
- **Implicit Dependencies** - B doesn't import A but depends on A's output format/behavior

### Principle2: Think Once, Cover Everything
- **Forbidden**: Only look at code references, ignore data flows and contracts
- **Required**: Understand the complete dependency network
- **Output**: Complete list of all affected layers

### Principle3: Synchronize Changes or Evaluate Alternatives
- Option A: When modifying A, synchronize changes to all affected layers
- Option B: If impact is too large, evaluate better architecture options

## Multi-Dimensional Dependency Analysis Framework

### Dimension1: Code Reference Dependencies (Existing)

```
Analysis Target: Functions, classes, variables, modules
Analysis Method: lsp_find_references, grep import
Output: Direct callers, indirect callers
```

### Dimension2: Data Flow Dependencies (New)

```
Analysis Target: Data producer → Data consumer
Analysis Method:
- Track data flow: From data production to final consumption
- Identify implicit dependencies: B doesn't import A but uses A's output
- Check serialization/deserialization: JSON, ProtoBuf, FormData, etc.

Output: Data flow diagram
```

**Example Scenario**:
```
A: UserSerializer.toJSON()  // Produces user data
B: AnalyticsService.process()  // Consumes user data but doesn't directly import UserSerializer
C: ReportGenerator.export()    // Gets data from AnalyticsService

Modify A's toJSON() output fields → B and C's processing logic may break
```

### Dimension3: Contract Dependencies (New)

```
Analysis Target: API interfaces, data formats, type definitions
Analysis Method:
- API Contracts: OpenAPI/Swagger docs, route definitions
- Data Formats: TypeScript interfaces, Python dataclass, database Schema
- Message Formats: Event messages, Queue messages, WebSocket messages

Output: Contract dependency graph
```

**Example Scenario**:
```
Frontend: POST /api/user { name, email, phone }
Backend: UserController.create(req.body)

Frontend adds phone field → Backend DTO missing → 500 error
```

### Dimension4: Configuration Dependencies (Existing but Enhanced)

```
Analysis Target: Config files, environment variables, Feature Flags
Analysis Method:
- grep config key names
- Track config usage locations
- Identify impact chains of config changes

Output: Configuration impact scope
```

### Dimension5: Frontend-Backend Consistency (New Core)

```
Analysis Target: All layers of frontend-backend interaction
Analysis Method:
1. API Route Consistency
   - Frontend request paths vs backend route definitions
   - HTTP method consistency
   
2. Request/Response Format Consistency
   - Frontend request body format vs backend DTO definitions
   - Backend response format vs frontend type definitions
   
3. Status Code/Error Handling Consistency
   - Backend returned error codes vs frontend handled error codes
   
4. Business Logic Consistency
   - Frontend validation rules vs backend validation rules
   - Frontend calculation logic vs backend calculation logic

Output: Frontend↔Backend consistency check report
```

## Operational Flow

### Phase1: Multi-Dimensional Dependency Scanning

```
┌─────────────────────────────────────────────────────────────────┐
│ Step1: Locate Modification Target                                │
├─────────────────────────────────────────────────────────────────┤
│ • File path + code location                                      │
│ • Modification type: function/class/type/config/API/data format │
│ • Change nature: add/delete/modify                              │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ Step2: Multi-Dimensional Dependency Scanning                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ DimensionA: Code Reference Dependencies                          │
│ ├── lsp_find_references → all direct references                 │
│ ├── grep "import" / "require" → module importers                │
│ └── recursively analyze indirect references                     │
│                                                                 │
│ DimensionB: Data Flow Dependencies                               │
│ ├── track data producer → consumer chains                       │
│ ├── grep data format usage (JSON.parse, serialize, etc.)        │
│ ├── check data transformation/mapping logic                     │
│ └── identify implicit dependencies (no import but uses output)  │
│                                                                 │
│ DimensionC: Contract Dependencies                                │
│ ├── API routes: grep route definitions + request paths          │
│ ├── Data formats: TypeScript interfaces, Python types           │
│ ├── Message formats: Event schemas, Queue messages              │
│ └── Validation rules: frontend validation vs backend validation │
│                                                                 │
│ DimensionD: Configuration Dependencies                           │
│ ├── grep config keys/environment variables                      │
│ ├── track config usage locations                                │
│ └── identify impact chains of config changes                    │
│                                                                 │
│ DimensionE: Frontend-Backend Consistency                        │
│ ├── Route mapping: frontend API calls vs backend routes         │
│ ├── Format mapping: request/response body format consistency    │
│ ├── Type mapping: TypeScript interfaces vs backend models       │
│ └── Error code mapping: error codes handling consistency        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ Step3: Build Complete Dependency Graph                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ Modification Target: updateUserProfile(user: UserInput)         │
│                                                                 │
│ Dependency Graph:                                                │
│                                                                 │
│ Code References (4 locations)                                    │
│ ├── user.service.ts:42 - updateUserProfile()                    │
│ ├── admin.controller.ts:15 - calls updateUserProfile            │
│ └── user.controller.spec.ts:89 - test case                      │
│                                                                 │
│ Data Flows (2 implicit)                                          │
│ ├── UserInput → UserSerializer.toJSON() → Analytics             │
│ └── UserInput → AuditLogger.log() → Logging System              │
│                                                                 │
│ Contract Dependencies (6 locations)                              │
│ ├── Frontend: POST /api/users/{id} body: { name, email, phone }│
│ ├── Backend: PUT /users/:id DTO: { name, email }  ← phone missing!│
│ ├── Frontend Type: UserInput { name, email, phone }             │
│ ├── Backend Type: UpdateUserDTO { name, email }  ← phone missing!│
│ ├── Frontend Validation: phone.required()                       │
│ └── Backend Validation: no phone validation                     │
│                                                                 │
│ Configuration Dependencies (1 location)                          │
│ └── user.update.enabled: true                                   │
│                                                                 │
│ ⚠️ Critical Issues Found:                                        │
│ Frontend adds phone field, backend DTO missing phone → 500 error│
│ Frontend validates phone.required, backend has no validation → data inconsistency│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Phase2: Option Development

```
1. Assess Change Scope
   ┌────────────────────────────────────────────────────────────┐
   │ Code References: N locations (direct changes)              │
   │ Data Flows: M locations (implicit dependencies, need adaptation)│
   │ Contracts: K locations (API/types, need synchronization)   │
   │ Frontend-Backend: L locations (inconsistencies, need alignment)│
   │ Configurations: P locations (may need adjustment)          │
   └────────────────────────────────────────────────────────────┘

2. Generate Option Options

   OptionA: Full Synchronized Changes (Recommended)
   ─────────────────────────────────────────────
   Changes: N+M+K+L+P locations
   Risk: Medium
   Applicable: Simple changes, need quick alignment

   OptionB: Progressive Compatibility
   ─────────────────────────────────────────────
   Changes: Backend adds optional fields, doesn't break existing logic
   Risk: Low
   Applicable: Production environment, need smooth migration

   OptionC: Architecture Refactoring
   ─────────────────────────────────────────────
   Changes: Extract shared type definitions, force frontend-backend type consistency
   Risk: High
   Applicable: Long-term technical debt, need fundamental solution

3. Recommended Option + Complete Change List
   ┌────────────────────────────────────────────────────────────┐
   │ [Recommended Option] OptionA - Full Synchronized Changes   │
   │                                                            │
   │ Complete Change List:                                      │
   │                                                            │
   │ Code Layer (3 locations)                                   │
   │ ├── user.dto.ts: add phone field to UpdateUserDTO         │
   │ ├── user.service.ts: handle phone field                   │
   │ └── admin.controller.ts: adapt to new DTO                 │
   │                                                            │
   │ Data Flow Layer (2 locations)                              │
   │ ├── UserSerializer: add phone field to output             │
   │ └── AnalyticsService: adapt to new user data structure    │
   │                                                            │
   │ Contract Layer (4 locations)                               │
   │ ├── OpenAPI spec: update request/response body            │
   │ ├── TypeScript UserInput: add phone                       │
   │ ├── Python UpdateUserDTO: add phone                       │
   │ └── Backend validation: add phone.required()              │
   │                                                            │
   │ Test Layer (2 locations)                                   │
   │ ├── user.controller.spec.ts: update test data             │
   │ └── e2e/user.test.ts: update e2e test cases               │
   │                                                            │
   │ Execution Order:                                           │
   │ Step 1: Modify type definitions (contracts first)          │
   │ Step 2: Modify data flow (Serializer, Consumer)            │
   │ Step 3: Modify business logic (Service, Controller)        │
   │ Step 4: Update tests                                       │
   │ Step 5: Run diagnostics + test verification                │
   └────────────────────────────────────────────────────────────┘
```

### Phase3: Execution and Verification

```
Core Principle: Think completely, execute atomically

Before Execution:
✓ Dependency scan covers all dimensions
✓ Option includes all change locations
✓ Frontend-backend inconsistencies identified

After Execution:
✓ lsp_diagnostics no errors
✓ Type checks pass
✓ All relevant tests pass
✓ Frontend-backend contracts aligned
✓ Data flow verification passed
```

## Complete Execution Example

### User Request
```
"Frontend needs to add user phone number field, backend needs to support it"
```

### Phase1: Multi-Dimensional Dependency Scanning

```
=== Multi-Dimensional Dependency Analysis Report ===

Modification Target: UserInput type add phone field

【DimensionA: Code Reference Dependencies】
user.dto.ts:15 - UserInput definition
user.service.ts:42 - Uses UserInput
admin.controller.ts:15 - Passes UserInput
user.test.ts:89 - Test case

【DimensionB: Data Flow Dependencies】
UserInput → UserSerializer.toJSON() → AnalyticsService
UserInput → AuditLogger.log() → Logging System

【DimensionC: Contract Dependencies】
Frontend API: POST /api/users { name, email, phone }
Frontend Type: UserInput { name, email, phone }
Backend Route: PUT /users/:id
Backend DTO: UpdateUserDTO { name, email }  ← phone missing!
Backend Validation: no phone validation rules

【DimensionD: Frontend-Backend Consistency】
❌ Inconsistency1: Frontend phone required, backend no phone field
❌ Inconsistency2: Frontend type has phone, backend DTO doesn't
❌ Inconsistency3: Frontend validates phone.required, backend has no corresponding validation

【DimensionE: Configuration Dependencies】
No configuration dependencies

【Summary】
Impact Scope: 5 files need modification
Critical Risk: Backend DTO missing phone field → 500 error
Recommendation: OptionA - Full Synchronized Changes
```

### Phase2: Option Development

```
【OptionA: Full Synchronized Changes】 Recommended

Change List:
1. user.dto.ts - add phone field to UpdateUserDTO
2. user.service.ts - handle phone field
3. UserSerializer - serialize phone field
4. AnalyticsService - adapt to new field
5. OpenAPI spec - update contract documentation
6. Frontend UserInput - already has phone (no change needed)
7. Backend validation - add phone.required()
8. user.test.ts - update test data

Execution Order:
Step 1: Modify type definitions (dto, interfaces)
Step 2: Modify serialization/deserialization
Step 3: Modify business logic
Step 4: Update tests
Step 5: Verify (diagnostics + tests)
```

### Phase3: Complete Execution

```
apply_patch << 'EOF'
*** Begin Patch
*** Update File: src/dto/user.dto.ts
@@
- interface UpdateUserDTO {
-   name: string;
-   email: string;
- }
+ interface UpdateUserDTO {
+   name: string;
+   email: string;
+   phone?: string;  // New field
+ }

*** Update File: src/services/user.service.ts
@@
- async function updateUser(id: string, input: UserInput) {
-   const { name, email } = input;
+ async function updateUser(id: string, input: UserInput) {
+   const { name, email, phone } = input;
+   // Handle phone field
+   if (phone) {
+     await userPhoneService.updatePhone(id, phone);
+   }

*** Update File: src/serializers/user.serializer.ts
@@
- toJSON(user: User): object {
-   return { name: user.name, email: user.email };
+ toJSON(user: User): object {
+   return { name: user.name, email: user.email, phone: user.phone };

*** Update File: src/validators/user.validator.ts
@@
- const updateUserSchema = z.object({
-   name: z.string(),
-   email: z.string().email(),
- });
+ const updateUserSchema = z.object({
+   name: z.string(),
+   email: z.string().email(),
+   phone: z.string().optional(),
+ });

*** Update File: tests/user.test.ts
@@
- const mockInput = { name: 'John', email: 'john@example.com' };
+ const mockInput = { name: 'John', email: 'john@example.com', phone: '1234567890' };
*** End Patch
EOF

→ Run lsp_diagnostics
→ Run tests (unit tests + e2e tests)
→ Verify frontend-backend contract alignment
→ Confirm data flow is normal
```

## Frontend-Backend Consistency Checklist

When modifying any frontend-backend interaction code, must check:

```
□ Route Consistency
  - Frontend request path matches backend route
  - HTTP methods match (GET/POST/PUT/DELETE)

□ Request Format Consistency
  - Frontend request body vs backend DTO fields
  - Required fields match
  - Field types match

□ Response Format Consistency
  - Backend response structure vs frontend type definitions
  - Pagination data format
  - Error response format

□ Status Code Consistency
  - Backend HTTP status codes vs frontend handling
  - Business error code mapping

□ Validation Rule Consistency
  - Frontend validation vs backend validation
  - Error messages match

□ Business Logic Consistency
  - Frontend calculation vs backend calculation
  - Sort/filter rules
  - Pagination logic
```

## Tool Usage Specification

| Analysis Dimension | Tool | Output |
|-------------------|------|--------|
| Code References | lsp_find_references | Call chain |
| Data Flow | grep + tracking | Data flow diagram |
| API Routes | grep + AST | Route mapping table |
| Data Formats | lsp_symbols | Type definitions |
| Configuration | grep | Configuration impact diagram |
| Frontend-Backend | grep + comparison | Consistency report |

## Common Scenarios and Handling

### Scenario1: Frontend Changes Affect Backend
```
Trigger: Modify frontend API call/data format
Check:
  → Does corresponding backend route exist
  → Does backend DTO match
  → Do validation rules align
```

### Scenario2: Backend Changes Affect Frontend
```
Trigger: Modify backend API/response format
Check:
  → Do frontend type definitions need update
  → Do frontend calls need adaptation
  → Is error handling consistent
```

### Scenario3: Data Structure Changes
```
Trigger: Modify TypeScript interface/database Schema
Check:
  → All serialization/deserialization points
  → Do data consumers need adaptation
  → Does cache layer need update
```

### Scenario4: Configuration Changes
```
Trigger: Modify environment variables/Feature Flag
Check:
  → All services depending on this config
  → Is startup order affected
  → Is fallback plan ready
```

## Core Value

```
❌ Traditional Approach:
User: Add phone field on frontend
AI: Done
User: Backend returns 500
AI: Oh, backend DTO missing
User: Tests also failed
AI: Oh, test data missing
...loop

✅ DependencyGuard Approach:
User: Add phone field on frontend
AI: (Multi-dimensional analysis)
   - Backend DTO needs phone
   - Validation rules need phone
   - Test data needs phone
   - Analytics consumer needs adaptation
   Option: Change all points at once, verified successfully
```

## Default System Prompt

```
Enable dependency-guard skill

When receiving any modification request:
1. Perform multi-dimensional dependency analysis (code references + data flow + contracts + config + frontend-backend)
2. Identify implicit dependencies (B doesn't import A but uses A's output)
3. Check frontend-backend consistency (routes/formats/validation/status codes)
4. Generate complete plan including all affected layers
5. Execute all necessary changes at once
6. Complete verification after execution (diagnostics + tests + frontend-backend contract alignment)

Forbidden:
- Only analyze code references, ignore data flow and contracts
- Only modify frontend or only modify backend
- Ignore implicit dependencies (B doesn't import A but uses A's output)

Output Format:
- Multi-dimensional dependency analysis report
- Frontend-backend consistency check results
- Complete change list (grouped by dimension)
- Execution order (contracts first → data flow → logic → tests)
```
