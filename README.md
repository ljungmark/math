Brick\Math
==========

Provides the `BigInteger` and `BigDecimal` classes to work with arbitrary precision numbers.

[![Build Status](https://secure.travis-ci.org/brick/math.svg?branch=master)](http://travis-ci.org/brick/math)
[![Coverage Status](https://coveralls.io/repos/brick/math/badge.svg?branch=master)](https://coveralls.io/r/brick/math?branch=master)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](http://opensource.org/licenses/MIT)

Installation
------------

This library is installable via [Composer](https://getcomposer.org/).
Just define the following requirement in your `composer.json` file:

    {
        "require": {
            "brick/math": "*"
        }
    }

Requirements
------------

This library requires PHP 5.6, PHP 7 or [HHVM](http://hhvm.com/).

Although the library can work seamlessly on any PHP installation, it is highly recommended that you install the
[GMP](http://php.net/manual/en/book.gmp.php) or [BCMath](http://php.net/manual/en/book.bc.php) extension
to speed up calculations. The fastest available calculator implementation will be automatically selected at runtime.

Package contents
----------------

This library provides the following public classes:

- `Brick\Math\ArithmeticException`: exception thrown when an error occurs.
- `Brick\Math\BigInteger`: represents an arbitrary-precision integer number.
- `Brick\Math\BigDecimal`: represents an arbitrary-precision decimal number.
- `Brick\Math\RoundingMode`: holds constants for the [rounding modes](#division--rounding-modes).

Overview
--------

### Instantiation

The constructor of `BigInteger` and `BigDecimal` is private,
you must use the `of()` factory method to obtain an instance:

    $integer = BigInteger::of('123456');
    $decimal = BigDecimal::of('123.456');

The following types are accepted: `integer`, `float`, `string` as long as they can be safely converted to an integer
in the case of `BigInteger::of()`, or to a decimal number in the case of `BigDecimal::of()`.

Prefer instantiating from `string` or `integer` rather than `float`, as floating-point values are imprecise by design,
and using them with `of()` defeats the purpose of using an arbitrary precision library.

Only `string` allows you to safely instantiate a number with an unlimited number of digits.

### Immutability

The `BigInteger` and `BigDecimal` classes are immutable: their value never changes, so that they can be safely passed around. All methods that return a `BigInteger` or `BigDecimal` return a new object, leaving the original object unaffected:

    $ten = BigInteger::of(10);

    echo $ten->plus(5); // 15
    echo $ten->multipliedBy(3); // 30

### Parameter types

All methods that accept a number: `plus()`, `minus()`, `multipliedBy()`, etc. accept the same types as `of()`.
As an example, given the following number:

    $integer = BigInteger::of(123);

The following lines are equivalent:

    $integer->multipliedBy(123);
    $integer->multipliedBy('123');
    $integer->multipliedBy($integer);

### Chaining

All the methods that return a new number can be chained, for example:

    echo BigInteger::of(10)->plus(5)->multipliedBy(3); // 45

### Division & rounding modes

#### BigInteger

Dividing a `BigInteger` always returns the *quotient* of the division:

    echo BigInteger::of(1000)->dividedBy(3); // 333

You can get the remainder of the division with the `remainder()` method:

    echo BigInteger::of(1000)->remainder(3); // 1

You can also get both the quotient and the remainder in a single method call:

    list ($quotient, $remainder) = BigInteger::of(1000)->divideAndRemainder(3);

#### BigDecimal

When dividing a `BigDecimal`, if the number cannot be represented at the requested scale, the result needs to be rounded up or down.
By default, the library assumes that rounding is unnecessary, and throws an exception if rouding was in fact necessary:

    BigDecimal::of('1000.0')->dividedBy(3); // throws an ArithmeticException

In that case, you need to explicitly provide a rounding mode:

    echo BigDecimal::of('1000.0')->dividedBy(3, RoundingMode::DOWN); // 333.3
    echo BigDecimal::of('1000.0')->dividedBy(3, RoundingMode::UP); // 333.4

By default, the result has the same scale as the number, but you can also specify a different scale:

    echo BigDecimal::of(3)->dividedBy(11, RoundingMode::UP, 2); // 0.28
    echo BigDecimal::of(3)->dividedBy(11, RoundingMode::DOWN, 6); // 0.272727

There are a number of rounding modes you can use:

Rounding mode  | Description
-------------- | -----------
`UNNECESSARY`  | Assumes that no rounding is necessary, and throws an exception if it is.
`UP`           | Rounds away from zero.
`DOWN`         | Rounds towards zero.
`CEILING`      | Rounds towards positive infinity. If the result is positive, behaves as for `UP`; if negative, behaves as for `DOWN`.
`FLOOR`        | Rounds towards negative infinity. If the result is positive, behave as for `DOWN`; if negative, behave as for `UP`.
`HALF_UP`      | Rounds towards "nearest neighbor" unless both neighbors are equidistant, in which case round up. Behaves as for `UP` if the discarded fraction is >= 0.5; otherwise, behaves as for `DOWN`.
`HALF_DOWN`    | Rounds towards "nearest neighbor" unless both neighbors are equidistant, in which case round down. Behaves as for `UP` if the discarded fraction is > 0.5; otherwise, behaves as for `DOWN`.
`HALF_CEILING` | Rounds towards "nearest neighbor" unless both neighbors are equidistant, in which case round towards positive infinity. If the result is positive, behaves as for `HALF_UP`; if negative, behaves as for `HALF_DOWN`.
`HALF_FLOOR`   | Rounds towards "nearest neighbor" unless both neighbors are equidistant, in which case round towards negative infinity. If the result is positive, behaves as for `HALF_DOWN`; if negative, behaves as for `HALF_UP`.
`HALF_EVEN`    | Rounds towards the "nearest neighbor" unless both neighbors are equidistant, in which case rounds towards the even neighbor. Behaves as for `HALF_UP` if the digit to the left of the discarded fraction is odd; behaves as for `HALF_DOWN` if it's even.

### Serialization

`BigInteger` and `BigDecimal` can be safely serialized on a machine and unserialized on another.
No context is stored in the serialized variable, only the value.