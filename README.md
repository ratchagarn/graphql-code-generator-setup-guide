# GraphQL Code Generator Setup Guide

Step by step for setup [GraphQL Code Generator](https://www.graphql-code-generator.com/) for [Apollo Client React](https://www.apollographql.com/docs/react/) with [TypeScript](https://www.typescriptlang.org/) and [VSCode](https://code.visualstudio.com/)

These guide focusing on you don't have GraphQL server on your local environment and need to set authorization for access GraphQL schema on server

### Table of contents

1. [Prepare directories](#1-prepare-directories)
2. [Install packages](#2-install-packages)
3. [Create configuration files](#3-create-configuration-files)
4. [Update package scripts](#4-update-package-scripts)
5. [Add your Query documents](#5-add-your-query-documents)
6. [Install VSCode Extension](#6-install-vscode-extension)
7. [Create apollo.config.js](#7-create-apolloconfigjs)
8. [Update .gitignore](#8-update-gitignore)
9. [Example Usages](#9-example-usages)

---

### 1) Prepare directories

Create directories for Query documents and files generate

- `./src/graphql/documents`
- `./src/graphql/generate`

---

### 2) Install packages

```
yarn add \
  graphql \
  @graphql-codegen/cli \
  @graphql-codegen/add \
  @graphql-codegen/schema-ast \
  @graphql-codegen/typescript \
  @graphql-codegen/typescript-operations \
  @graphql-codegen/typescript-react-apollo \
  dotenv
```

---

### 3) Create configuration files

Create these files at your root directory

#### `env.local`

For setup your envoroment variables

```
CODEGEN_AUTH_TOKEN=YOUR_AUTH_TOKEN
```

Replace `YOUR_AUTH_TOKEN` with your `token` for authentication on your GraphQL server

#### `codegen.json`

Configuration for download GraphQL schema from server (If you don't have GraphQL Server in Local)

```json
{
  "schema": [
    {
      "YOUR_GRAPHQL_SERVER_ENDPOINTS": {
        "headers": {
          "Authorization": "${CODEGEN_AUTH_TOKEN}"
        }
      }
    }
  ],
  "generates": {
    "./src/graphql/generate/schema.graphql": {
      "plugins": [
        "schema-ast"
      ]
    }
  }
}
```

Replace `YOUR_GRAPHQL_SERVER_ENDPOINTS` with your GraphQL server endpoints

#### `codegen.yaml`

Configuration for generate operations (TypeScript types, Apollo custom hooks with type safety)

```yaml
schema: './src/graphql/generate/schema.graphql'
documents: './src/graphql/documents/**/*.gql'
generates:
  ./src/graphql/generate/operations.ts:
    plugins:
      - add:
          content: '// ***** THIS FILE IS GENERATED, DO NOT EDIT! *****'
      - typescript
      - typescript-operations
      - typescript-react-apollo
    config:
      preResolveTypes: true
      skipTypeNameForRoot: true
      skipTypename: true
      withHooks: true
      withMutationFn: true

```

If you need `Typename` you can set both `skipTypeNameForRoot` and `skipTypename` to `false`

---

### 4) Update package scripts

Add these scripts at your `package.json` scripts

```json
"generate:schema": "DOTENV_CONFIG_PATH=./.env.local graphql-codegen -r dotenv/config --config codegen.json",
"generate:operations": "graphql-codegen --config codegen.yaml",
"generate:all": "yarn generate:schema && yarn generate:operations --watch=false"
```

---

### 5) Add your Query documents

Query documents should be stored at `./src/graphql/documents` at least one file for code generator

Example:

`./src/graphql/documents/users.gql`

```gql
query users {
  users {
    id
    name
  }
}
```

Run generate command

```
yarn generate:all
```

If everythings work properly, you should see generate files at `./src/graphql/generate/`

- `./src/graphql/generate/operations.ts`
- `./src/graphql/generate/schema.graphql`

When development you just need to run

```
yarn generate:operations --watch
```

You need to open `codegen.yaml` then just save it without any change to trigger update when you update Query documents

---

### 6) Install `VSCode` Extension

[Apollo GraphQL Extension](https://marketplace.visualstudio.com/items?itemName=apollographql.vscode-apollo)

[YAML Extension](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) (for `codegen.yaml` code suggestion - *Optionally*)

---

### 7) Create `apollo.config.js`

```js
module.exports = {
  client: {
    service: {
      name: 'your-service-name',
      localSchemaFile: './src/graphql/generate/schema.graphql',
    },
    addTypename: false,
    includes: ['./src/graphql/documents/**/*.gql'],
    excludes: ['./src/graphql/generate/**/*.ts', '**/__tests__/**'],
  },
}
```

If you need `Typename` set `addTypename` to `true`

---

### 8) Update `.gitignore`

Add this ignore to your `.gitignore`

```
env.local
/src/graphql/generate/*
!/src/graphql/generate/schema.graphql
```

We need to ignore `.env.local` for personal configuration and `schema.graphql` for developer in the team to make sure we use same GraphQL schema with other team member\
_**(please don't edit generate files manually, those files will created by code generator after run the command)**_

---

### 9) Example Usages

```tsx
import type { FC } from 'react'

import { useUsersQuery } from 'graphql/generate/operations'

const UserList: FC = () => {
  const query = useUsersQuery({
    fetchPolicy: 'no-cache',
  })
  
  if (query.loading) {
    return <div>Loading...</div>
  }
  
  if (query.error) {
    return <div>Query ERROR!</div>
  }

  return query.data?.users.map((user) => (
    <div key={user.id}>{user.name}</div>
  ))
}

export default UserList
```

Have a good day 🚀
