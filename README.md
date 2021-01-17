# jest-clean-console-reporter (alpha)

_A custom Jest reporter to reduce `console` spam in your test output._

---

Remember back in the day when [Jest](https://jestjs.io/) used to swallow `console.log` messages by default, and it was basically impossible to figure out why your code was failing under test? \***Sigh\***. Those were the days.

Now that Jest prints all console messages, whether from our own code, third-party framework code, or the test runner itself, we have the opposite problem: **It's easy to lose actually important warnings among noise.**

The `jest-clean-console-reporter` reporter collects all console messages written by the test process, and allows you to group them by known error warning types, or ignore them outright.

## Usage

Install with your favorite package manager:

```sh
npm install --save-dev jest-clean-console-reporter
```

Configure:

```js
// List known warnings you want to group or suppress.
// (Tip: You can import this from an external file)
const knownWarnings [
  // Group all matching messages and print a summary at the end of the run
  {
    match: /^You are using the simple \(heuristic\) fragment matcher/,
    group: "Apollo: You are using the simple (heuristic) fragment matcher.",
  },
  // Ignore these altogether
  {
    match: /^Heuristic fragment matching going on/,
    group: null,
  },
  {
    match: /^Warning: componentWillMount has been renamed/,
    group: "React deprecation warning: componentWillMount",
  },
  //... more
];

// Add reporters to your jest config
module.exports = {
  // ... your normal jest config
  reporters: [
    // Add jest-clean-console-reporter. This takes place of the
    // default DefaultReporter, and behaves otherwise identically
    ["jest-clean-console-reporter", { rules: knownWarnings }],

    // Overriding config.reporters wipes out default reporters, so
    // we need to restore the summary reporter
    "@jest/reporters/build/SummaryReporter",
  ],
};
```

## Options

Pass options to the reporter in your jest configuration as follows:

```js
const jestConfig = {
  reporters: [
    ["jest-clean-console-reporter", { rules: [/*...*/], levels: [/*...*/]]
    "@jest/reporters/build/SummaryReporter",
  ],
};
```

### `options.rules`

Rules tell the reporter which console messages should be filtered, and how
they should be grouped in the summary. Each rule has two parts, `match` and `format`.

#### `rule.match : RegExp | string | (message, level) => boolean`

`match` is either a regular expression, a string, or a predicate function:

- `RegExp`: Matches console message against this regular expression.
- `string`: Matches console message against this string, first using literal `===` comparison, and falls back to regular expression comparison using `message.match(match)`. The latter is not recommended, but useful if you need to serialize your regular expressions.
- A predicate function that's called with `match(message, level)` where
  - `message` is the full console message
  - `level` is the log level (error, warning, log etc..).
  - To match this message, the predicate may return any truthy value.

Rules are matched in order, from top down. A message that is not matched by any rule will be displayed in the Jest test output as normal.

Matched messages are grouped according to the `group` property:

#### `rule.group : string | null | (message, level, matcher) => string | null

`group` is either a string, a formatter function, or null:

- `string`: Matched messages are grouped by this literal string
- `null`: Matched message is ignored.
- Formatter function that's called with `group(message, level, matcher)` where
  - `message` is the full console message.
  - `level` is the log level (error, warning, log etc..).
  - `matcher` is the original matcher used to match this message. This can be useful if you want to e.g. execute the regular expression for capture groups.
  - The value returned by this function is used as capture key. If the function returns `null`, the message is ignored.

### options.levels

Define which log levels to display in the summary at the end of the test run:

Default: `["error", "warn", "info", "debug", "log"]`

These levels only affect the summary display, and have no effect on whether messages are matched. For that, see [Can I ignore all messages of certain log level?](#can-i-ignore-all-messages-of-certain-log-level).

```js
const knownWarnings = [
  // ...other rules...
  // this rule should be defined last
  { match: (message, level) => level === "log", group: null },
];
```

## Never Asked Questions

Here are some questions nobody has ever asked me, but might be helpful anyway.

### Can I ignore all messages of certain log level?

Yes. Use the second parameter passed to a function macher:

```js
{ match: (message, level) => level === "log", group: null }
```

### Can I ignore all messages, period?

Yes, but it's probably a bad idea:

```js
{ matcher: () => false, group: null }
```

### Can I group multiple error messages into the same bucket?

Yes, just give them the same group key:

```js
[
  {
    match: /^Warning: componentWillMount has been renamed/,
    group: "React componentWill* deprecation warnings",
  },
  {
    match: /^Warning: componentWillReceiveProps has been renamed/,
    group: "React componentWill* deprecation warnings",
  },
];
```

### Can I use this with .json config?

You can use string matchers, which are first compared to the message as literal strings, and failing that, attempted to test against the message as regular expressions:

```json
{
  "match": "Warning: An update to \\w* inside a test was not wrapped in act",
  "group": "An update to (Component) inside a test was not wrapped in act"
}
```

### Can I parameterize log groups using regex capture groups?

You can use the grouping function, where the original matcher is provided as a third argument.

```js
{
    match: /^Warning: An update to (\w*) inside a test was not wrapped in act/,
    group: (message, _level, matcher) => {
      // Note: String.matchAll requires Node 12.x or higher
      const [match] = message.matchAll(matcher);
      return `React: An update to ${match[1]} was not wrapped in act.`;
    }
}
```

### Can I help make this library better?

Yes, see [Contibuting](#contributing).

## Contributing

This software is very much at alpha stage. If you want to help, here are a few things that could be helpful:

- [ ] Write tests
- [ ] Convert project to TypeScript
- [ ] NPM scripts for workflows
- [ ] Provide better summary that shows
- [ ] Known issues
  - [ ] When running Jest with a single file parameter (e.g. `jest src/file.js`), the reporter is not activated

## License

[MIT](LICENSE)