---
layout: post
title: "ESLint v8 에서 v9으로 마이그레이션 하기"
date: "2024-07-18 08:05:44 +0900"
tags: [etc]
---

이 포스트는 제가 사용하고 있던 [nestjs-boilerplate](https://github.com/sweatpotato13/nestjs-boilerplate)에 대해 ESLint v8에서 v9로 마이그레이션을 진행한 내용을 정리한 포스트 입니다.

## ESLint v9

ESLint v9으로 업데이트가 되면서 몇몇 설정이 변경되었습니다. 제일 큰 변경점은 설정 파일이 `.eslintrc` 파일이 `eslint.config.js` 로 변경 되면서 설정 파일의 포맷이 모두 변경되었습니다.

v9가 막 업데이트 되었을 때는 아직 많은 플러그인들이 v9를 지원하지 않아서 업데이트를 하지 않았었는데, 이제는 대부분의 플러그인들이 v9를 지원하고 있어서 업데이트를 진행하였습니다.

![ss](https://i.imgur.com/4oQKVSA.png)

## 마이그레이션 

우선 기존 사용하던 플러그인을 모두 업데이트 해주었습니다. 일부 플러그인이 v9을 지원히지 않아 삭제 혹은 대체한 플러그인도 존재합니다.

```json
"eslint": "^9.7.0",
"eslint-config-prettier": "^9.1.0",
"eslint-plugin-security": "^3.0.1",
"eslint-plugin-simple-import-sort": "^12.1.1",
"typescript-eslint": "8.0.0-alpha.10"
"typescript": "5.4.5",
"@eslint/js": "^9.7.0",
"@types/eslint__js": "^8.42.3",

"@typescript-eslint/eslint-plugin": "^7.11.0", // Removed
"@typescript-eslint/parser": "^7.11.0", // Removed
"eslint-plugin-import": "^12.1.0", // Removed
```

다른 프로젝트를 참고하려고 서칭을 하던 도중 ESLint에서 직접 Migrator를 제공하고 있어서 이를 이용하여 마이그레이션을 진행하였습니다.

[ESLint Configuration Migrator](https://eslint.org/blog/2024/05/eslint-configuration-migrator/)

`.eslintrc` 와 `.eslintignore` 파일을 참조하여 자동으로 설정파일을 만들어 주는 플러그인입니다. 제가 사용하는 설정의 경우 완벽하게 마이그레이션 되진 않았고 일부 수정이 필요한 부분이 있었습니다. 그래도 대부분의 설정이 마이그레이션 되어서 편하게 업데이트를 진행할 수 있었습니다.

아래 코드는 마이그레이션된 설정 파일입니다. 제가 개인적으로 사용하고 있는 설정이어서 다른 프로젝트에 적용하기 위해서는 수정이 필요할 수 있습니다.

```javascript
import { FlatCompat } from "@eslint/eslintrc";
import eslintJs from "@eslint/js";
import pluginSecurity from "eslint-plugin-security";
import simpleImportSortPlugin from "eslint-plugin-simple-import-sort";
import globals from "globals";
import path from "path";
import eslintTs from "typescript-eslint";
import tseslint from "typescript-eslint";
import { fileURLToPath } from "url";

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
const compat = new FlatCompat({
    baseDirectory: __dirname,
    recommendedConfig: eslintJs.configs.recommended
});

export default eslintTs.config(
    eslintJs.configs.recommended,
    ...eslintTs.configs.recommendedTypeChecked,
    ...compat.extends(
        "plugin:@typescript-eslint/eslint-recommended",
        "plugin:@typescript-eslint/recommended",
        "prettier"
    ),
    {
        ignores: ["**/types.d.ts", "**/*.js", "**/node_modules/"]
    },
    {
        languageOptions: {
            globals: {
                ...globals.es2020,
                ...globals.node,
                Atomics: "readonly",
                SharedArrayBuffer: "readonly"
            },
            parserOptions: {
                project: "tsconfig.json"
            }
        },
        settings: {
            "import/resolver": {
                node: {
                    extensions: [".js", ".jsx", ".ts", ".tsx"]
                }
            }
        },

        plugins: {
            "@typescript-eslint": tseslint.plugin,
            "simple-import-sort": simpleImportSortPlugin,
            security: pluginSecurity.configs.recommended,
        },
        rules: {
            "no-empty-pattern": "off",
            "@typescript-eslint/no-unsafe-return": "off",
            "@typescript-eslint/no-unsafe-call": "off",
            "@typescript-eslint/no-unsafe-member-access": "off",
            "@typescript-eslint/no-unsafe-assignment": "off",
            "@typescript-eslint/no-unsafe-argument": "off",
            "@typescript-eslint/no-explicit-any": "off",
            "simple-import-sort/imports": "error",
            "simple-import-sort/exports": "error",
            semi: "error"
        }
    }
);
```

## vscode에 적용

vscode에 ESLint v9의 설정파일을 적용하기 위해서 `eslint.useFlatConfig` 설정을 활성화할 필요가 있습니다.

![vscode](https://i.imgur.com/48YUfn4.png)

위 설정을 활성화하면 `eslint.config.js' 파일을 참고하여 설정을 적용할 수 있습니다.

## 마치며

ESLint v9로 업데이트를 진행하면서 몇몇 설정이 변경되었지만, 대부분의 설정은 그대로 사용할 수 있었습니다. 마이그레이션도 Migrator의 존재 때문에 쉽게 진행할 수 있었고, vscode에서도 설정을 쉽게 적용할 수 있어서 편하게 업데이트를 진행할 수 있었습니다.

지금은 충분히 v9로 마이그레이션을 고려해도 좋을 만큼 충분히 안정화 되었다고 생각합니다.