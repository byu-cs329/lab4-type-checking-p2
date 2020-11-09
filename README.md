# Objective

The objective of this lab is to implement the last of the [static type checking](https://en.wikipedia.org/wiki/Type_system#Static_type_checking) for a subset of Java from lab 3, write tests that test the tests generated for the type checker to create a regression set of tests, and then run mutation analysis on the regression set adding new tests as needed to kill all mutants.

# Reading

[PIT Mutation Analysis](https://bitbucket.org/byucs329/byu-cs-329-lecture-notes/src/master/pit-mutation-analysis/mutation-analysis.md)

# Lab Requirements

Copy into this directory the solution from lab 3. Extra points if it is done using `git` in some way (i.e., `format-patch`, `apply`, forced merge from unrelated histories, etc.). The lab 3 solution is the starting point for this lab.

## Finish the Type Checker

Implement the remaining type rules for the static type proof that were not assigned in lab 3:

  * `FieldAccess:<T>` (e.g., `this.a`) (lab 4)
  * `QualifiedName:<T>` (e.q., `n.a`) (lab 4)
  * `MethodInvocation:<T>` (lab 4)

The `<T>` means some type as defined in the environment. Please note that the fields and methods are already in the `ISymbolTable` instance generated by `ISymbolTableBuilder`.

## Tests to Test the Dynamic Tests

Lab 3 uses visual inspection to check that the generated type proof (e.g., the `DynamicNode` returned from `TypeCheckBuilder` holding the tests for the type proof) is correct. This lab requires tests to tests those tests. In other words, create tests that make sure the set of generated dynamic tests from the `TypeCheckBuilder` is correct. A test to test the tests needs to inspect the contents and structure of the `DynamicNode` returned from the `TypeCheckBuilder` and make sure it has expected structure and content of the type proof. If it does not, then the test fails. 

The `DynamicNode` class provides a `DynamicNode.getDisplayName()` method. The `DynamicContainer` class gives access to the stream of nodes in the container with `DynamicContainer.getChilderen()`. The `DynamicTest` class gives access to the embedded executable with `DynamicTest.getExecutable()`. The `Executable` class is able to run the test with `Executable.execute()`. If the test fails, then it throws an instance of `AssertionFailedError`. These methods provide the interface to write tests to test tests. Here is a simplistic example.

```java
@Test
  void myTest() {
    String fileName = "typeChecker/should_proveTypeSafe_when_givenVariableDeclrationsWithCompatibleInits.java";
    List<DynamicNode> tests = new ArrayList<>();
    boolean isTypeSafe = getTypeChecker(fileName, tests);
    Assertions.assertTrue(isTypeSafe);
    Iterator<DynamicNode> iter = tests.iterator();
    DynamicNode n = iter.next();
    Assertions.assertTrue(n instanceof DynamicTest);
    DynamicTest t = (DynamicTest) n;
    Assertions.assertDoesNotThrow(t.getExecutable());
    n = iter.next();
    Assertions.assertTrue(n instanceof DynamicTest);
    t = (DynamicTest) n;
    Assertions.assertThrows(AssertionFailedError.class, t.getExecutable());
  }
```

The above is more detailed than it should be for the type checker test code, but it shows some of what is possible. The `DynamicContainer` only give access to a stream, but from there it should be possible to roughly test for an expected structure it terms of display names, containers, number of tests, or even number of tests in each container.  The [lecture notes](https://bitbucket.org/byucs329/byu-cs-329-lecture-notes/src/master/type-checking.md) have a detailed discussion to what these tests might look like for the type checker.

Write a minimum set of regression tests to cover the behavior of the type checker. The lab3 tests are an excellent starting point. To only difference here is that those tests do not run directly but are rather inspected for correctness by the tests in this lab.

## PIT Mutation Analysis

Analyze the fitness of the regression test suite using [PIT](http://pitest.org) for mutations analysis with the `DEFAULT` group and the added `RETURN_VALS` as mutators. The `pom.xml` is already configured for the PIT analysis. That analysis is run with `mvn test org.pitest:pitest-maven:mutationCoverage`. The report is in the `./target/pit-reports` directory organized by time-stamp. 

Add tests to kill all mutants or argue why a mutant cannot be killed. Clearly indicate where the explanations are located in the pull request.

# What to turn in?

Create a pull request when the lab is done. Submit to Canvas the URL of the repository.

# Rubric

| Item | Point Value |
| ------- | ----------- |
| `ReturnStatement:void` | 10 | 
| `ReturnStatement:void` tests| 15 |
| `ExpressionStatement:void` for `Assignment` | 10 | 
| `ExpressionStatement:void` for `Assignment` tests | 15 | 
| `PrefixExpression:boolean` | 10 |
| `PrefixExpression:boolean` tests | 15 |
| `InfixExpression:int` | 10 |
| `InfixExpression:int` tests | 15 |
| `InfixExpression:boolean` | 10 | 
| `InfixExpression:boolean` tests | 15 | 
| `IfStatement:void` | 10 |
| `IfStatement:void` tests | 15 |
| `WhileStatement:void` | 10 | 
| `WhileStatement:void` tests | 15 | 
| Style, documentation, naming conventions, test organization, readability, etc. | 25 |
