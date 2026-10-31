# typescript-library

Publishable TypeScript library. Exports `hello(name)` returning `` `Hello ${name}!` ``.

## Install

Published to GitHub Packages on every release. Point the scope at GPR once:

```ini
# .npmrc
@continuous-release:registry=https://npm.pkg.github.com
```

then authenticate with a token that can read packages (`GITHUB_TOKEN` works in Actions):

```bash
pnpm add @continuous-release/typescript-library
```

## Use

```ts
import { hello } from '@continuous-release/typescript-library';

console.log(hello('World')); // Hello World!
```

## Develop

```bash
pnpm nx affected --target=build --projects=@continuous-release/typescript-library
pnpm nx affected --target=unit-test --projects=@continuous-release/typescript-library
```

Unit tests live in `tests/unit/*.test.ts` and run with `vitest --dir=tests/unit`.
