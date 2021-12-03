# Summary
A Typescript-related problem occurs when a project requires multiple versions of the same package, where one of the versions has a TS declaration file, while the other does not.

In this example, storybook v6.4.4 runs into conflicts with `react-router-dom@5.3.0` since both dependencies require `react-router`, with `react-router@6.0.2` required by the former and `react-router@5.2.1` required by the latter.

However, the manner that berry resolves the dependency causes the typing issues.

# Specs
OSX 11.6  
yarn 3.1.1 (berry)  
node 14.17.0

# Steps to reproduce

```bash
git clone https://github.com/fkurniaw/yarn-types-resolution.git
yarn start
```

The resultant error is as follows when [http://localhost:3000](http://localhost:3000) is opened:
```
Failed to compile
/Users/fkurniaw/yarn-types-resolution/src/App.tsx
TypeScript error in /Users/fkurniaw/yarn-types-resolution/src/App.tsx(20,11):
Property 'user' does not exist on type 'Readonly<Params<{ user?: string | undefined; }>>'.  TS2339

    18 | 
    19 | function Users() {
  > 20 |   const { user } = useParams<{ user?: string }>();
       |           ^
    21 |   return <h2>{`User: ${user}`}</h2>;
    22 | }
    23 |
This error occurred during the build time and cannot be dismissed.
```

When `yarn list react-router-dom` is run, the following dependency tree is shown:
```
yarn why react-router-dom
├─ @storybook/router@npm:6.4.4
│  └─ react-router-dom@npm:6.0.2 (via npm:^6.0.0)
│
├─ @storybook/router@npm:6.4.4 [371ff]
│  └─ react-router-dom@npm:6.0.2 [36ca9] (via npm:^6.0.0 [36ca9])
│
└─ yarn-types-resolution@workspace:.
   └─ react-router-dom@npm:5.3.0 [c8220] (via npm:5.3.0 [c8220])
```

The corresponding `node_modules` structure of the project is as follows:
```
├─ project/node_modules
│  └─ @storybook/router@6.4.4
│  └─ react-router-dom@5.3.0
│     └– node_modules
│        └─ react-router@5.2.1  (no TS declaration files)
│  └─ react-router@6.0.2
│     └– index.d.ts
```

This resolution manner causes issues with typescript, as `react-router-dom@5.3.0` has a dependency on react-router@5.2.1`, while storybook depends on `react-router@6.0.2`

Since `react-router@5.2.1` does not have a TS declaration file but `react-router@6.0.2` does, `typescript` defaults to looking at the root `node_modules` for a typescript declaration file. This raises TS compilation errors as the source code uses `react-router` v5, whose API's have differing method signatures compared to v6.

## Expected Behavior
There should be no typescript errors due to the hoisting of a dependency with multiple versions in the same project.

In the event of a dependency mismatch (e.g. differing major versions), the structure of node_modules should ideally look like the following to avoid the importing of conflicting declaration files:
```
├─ project/node_modules
│  └─ @storybook/router@6.4.4
│     └─ node_modules
│        └─ react-router@6.0.2
│  └─ react-router-dom@5.3.0
│     └─ node_modules
│        └─ react-router@5.2.1
```
