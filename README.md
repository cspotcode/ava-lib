Helper lib to bring suites and concurrency limiting to ava, declare context extensions and typed macros.

Created for ts-node, then extracted to its own library.

test.macro()
Use https://github.com/avajs/ava/commit/fadc6c8a5249e912d0039404e34ab4346a36c025

Unify with our approach?

## supporting teardown, and teardown with serial?

```
test.suite('name', _test => {
  const test = _test.contextAll(t => {
    const service = createService();
    t.teardown(t => {
      
    });
    return {service};
  });

  test('foo', async ({context: {service}}) => {
    
  });
});
```

Every single `contextAll`:
- maintains internal state: list of in-flight tests
- registers teardown CB
- delays teardown CB by at least a single event loop cycle
- if is registered on a serial suite
  - IMPOSSIBLE
  - cannot know when one test case ends if another will run
  - our only hope is to postpone teardown till the end of the file, with a `.serial.after`
- DO NOT REGISTER TEARDOWN TESTCASE because testcases get skipped; break serial execution
