# New Watch API not detecting file changes

With TypeScript 2.7 support for the API is being exposed.  [This wonderful PR](https://github.com/TypeStrong/ts-loader/pull/685) by @sheetalkamat to ts-loader added support for the API to ts-loader.

Naturally I'm super-excited about this! We pushed out [v3.3.0](https://github.com/TypeStrong/ts-loader/releases/tag/v3.3.0) of ts-loader including support for the new watch API (which is available in the night builds).

Alas, the watch API functionality doesn't seem to be working as expected.  It's unclear whether the problem is with the watch API or with the implementation in ts-loader.  I found and fixed a [suspected minor issue](https://github.com/TypeStrong/ts-loader/commit/6d55769318841d1d51e74ccf5882c421c2a7a2bb) with the ts-loader watch API support.  However everything else looks correct as best I can judge.

So I'm wondering if this is actually a problem with the watch API itself. It's brand new so that's always possible I guess.

Here is a simple repro of the issue I've seen.

## Seeing the issue

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