HikoTest Unit Testing Framework
===============================

I created my own unit testing framework since I had several issues
with other unit-testing frame works with testing the HikoGUI library.

Design decisions:
 * Simple test-dispatch architecture so that it is easy to debug.
   In debug-mode no exceptions are caught, and break-points are set.
 * Limit the amount of stack space required. To not cause the
   address sanitizer to trigger on stack overflow with a bunch of simple tests.
 * Comparisons for: values, ranges, absolute difference and ranges
   of absolute difference.
 * Print operands of comparisons using `std::format()`, `std::ostream()`,
   `to_string()`, `.string()` or `.str()`.
 * Command line and output compatible with google-test so that
   is can be used with IDEs.
 * Very low number of test macros: `REQUIRE()` and `REQUIRE_THROW()`.
 * A test-suite is a class, a test is a member-function.
  

Example
-------

```cpp
#include <hikotest/hikotest.hpp>

// Adds the test suite "my" the _suite suffix is automatically removed.
TEST_SUITE(my_suite) {
    // Adds the test "first" the _test suffix is automatically removed.
    TEST_CASE(first_test)
    {
        auto foo = 42;
        auto bar = 42.2f;

        // The compare operator is overloaded by the test framework;
        // on error the left and right hand side values are displayed.
        REQUIRE(foo == 42);

        // The !=, <, >, <=, >= operators may also be used with exact comparisons.
        REQUIRE(foo < 43);

        // The optional arguments to REQUIRE() are passed to the constructor
        // of ::test::error. In this case the comparison compares the values
        // with an absolute error of maximum 0.5.
        REQUIRE(bar == 42, 0.5);

        for (auto i = 0; i != 10; ++i) {
            // You may also pass in a std::string to the ::test::error constructor.
            // This string is displayed on error. Useful for tracking the iteration
            // of a loop.
            REQUIRE(i < 10, std::format("i = {}", i));
        }
    }

    TEST_CASE(second_test)
    {
        auto foo = std::array{1, 42};
        auto bar = std::array{0.9f, 42.2f};

        // The comparison will also handle the comparison of ranges.
        REQUIRE(foo == std::array{1, 42});

        // Range comparison can even be compared with an absolute error value.
        REQUIRE(bar == std::array{1, 42}, 0.5);
    }

    TEST_CASE(second_test)
    {
        auto foo = std::vector{};
        auto bar = 42;

        // If you do not use a comparator the expression is converted to a bool.
        REQUIRE(foo.empty());

        // If you want to explicitly test operator==() of a custom type, then
        // you can add brackets like this.
        REQUIRE((bar == 42));
    }

    TEST_CASE(forth_test)
    {
        // We use a separate macro for testing if a expression throws an exception.
        REQUIRE_THROWS(throw std::runtime_error{"oops"}, std::runtime_error);
    }
};
```

REQUIRE macros
--------------

### REQUIRE(expression, ...)

Expression may be:
 * A comparison expression using one of the following operators: `==`, `!=`, `<`, `>`, `<=`, `>=`.
 * A boolean expression; resulting in a type that can be implicitly converted to `bool`.

You may escape a comparison expression by surrounding it with parentheses `(` `)` so that
it will be interpreted as a boolean expression.

Internally `REQUIRE()` will concatenate `<=> ::test::error(__VA_ARGS__)` to the expression.
This is done to separate the right operand from the comparison, so that the comparison operator
will be replaced by the unit testing framework.

The optional arguments of `REQUIRE()` are passed to the `::test::error` constructor.
The `::test::error` constructor accepts the following arguments:
 * Empty: The comparison will do an exact match.
 * A floating point number: The equality comparison will take into account
   an absolute error.
 * A `::test::error` object: Will use the copy constructor.
 * (optional) `std::string` object: Will display the string on error. 

### REQUIRE\_THROWS(expression, exception-type)

Command Line Arguments
----------------------

 :--------------------------- |:---------------------------------
  `--help`                    | List all command line arguments.
  `--gtest_list_tests`        | List all the tests in the executable.
  `--gtest_filter=<filter>`   | Filter the tests to be run.
  `--gtest_output=xml:<path>` | Write a JUnit XML file to `<path>`.

### filter

A filter contains two parts:
 * On the left side a colon ':' separated list of inclusion glob patterns
 * On the right side a minus '-' sign followed by a colon ':' separated list of exclusion glob pattern.

Both the inclusion list and exclusion lists are optional.

The glob pattern may contain:
 * A character that is part of a C++ identifier
 * A single '.' to separate the 'suite' and 'test'.
 * A '?' to replace a single character
 * A '*' to replace zero or more characters.

