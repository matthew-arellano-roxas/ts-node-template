# TS Node Template

A Node.js API built with **TypeScript**, **Express**, **Nodemon**, **ESLint**, and **Prettier**.

## Prerequisites

- Node.js `^20.19.0`, `^22.13.0`, or `>=24`
- npm

## Quick Start

```bash
npm install
npm run dev
```

The server runs at http://localhost:3000.

## Scripts

| Script                 | Description                             |
| ---------------------- | --------------------------------------- |
| `npm run dev`          | Start the dev server with auto-reload   |
| `npm run build`        | Compile TypeScript to `dist/`           |
| `npm start`            | Run the compiled app (`dist/index.js`)  |
| `npm run lint`         | Check code with ESLint                  |
| `npm run lint:fix`     | Fix ESLint issues automatically         |
| `npm run format`       | Format all files with Prettier          |
| `npm run format:check` | Check formatting without changing files |

## Project Structure

```
my-app/
├── src/
│   └── index.ts          # Express entry point
├── dist/                 # Compiled output (generated)
├── eslint.config.mjs     # ESLint flat config
├── nodemon.json          # Nodemon config
├── tsconfig.json         # TypeScript config
├── .prettierrc           # Prettier rules
├── .prettierignore
├── Dockerfile
├── .dockerignore
└── package.json
```

---

## Copy the repo's Docker files

If you want to start from the exact Docker-related files in this repository, copy the raw files directly below:

- [.gitignore](https://raw.githubusercontent.com/matthew-arellano-roxas/ts-node-template/main/.gitignore)
- [.dockerignore](https://raw.githubusercontent.com/matthew-arellano-roxas/ts-node-template/main/.dockerignore)
- [Dockerfile](https://raw.githubusercontent.com/matthew-arellano-roxas/ts-node-template/main/Dockerfile)

---

## Setting It Up From Scratch

### 1. Initialize the project

```bash
mkdir my-app && cd my-app
npm init -y
git init
```

### 2. Install dependencies

```bash
# runtime
npm install express

# dev
npm install -D typescript ts-node @types/node @types/express \
  eslint @eslint/js typescript-eslint \
  prettier eslint-config-prettier

# install nodemon globally
npm install -g nodemon
```

### 3. TypeScript config

Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "sourceMap": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

### 4. Express app

Create `src/index.ts`:

```ts
import express, { Request, Response } from 'express';

const app = express();
const PORT = process.env.PORT ?? 3000;

app.use(express.json());

app.get('/', (_req: Request, res: Response) => {
  res.json({ message: 'Hello from Express + TypeScript' });
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

### 5. Nodemon config

Create `nodemon.json`:

```json
{
  "watch": ["src"],
  "ext": "ts,json",
  "exec": "ts-node src/index.ts"
}
```

### 6. ESLint config

Create `eslint.config.mjs`:

```js
import { defineConfig, globalIgnores } from 'eslint/config';
import js from '@eslint/js';
import tseslint from 'typescript-eslint';
import prettier from 'eslint-config-prettier';

export default defineConfig([
  globalIgnores(['dist', 'node_modules']),
  {
    files: ['**/*.ts'],
    plugins: { js },
    extends: [js.configs.recommended, tseslint.configs.recommended, prettier],
  },
]);
```

`eslint-config-prettier` is listed last so it turns off any ESLint rules that conflict with Prettier.

### 7. Prettier config

Create `.prettierrc`:

```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100
}
```

Create `.prettierignore`:

```
dist
node_modules
```

### 8. Add scripts to `package.json`

```json
"scripts": {
  "dev": "nodemon",
  "build": "tsc",
  "start": "node dist/index.js",
  "lint": "eslint .",
  "lint:fix": "eslint . --fix",
  "format": "prettier --write .",
  "format:check": "prettier --check ."
}
```

### 9. Add a `.gitignore`

```
node_modules
dist
.env
```

---

## Editor Integration (VS Code)

Install the **ESLint** and **Prettier** extensions, then create `.vscode/settings.json`:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": { "source.fixAll.eslint": "explicit" }
}
```

---

## Docker

The included `Dockerfile` is multi-stage: it installs production dependencies, builds the app, and ships a minimal Alpine runtime image that runs as a non-root user.

```bash
docker build -t my-app .
docker run --rm -p 3000:3000 my-app
```

| Variable     | Default         | Description                    |
| ------------ | --------------- | ------------------------------ |
| `PORT`       | `3000`          | Port the API listens on        |
| `ENTRY_FILE` | `dist/index.js` | File Node runs on start        |
| `NODE_ENV`   | `production`    | Set automatically in the image |

To use a different Node version:

```bash
docker build --build-arg NODE_VERSION=24 -t my-app .
```

---

## Notes

- **Faster startup:** swap `ts-node` for `tsx` (`npm i -D tsx`) and set the nodemon `exec` to `tsx src/index.ts`. It starts quicker and handles ESM more smoothly.
- **Pre-commit checks:** add `husky` and `lint-staged` to run lint and format on staged files.
- **ESLint versions:** if you hit peer-dependency errors between `eslint` and `typescript-eslint`, check the typescript-eslint docs for the supported ESLint version, or pin `eslint@9`.
