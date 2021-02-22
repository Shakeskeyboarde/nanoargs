# Argser (arg-parser)

A miniscule arguments parser written in Typescript.

## Getting Started

The default export `argser` function accepts an optional args array and a map of option definitions. It returns a map of values parsed from the args array.

```ts
import argser from 'argser';

const args = ['--flag', '--string=foo', '--integer', '123', '--example=1', '-e', '2'];

const [options, err] = argser(args, {
  flag: { value: false },
  string: { value: true },
  integer: { value: parseInt },
  example: { value: parseInt, alias: 'e', many: true },
});
```

The above `options` value would be...

```ts
{
  _: [],
  flag: true,
  string: 'foo',
  integer: 123,
  example: [1, 2],
}
```

### Option Definition Shorthand

For options with only a `value` definition, you can shorten the definition to just the value of the `value` definition.

```ts
argser({
  flag: false, // Same as {} or { value: false }
  string: true, // Same as { value: true }
  integer: parseInt, // Same as { value: parseInt }
});
```

### Errors

An error will be _returned_ in the following cases.

- An undefined option is encountered.
- No value is present for an option which expects a value.

Errors are returned instead of thrown to allow the partially parsed options object to be returned with the error, and to remove the necessity of a try/catch block. Parsing stops when an error occurs, and any remaining arguments (including the error argument) will be added to the options underscore (`_`) array. The returned error will have `arg` and `reason` properties to support custom messaging.

## Commands

The `argser.command` function accepts an optional args array, and a variable number of command names. If the first arg matches one of the command names, it returns the matched command name, and an args array with the command removed.

```ts
import argser from 'argser';

const args = ['foo', '--help'];

const [command, commandArgs] = argser.command(args, 'foo');
```

The above `command` value would be `"foo"`, and the `commandArgs` value would be `["--help"]`.

If no command is matched, `command` will be `undefined`, and `commandArgs` will include all of the original args.

## Process Arguments

When passing in [process.argv](https://nodejs.org/docs/latest/api/process.html#process_process_argv), make sure to remove the first two non-argument values.

```ts
argser(process.argv.slice(2), {
  ...
});

argser.command(process.argv.slice(2), ...);
```

Alternatively, omit the arguments array, in which case the default arguments array is `process.argv.slice(2)`.

```ts
argser({
  ...
});

argser.command(...);
```

## Help/Usage Text

Generating and printing help and usage text isn't an included feature. I recommend simply writing a `usage.ts` file which exports a default usage string value, and then printing the string as necessary. Most terminals will be at least 80 characters wide, so it's a good idea to manually hard wrap at about that width.

```ts
export default `
Usage:  my-cli [options]
        my-cli --version
        my-cli --help

My cool CLI that does cool things.

Options:
  -f, --foo <value>   Foos are the best.
  -b, --bar           Bars are okay too.
  --version           Print the current version.
  --help              Print this help text.
`.trim();
```
