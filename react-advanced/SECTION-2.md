# Creating the NextJS boilerplate

## Requirements

- node > 10
- IDE
- terminal

## create-next-app

Using the commands from the [docs](https://nextjs.org/docs)

```
npx create-next-app
```

## integrating with typescript

You neeed to create the `tsconfig.json` file first. Run the following command inside the client folder.

```
touch tsconfig.json
```

After it you need to add the typescript dependencies

```
yarn add --dev typescript @types/react @types/node
```

## Configuring the editorconfig

Create the `.editorconfig` file and add the following rules

```
root = true

[*]
indent_style = spaces
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
```

## Configuring the eslint

We need to run the eslint command to start it

```
npx eslint --init
```

Add a plugin to [react hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks) too

```
yarn add eslint-plugin-react-hooks --dev
```

We need to add the following rules to `eslintrc.json` file

```json
  "settings": {
    "react": {
      "version": "detect"
    }
  },

  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn",
    "react/prop-types": "off",
    "react/react-in-jsx-scope": "off",
    "@typescript-eslint/explicit-module-boundary-types": "off"
  }
```

The settings is to eslint knows what react version we are using and the new rules are to avoid the recommend rules from react plugin

## Configuring Prettier with eslint

```js
yarn add --dev prettier eslint-config-prettier eslint-plugin-prettier
```

After it we need to create the `.prettierrc` file andd add some rules

```json
{
  "trailingComma": "none",
  "semi": false,
  "singleQuote": true
}
```

and we need to add prettier inside the `eslintrc.json`

```json
  "extends": ["plugin:prettier/recommended"]
```

## Lint-staged and Husky - Fail fast and early

`yarn add -D husky lint-staged`

add to `package.json`

```json
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "src/**/*.{ts,tsx}": [
      "yarn lint --fix",
      "yarn test --findRelatedTests --bail"
    ]
  },
```

## Jest config

`yarn add --dev jest babel-jest @babel/preset-typescript @types/jest`

adds jest to `eslintrc.json`:

```json
  "env": {
    "browser": true,
    "es2020": true,
    "jest": true,
    "node": true
  },
```

and we create the `jest.config.js` file:

```js
module.exports = {
  testEnvironment: "jsdom",
  testPathIgnorePatterns: ["/node_modules/", "./next/"],
  collectCoverage: true,
  collectCoverageFrom: ["src/**/*.ts(x)"],
  setupFilesAfterEnv: ["<rootDir>/.jest/setup.ts"],
};
```

we need to create the `.babelrc` file to add the babel presets there and with it we can use the last js features inside the test files

```json
{
  "presets": ["next/babel", "@babel/preset-typescript"]
}
```

after it we create the script inside the `package.json`

```json
"test": "jest"
```

## Testing-library

`yarn add -D @testing-library/react @testing-library/jest-dom`

on the `.jest/setup.ts` file we need to import jest-dom

```ts
import "@testing-library/jest-dom";
```

## Adding styled-componets and setting to SSR

`yarn add -D @types/styled-components babel-plugin-styled-components`

add to babel file the folowing plugin

```json
  "plugins": [
    [
      "babel-plugin-styled-components",
      {
        "ssr": true
      }
    ]
  ]
```

after it we need to add a custom Document to Next works well with Styled-components.
So we have to create a `_document.tsx` file inside page folder:

```tsx
import Document, {
  Html,
  Head,
  Main,
  NextScript,
  DocumentContext,
} from "next/document";
import { ServerStyleSheet } from "styled-components";

export default class MyDocument extends Document {
  static async getInitialProps(ctx: DocumentContext) {
    const sheet = new ServerStyleSheet();
    const originalRenderPage = ctx.renderPage;

    try {
      ctx.renderPage = () =>
        originalRenderPage({
          enhanceApp: (App) => (props) =>
            sheet.collectStyles(<App {...props} />),
        });

      const initialProps = await Document.getInitialProps(ctx);
      return {
        ...initialProps,
        styles: (
          <>
            {initialProps.styles}
            {sheet.getStyleElement()}
          </>
        ),
      };
    } finally {
      sheet.seal();
    }
  }

  render() {
    return (
      <Html>
        <Head />
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    );
  }
}
```
