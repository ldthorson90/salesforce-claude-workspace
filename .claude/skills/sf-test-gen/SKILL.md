---
name: sf-test-gen
description: Generate comprehensive Apex test classes with bulk scenarios, boundary cases, and high coverage. Follows TDD principles.
user-invocable: true
---

# Salesforce Test Class Generator

Generate comprehensive Apex test classes for existing or new Apex code.

## Arguments

The user provides one of:
- `<ClassName>` — generate tests for an existing Apex class
- `<TriggerName>` — generate tests for an existing trigger + handler
- `batch <ClassName>` — generate tests for a Batch Apex class
- `service <ClassName>` — generate tests for a service/utility class

## Workflow

### Step 1: Analyze the Target Class

Read the source file. Extract:
- Class name and type (trigger handler, service, batch, queueable, schedulable, REST, controller)
- All public/global methods and their signatures
- SOQL queries and DML operations
- External callouts (HttpCalloutMock needed)
- Governor-limit-sensitive operations
- Dependencies (other classes, custom objects, custom metadata)

### Step 2: Generate Test Scenarios (Plain English First)

Before writing any code, list test scenarios:

**For every class:**
- Positive case: expected inputs produce expected outputs
- Negative case: invalid/null inputs handled gracefully
- Bulk case: 200+ records to verify bulkification
- Boundary case: edge values (empty lists, max field lengths, null fields)

**For trigger handlers additionally:**
- Each trigger context (before insert, after insert, before update, after update, delete, undelete)
- Recursion guard verification
- Mixed DML scenario if applicable

**For batch classes additionally:**
- Start method returns correct scope
- Execute processes records correctly
- Finish method performs cleanup/notification
- Batch with 0 records (empty scope)

**For callout classes additionally:**
- Successful response mock
- Error response mock (4xx, 5xx)
- Timeout/exception handling

Present the scenario list to the user for confirmation before writing code.

### Step 3: Generate the Test Class

Follow these rules strictly:

```apex
@IsTest
private class <ClassName>Test {

    @TestSetup
    static void makeData() {
        // Create ALL test data here
        // Use Test.loadData() for complex datasets
        // Never use SeeAllData=true
    }

    @IsTest
    static void testMethodName_positiveCase() {
        // Arrange: query @TestSetup data
        // Act: call the method under test
        // Assert: verify outcomes with meaningful messages
        System.assertEquals(expected, actual, 'Description of what failed');
    }
}
```

**Mandatory patterns:**
- `@IsTest` on the class (not `testMethod` keyword)
- `@TestSetup` for shared test data
- Static test methods with descriptive names: `test<Method>_<scenario>`
- `System.assertEquals` / `System.assertNotEquals` / `System.assert` with message parameter on EVERY assertion
- `Test.startTest()` / `Test.stopTest()` around the method under test to reset governor limits
- Bulk test: insert 200+ records via a loop, then trigger the operation
- Never use `SeeAllData=true`
- Never hardcode org-specific IDs
- Use `Test.isRunningTest()` sparingly and only for callout mocks

**For callouts:**
```apex
@IsTest
static void testCallout_success() {
    Test.setMock(HttpCalloutMock.class, new MockHttpResponse(200, 'OK', '{"key":"value"}'));
    Test.startTest();
    String result = MyService.makeCallout();
    Test.stopTest();
    System.assertNotEquals(null, result, 'Callout should return a response');
}
```

**For async:**
```apex
@IsTest
static void testBatch_processesBulkRecords() {
    // Create 200+ records in @TestSetup
    Test.startTest();
    Database.executeBatch(new MyBatchClass(), 200);
    Test.stopTest();
    // Assert post-batch state
    List<Account> updated = [SELECT Status__c FROM Account WHERE Id IN :testAccounts];
    for (Account a : updated) {
        System.assertEquals('Processed', a.Status__c, 'Batch should update all records');
    }
}
```

### Step 4: Coverage Analysis

After generating, estimate coverage:
- List which methods/branches are covered
- Identify any gaps and whether they're intentional (e.g., catch blocks for unlikely exceptions)
- Target: 85-95%+ meaningful coverage (not just line coverage)

### Step 5: Output

Write the test class file alongside the source:
- Source at `force-app/main/default/classes/<ClassName>.cls`
- Test at `force-app/main/default/classes/<ClassName>Test.cls`
- Test meta at `force-app/main/default/classes/<ClassName>Test.cls-meta.xml`

The meta.xml:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ApexClass xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <status>Active</status>
</ApexClass>
```

## Important

- Tests are specifications. Once written and confirmed, they are immutable.
- If a test fails, fix the implementation, not the test.
- Present the scenario list before writing code so the user can add/remove cases.
