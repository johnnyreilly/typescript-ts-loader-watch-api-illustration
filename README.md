# New Watch API issues illustration

With TypeScript 2.7 support for the API is being exposed.  [This wonderful PR](https://github.com/TypeStrong/ts-loader/pull/685) by @sheetalkamat to ts-loader added support for the API to ts-loader.

Naturally I'm super-excited about this! We pushed out [v3.3.0](https://github.com/TypeStrong/ts-loader/releases/tag/v3.3.0) of ts-loader including support for the new watch API (which is available in the night builds).

Alas, the watch API functionality doesn't seem to be working as expected.  It's unclear whether the problem is with the watch API or with the implementation in ts-loader.  I found and fixed a [suspected minor issue](https://github.com/TypeStrong/ts-loader/commit/6d55769318841d1d51e74ccf5882c421c2a7a2bb) with the ts-loader watch API support.  However everything else looks correct as best I can judge.

There's 2 issues I've spotted:
- The new watch API is not detecting file changes
- The new watch API does not find the TypeScript APIs (`lib.dom.d.ts` etc) - if you use `lib` in your `tsconfig.json` there are problems.

I'm wondering if this is actually a problem with the watch API itself.

Here is a simple repro of the issues I've seen.

## Not detecting file changes

1. Clone this repro
2. `yarn install`
3. run either `yarn start` (to use `webpack-dev-server`) or `yarn watch` (to use `webpack --watch`)
4. See the following in the console:

```
ERROR in ./src/render.ts
[tsl] ERROR in C:\source\typescript-ts-loader-watch-api-illustration\src\render.ts(2,5)
      TS2304: Cannot find name 'i'.

ERROR in ./src/render.ts
[tsl] ERROR in C:\source\typescript-ts-loader-watch-api-illustration\src\render.ts(2,7)
      TS2304: Cannot find name 'am'.

ERROR in ./src/render.ts
[tsl] ERROR in C:\source\typescript-ts-loader-watch-api-illustration\src\render.ts(2,10)
      TS2304: Cannot find name 'an'.

ERROR in ./src/render.ts
[tsl] ERROR in C:\source\typescript-ts-loader-watch-api-illustration\src\render.ts(2,13)
      TS2552: Cannot find name 'error'. Did you mean 'Error'?
webpack: Failed to compile.
```

5. comment out line 2 in `render.ts` (the error) and save the file
6. note webpack rebuilds but TypeScript continues to report the same errors; it fails to acknowledge the file change.

If you amend the `webpack.config.js` to set `experimentalWatchApi: false` and repeat steps 3-6 then you will see that behaviour is as desired.  But this is **without** using the new watch API.

## Usage of `lib` in `tsconfig.json` does not work

1. comment out line 2 in `render.ts` (the error) and save the file
2. Uncomment the `lib` section in your `tsconfig.json` so it looks like this:

```
{
    "compilerOptions": {
        "sourceMap": true,
        "lib": [
            "dom",
            "es2015"
        ],
    }
  }
```

3. run `yarn watch`
4. See the following in the console:

```
C:\source\typescript-ts-loader-watch-api-illustration [master ↑2 +0 ~3 -0 !]> yarn watch
yarn run v1.3.2
$ webpack --watch

Webpack is watching the files…

ts-loader: Using typescript@2.7.0-dev.20180120 and C:\source\typescript-ts-loader-watch-api-illustration\tsconfig.json
Using watch api
Hash: cec32e52daf1e2bcf519
Version: webpack 3.10.0
Time: 671ms
        Asset     Size  Chunks             Chunk Names
dist/index.js  6.85 kB       0  [emitted]  main
   [0] ./src/index.ts 89 bytes {0} [built]
   [1] ./src/render.ts 164 bytes {0} [built] [1 error]

ERROR in ./src/render.ts
[tsl] ERROR in C:\source\typescript-ts-loader-watch-api-illustration\src\render.ts(3,5)
      TS2304: Cannot find name 'document'.

ERROR in C:\source\typescript-ts-loader-watch-api-illustration\tsconfig.json
[tsl] ERROR
      TS6053: File 'C:/lib.dom.d.ts' not found.

ERROR in C:\source\typescript-ts-loader-watch-api-illustration\tsconfig.json
[tsl] ERROR
      TS6053: File 'C:/lib.es2015.d.ts' not found.

```