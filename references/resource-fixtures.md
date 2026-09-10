# Fixture ownership

Allocate resources inside the schedulable case and register their cleanup at allocation.
Related tabs/devices belong to one case when their shared state is the behavior.
Unrelated tests must not reset that state. A closed, immutable database template may
be copied; copying a live WAL directory does not establish a consistent snapshot.

This Node example holds the OS-selected port with the actual HTTP server. It neither
reserves and releases a guessed future port nor kills a process occupying one.

```js
import test from 'node:test';
import assert from 'node:assert/strict';
import {createServer} from 'node:http';
import {once} from 'node:events';
import {mkdtemp, readFile, writeFile, rm} from 'node:fs/promises';
import {tmpdir} from 'node:os';
import {join} from 'node:path';

test('capture is readable from the owned store', async t => {
  const directory = await mkdtemp(join(tmpdir(), 'capture-case-'));
  t.after(() => rm(directory, {recursive:true, force:true}));
  const file = join(directory, 'capture.txt');
  await writeFile(file, 'A retained thought');
  const server = createServer(async (_request, response) => {
    response.end(await readFile(file));
  });
  t.after(async () => {
    if (server.listening) await new Promise((done, reject) =>
      server.close(error => error ? reject(error) : done()));
  });
  server.listen(0, '127.0.0.1');
  await once(server, 'listening');
  const response = await fetch(`http://127.0.0.1:${server.address().port}`);
  assert.equal(await response.text(), 'A retained thought');
});
```

Use the repository's equivalent fixture rather than copying this HTTP example into
every test. Playwright fixtures wrap `await use(resource)` in `try/finally`;
Vitest fixtures do the same. Keep mutable resources test-scoped. Register child
process shutdown immediately, await native exit, then close stores and remove
temporary data. Preserve failed reports before removing their fixture directory.

For a race, hold the exact write/acknowledgment promise, wait until both clients reach
the boundary, release it, and assert both intermediate and durable final records.
For timeout/cancellation, keep the timer because time is the subject; assert the
owned descendant has exited and dependent work never ran. Incidental sleeps do
not establish transaction completion.

Use run-owned paths for reports and output, including focused invocations. Reuse a
server only when its mutable partitions and teardown ownership are explicit.
See [Playwright fixtures](https://playwright.dev/docs/test-fixtures) and
[Node test cleanup](https://nodejs.org/api/test.html#afterfn-options).
