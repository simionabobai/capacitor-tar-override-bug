# capacitor-tar-override-bug

Minimal reproduction for a `@capacitor/cli` bug where forcing a patched
version of `tar` (to fix [GHSA-8qq5-rm4j-mr97](https://github.com/advisories/GHSA-8qq5-rm4j-mr97) /
CVE-2026-23745) via npm `overrides` breaks `npx cap add <platform>`.

This is the same root cause reported in
[ionic-team/capacitor#8310](https://github.com/ionic-team/capacitor/issues/8310),
but reproduced here on `@capacitor/cli ^6.0.0` (that issue was filed against 7.x),
confirming the incompatibility spans multiple major versions.

## The problem

`@capacitor/cli`'s `extractTemplate()` (in `dist/util/template.js`) calls
`tar.extract(...)` using the API shape from `tar` v6. `tar` v7 restructured
its exports, so that function no longer exists there, causing:

```
TypeError: Cannot read properties of undefined (reading 'extract')
    at extractTemplate (node_modules/@capacitor/cli/dist/util/template.js:9:25)
    at async runTask (node_modules/@capacitor/cli/dist/common.js:165:23)
    at async addAndroid (node_modules/@capacitor/cli/dist/android/add.js:13:5)
    at async node_modules/@capacitor/cli/dist/tasks/add.js:106:13
```

## Steps to reproduce

```bash
npm install
npx cap init reprodemo com.example.repro --web-dir=.
npx cap add android
```

You should see the `TypeError: Cannot read properties of undefined (reading 'extract')`
error above.

## package.json override causing the break

```json
"overrides": {
  "tar": "^7.5.19"
}
```

Removing this override allows `npx cap add android` to succeed again, but
leaves the `tar` path-traversal vulnerability unpatched.

## Expected behavior

`npx cap add android` should succeed with a patched, non-vulnerable version
of `tar` installed.
