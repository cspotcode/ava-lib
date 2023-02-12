Notes about changes to the lib's behavior and/or API.  Probably outdated at this point, kept because I have a difficult
time throwing stuff away.

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
  - our only hope is to postpone teardown till the end of the file, with a `.serial.after.always`
    - we need to pass it a suitable `context`
- can we trigger serial teardown in a beforeEach that detects when the first non-serial test runs?
  - messy because *all* non-serial tests will need to wait for serial teardown to finish.  If it fails, do those tests swallow the failure?
- DO NOT REGISTER TEARDOWN TESTCASE because testcases get skipped; break serial execution