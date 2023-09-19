# Creating a fullstack project with express, react, typescript, eslint, Docker

In this guide I am going to teach you how to create a simple web app with the following features:

- Backend with express

- Frontend with react

- Both utilizing typescript, eslint and prettier

- nginx to serve as the webserver

- ssl certificated created with letsencrypt

- deployed as docker containers on aws

Requirements:

- Linux (22.04.3 LTS)

- Docker (24.0.5)

- Node (20.5.1)

Recommended:

- VSCode

## Init

create directory structure:

    root
    ├── services
    │   ├── backend       // Backend root
    │   ├── frontend      // Frontend root
    │   └── nginx
    ├── .vscode
    │   └── settings.json
    └── .gitignore

.gitignore

```
node_modules/
dist/

.env
```

settings.json

```json
    {
      "eslint.workingDirectories": [
        "services/backend",
        "services/frontend"
      ]
    }
```

## Backend

Run inside backend root:

```
npm init -y

npm install express dotenv

npx install-peerdeps --dev eslint-config-airbnb-base

npm i -D concurrently nodemon typescript eslint eslint-config-airbnb-typescript @types/express @types/node @typescript-eslint/eslint-plugin@^6.0.0 @typescript-eslint/parser@^6.0.0

npx tsc --init
```

Add into scripts section of package.json:

```
    "build": "npx tsc",
    "start:prod": "node dist/index.js",
    "start:dev": "concurrently \"npx tsc --watch\" \"nodemon -q dist/index.js\"",
    "lint": "eslint . --ext .js,.jsx,.ts,.tsx",
    "lint:fix": "eslint . --ext .js,.jsx,.ts,.tsx --fix"
```

inside tsconfig.json, uncomment the line outDir and change its value to to ./dist

create directory structure:

```
root
├── src             // backend project
├── .env
├── .eslintignore
└── .eslintrc.json
```

.env

```
PORT=5000
```

.eslintignore

```
node_modules/
dist/
```

.eslintrc.json

```json
{
  "root": true,
  "parser": "@typescript-eslint/parser",
  "plugins": [
    "@typescript-eslint"
  ],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/eslint-recommended",
    "plugin:@typescript-eslint/recommended",
    "airbnb-base",
    "airbnb-typescript/base"
  ],
  "parserOptions": {
    "project": "./tsconfig.json"
  },
  "rules": {
    "no-console": 0
  }
}
```

### Backend structure

Create structure inside src:

    src
    ├── api
    │   ├── health
    │   │   ├── health.controllers.ts
    │   │   ├── health.routes.ts
    │   │   └── index.ts
    │   ├── api.routes.ts
    │   └── index.ts
    ├── app.ts
    ├── config.ts
    └── index.ts

src/api/health/health.controllers.ts

```typescript
import { Request, Response } from 'express';

const get = async (req: Request, res: Response) => {
  res.send('ok');
};

export default { get };
```

src/api/health/health.routes.ts

```typescript
import express from 'express';

import healthControllers from './health.controllers';

const healthRoutes = express.Router();

healthRoutes.get('/', healthControllers.get);

export default healthRoutes;
```

src/api/health/index.ts

```typescript
// eslint-disable-next-line no-restricted-exports
export { default } from './health.routes';
```

src/api/api.routes.ts

```typescript
import express from 'express';

import healthRouter from './health';

const apiRoutes = express.Router();

apiRoutes.use('/health', healthRouter);

export default apiRoutes;
```

src/api/index.ts

```typescript
// eslint-disable-next-line no-restricted-exports
export { default } from './api.routes';
```

src/app.ts

```typescript
import express, { Express } from 'express';

import apiRouter from './api';

const app: Express = express();

app.use('/api', apiRouter);

export default app;
```

src/config.ts

```typescript
import dotenv from 'dotenv';

dotenv.config();

// eslint-disable-next-line prefer-destructuring
export const PORT = process.env.PORT;

export default { PORT };
```

src/index.ts

```typescript
import app from './app';
import { PORT } from './config';

app.listen(PORT, () => {
  console.log(`⚡️[server]: Server is running at http://localhost:${PORT}`);
});
```

### Verify

test commands

    npm run start:dev

    npm run build

    npm run start:prod
