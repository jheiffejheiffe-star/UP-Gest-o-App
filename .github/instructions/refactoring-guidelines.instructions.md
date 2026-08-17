---
name: refactoring-guidelines
description: "Use when: refactoring the SCA legacy VB6 project - provides architecture guidelines, anti-patterns to avoid, and refactoring priorities specific to this codebase"
applyTo: ["**/*.bas", "**/*.vb", "ONLINE.BAS"]
---

# SCA Refactoring Guidelines

## Project Context

This is a legacy Visual Basic 6 application with multiple modules and extensive third-party integrations. The system manages:
- User management and authentication
- Hardware integration (biometrics, card readers, turnstiles)
- Database operations (Access MDB)
- Financial transactions
- Hardware device communication
- Reporting and data analysis

## Architecture Analysis

### Current Structure
```
SCA/
├── ONLINE.BAS (main module)
├── Multiple .exe modules (admin, biometria, vendas, etc.)
├── Multiple .dll dependencies (hardware APIs, frameworks)
├── Access database (dados.mdb)
└── Hardware drivers and components (OCX files)
```

### Key Dependencies
- Visual Basic Runtime (MSVBVM60.DLL)
- Hardware libraries (Daruma, DcCatraca, etc.)
- Communication libraries (MSWINSCK.OCX, MSCOMM32.OCX)
- Biometric libraries (fingerprint, facial recognition)
- Financial frameworks (Boleto, payment processing)

## Refactoring Priorities

### Priority 1: Code Analysis & Mapping
- [ ] Create comprehensive code inventory
- [ ] Document all function signatures and entry points
- [ ] Map dependency graph
- [ ] Identify critical vs. optional modules
- [ ] Document database schema

### Priority 2: Duplication Elimination
- [ ] Identify repeated code patterns across modules
- [ ] Extract common utility functions
- [ ] Create shared module libraries
- [ ] Document refactoring impact

### Priority 3: Dead Code Removal
- [ ] Analyze which functions/modules are actively used
- [ ] Document deprecated features
- [ ] Create removal plan with safety checks
- [ ] Validate after each removal

### Priority 4: Performance Optimization
- [ ] Profile database queries
- [ ] Optimize hardware communication loops
- [ ] Reduce redundant API calls
- [ ] Implement caching where appropriate
- [ ] Profile UI responsiveness

### Priority 5: Error Handling & Security
- [ ] Standardize error handling patterns
- [ ] Add input validation at all entry points
- [ ] Implement secure credential handling
- [ ] Add logging and audit trails
- [ ] Review hardware communication for security

### Priority 6: Code Organization
- [ ] Reorganize modules by function
- [ ] Create consistent naming conventions
- [ ] Implement module interfaces
- [ ] Document module responsibilities
- [ ] Create utility module libraries

### Priority 7: Maintainability
- [ ] Add inline documentation
- [ ] Create architecture documentation
- [ ] Document integration points
- [ ] Create troubleshooting guides
- [ ] Document build and deployment process

## Anti-Patterns to Avoid

### DO NOT:
- [ ] Remove any user-facing functionality
- [ ] Change database schema without migration plan
- [ ] Break hardware integration points
- [ ] Remove exception handling
- [ ] Simplify without testing
- [ ] Make multiple unrelated changes at once
- [ ] Ignore backward compatibility
- [ ] Skip validation after each change

### DO:
- [ ] Test each change immediately
- [ ] Document all changes
- [ ] Make small, focused changes
- [ ] Preserve all existing behavior
- [ ] Create backup before major changes
- [ ] Validate with existing test cases
- [ ] Update documentation after changes

## Refactoring Techniques for VB6

### Module Consolidation
Extract common code into shared modules:
```vb
' AVOID: Duplicated across modules
Function ValidateEmail(email As String) As Boolean
  ' validation code repeated in Admin, User, Email modules
End Function

' DO: Create shared utils module
Module UtilsCommon
  Function ValidateEmail(email As String) As Boolean
    ' centralized validation
  End Function
End Module
```

### Error Handling Standardization
Replace scattered error handling with patterns:
```vb
' AVOID: Inconsistent error handling
On Error GoTo 0  ' ignoring errors
Sub DoSomething()
  ' code without proper error handling
End Sub

' DO: Consistent error handling
Function TryDoSomething(ByRef result As Variant) As Boolean
  On Error GoTo ErrorHandler
  ' code here
  TryDoSomething = True
  Exit Function
ErrorHandler:
  LogError "DoSomething", Err
  TryDoSomething = False
End Function
```

### Database Query Optimization
Cache and batch database operations:
```vb
' AVOID: Multiple round trips
For Each user In users
  Call db.GetUserData(user.Id)  ' One query per user!
Next

' DO: Batch queries
Dim userData As Collection
Set userData = db.GetUserDataBatch(userIds)
```

### Dependency Injection Pattern
Reduce tight coupling to hardware/external services:
```vb
' AVOID: Hard-coded dependencies
Sub ProcessBiometric()
  Call BiometricReader.Scan()  ' Can't test without hardware
End Sub

' DO: Inject dependencies
Sub ProcessBiometric(reader As IBiometricReader)
  Call reader.Scan()  ' Can mock/inject for testing
End Sub
```

## Testing Strategy

### Validation After Refactoring
1. **Unit Level**: Test individual functions still work
2. **Integration**: Verify modules still communicate correctly
3. **Hardware**: Ensure device communication unchanged
4. **Database**: Verify data operations unchanged
5. **End-to-End**: Test complete workflows

### Regression Prevention
- Document current behavior before changes
- Create smoke tests for critical paths
- Maintain compatibility shim layer during transition
- Use parallel runs (old vs. new) for validation

## Documentation Template

After each refactoring, document:
```markdown
## Refactoring: [Name]
**Date**: YYYY-MM-DD
**Scope**: [Files/Modules affected]
**Changes**:
- [Change 1]
- [Change 2]

**Testing**:
- [Test 1 result]
- [Test 2 result]

**Validation**: ✓ Passed / ✗ Failed
**Rollback Plan**: [If needed]
```

## Version Control

- Commit each refactoring separately with clear messages
- Include refactoring ticket number in commit
- Tag major milestones
- Maintain branch for each refactoring area
- Merge to main only after validation

## Success Criteria

✓ All original functionality preserved  
✓ No new bugs introduced  
✓ Improved code organization  
✓ Reduced duplication  
✓ Better error handling  
✓ Enhanced security  
✓ Improved maintainability  
✓ Documented changes  
✓ Passing validation tests
