# TC39 Intl Unit Protocol

Status: Stage 0

Champion: Shane F Carr (Google i18n)

## Background

The i18n community has converged on the principle that a number ought to be annotated with the quantity it is measuring. Unlike the localized decimal separators, numbering systems for digits, and grouping strategy, the unit is a core part of the data model to be formatted, not a style that gets applied by the formatter.

For example, when formatting messages, this allows the measurement to be localized with locale unit preferences.

```javascript
const message = "You are {$distance :unit usage=road} from your destination";
let formatter = new MessageFormat("en", message);
formatter.format({
  distance: /* what goes here? */
})
```

A number annotated with a unit is the only data type required for MessageFormat 2.0 that does not have an analog in JavaScript. See: [List of functions in MessageFormat 2.0](https://github.com/unicode-org/message-format-wg/blob/main/spec/functions/README.md).

Adding a new primordial, such as [Amount](https://github.com/tc39/proposal-amount), would solve this problem and multiple others impacting i18n, but it comes at a cost. As a starting point, we should explore solutions that would allow third-party polyfills and MessageFormat libraries to implement an Amount-like object, which is also something the TC39 committee wants to see as part of an Amount proposal.

## Proposed Solution

Add a protocol to `Intl.NumberFormat.prototype.format` that accepts a numeric type paired with a unit. The protocol should also be accepted by `Intl.PluralRules.prototype.select`.

Given these inputs:

```javascript
let locale = /* an Intl.Locale, a string, or a list of these */;
let unit = /* a string */;
let number = /* a Number, a BigInt, or a string */;
```

The user can currently write:

```javascript
let formatter = new Intl.NumberFormat(locale, {
    style: "unit",
    unit,
});
let result = formatter.format(number);
```

With a protocol, the user can instead write:

```javascript
let formatter = new Intl.NumberFormat(locale, {
    style: "unit",
});
let result = formatter.format({
    number,
    unit,
});
```

### Construction vs Formatting

Intl has long allowed constructors and formatting functions to be separate. This achieves two ends:

1. The formatter can encapsulate the locale and options to specify the style and context of the value being formatted, such as when initializing a templating engine.
2. Locale data can be initialized ahead of time, allowing increased efficiency when formatting multiple items in a loop.

By moving `unit` from the constructor bucket to the formatting bucket, we advance goal 1.

The proposal comes at some cost to goal 2, since loading the unit display name has some implementation cost that must now be deferred. We will work with ICU[4X] to minimize this cost.

### Currencies

The protocol would allow currency units to be specified in a similar way.

```javascript
let locale = /* an Intl.Locale, a string, or a list of these */;
let currency = /* a string */;
let number = /* a Number, a BigInt, or a string */;

// Today:
let formatter = new Intl.NumberFormat(locale, {
    style: "currency",
    currency,
});
let result = formatter.format(number);

// With the protocol:
let formatter = new Intl.NumberFormat(locale, {
    style: "currency",
});
let result = formatter.format({
    number,
    currency,  // or unit: currency
});
```

## Integration with Amount and Decimal

The Amount proposal seeks to add a primordial encapsulating a numeric type with a unit. It will implement the Intl protocol specified here.

The Decimal proposal seeks to add a primordial that represents a decimal number in a form designed for correct and efficient arithmetic. It will be supported as another number type for the `number` field in the protocol.
