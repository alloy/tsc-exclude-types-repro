This project is a repro for https://github.com/DefinitelyTyped/DefinitelyTyped/issues/33015.

It is a React DOM project, which makes use of StyledComponents. However, `@types/styled-components` depends on `@types/react-native`, which leads to many type clashes between TS’ DOM lib and the polyfills that the ReactNative types provides.

What happens is that `tsc` will by default load all the types it finds in `node_modules/@types/*`, which–due to the aforementioned dependency–includes `@types/react-native`.

A workaround is to be explicit about the types you want to use, by providing a whitelist using the `types` property of `tsconfig.json`:

```json
{
  "compilerOptions": {
    ...
    "types": ["react", "styled-components"]
  }
}
```

However, as you can imagine, this becomes tedious with a larger amount of dependencies.

## Repro

```
yarn start
[…]
node_modules/typescript/lib/lib.dom.d.ts:20050:6 - error TS2300: Duplicate identifier 'XMLHttpRequestResponseType'.

20050 type XMLHttpRequestResponseType = "" | "arraybuffer" | "blob" | "document" | "json" | "text";
           ~~~~~~~~~~~~~~~~~~~~~~~~~~

  node_modules/@types/react-native/globals.d.ts:254:14
    254 declare type XMLHttpRequestResponseType = '' | 'arraybuffer' | 'blob' | 'document' | 'json' | 'text';
                     ~~~~~~~~~~~~~~~~~~~~~~~~~~
    'XMLHttpRequestResponseType' was also declared here.


Found 31 errors.
```

Now uncomment the `types` property in `tsconfig.json` and run again:

```
yarn start
```

## Possible blacklist fix

A very simple suggestion would be to add a `excludeTypes` property to `tsconfig.json`. (See [the patch](./patches/typescript%2B3.9.2.patch).) You can test it like so:

```
yarn patch-package
yarn start
```
