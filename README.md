# Clasp without ts2gas

Like https://github.com/google/clasp but without useless typescript transpilations 

## Why ts2gas useless

* ts2gas supports a limited subset of typescript
* Modern js development is almost impossible without the use of a modular system and third-party libraries.
ts2gas does not solve the need to build the application into a single js file. It would seem there is no problem to just use webpack/rollup/your_favorite_bundler, but in this case ts->js transpilation is required before the build step.

## Simple example how to build ts app with rollup

code.ts
```ts
//mark main functions with "export" to protect from rollup tree-shake 
export function hello() {
    Logger.log('hello world')
}
```

tsconfig.json
```json
{
    "compilerOptions": {
        "target": "ES2019",
        "noImplicitUseStrict": true,
        "removeComments": true,
        "checkJs": false,
        "allowJs": false,
        "lib": [
            "ES2019",
        ],
        "types": [
            "google-apps-script"
        ]
    },
    "files": [
        "code.ts"
    ],
    "exclude": [
        "node_modules"
    ]
}
```

rollup.js
```js
import commonjs from '@rollup/plugin-commonjs';
import resolve from '@rollup/plugin-node-resolve';
import ts from 'rollup-plugin-ts';

export default {
    input: './code.ts',
    strictDeprecations: true,
    external: [],
    output: {
        //mark "export" syntax more detectable
        generatedCode: { objectShorthand: true, constBindings: true },
        externalLiveBindings: false,
        minifyInternalExports: false,
        freeze: false,
        inlineDynamicImports: true,
        format: 'es',
        file: 'code.gs',
        sourcemap: false
    },
    plugins: [
        resolve({ browser: true, preferBuiltins: false }),
        commonjs({ sourceMap: false }),
        ts({
            browserslist: false,
            include: '*.ts'
        }),
        {
            generateBundle(_, files, isWrite) {
                if (isWrite) {
                    for (let f in files) {
                        const i = files[f];
                        if (i.type === 'chunk' && i.exports.length) {
                            //remove invalid for gscript "export" syntax
                            i.code = i.code.replace(`export { ` + i.exports.join(`, `) + ` };`, '')
                        }
                    }
                }
            }
        }
    ]
}
```

.claspignore ( WARNING!!! syntax not same as .gitignore. Folders like "node_modules" not supported - use "node_modules/**" )
```
**/**

!appsscript.json
!*.gs
```