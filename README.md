# Reusable Components Styleguide

__Tips and tricks for making components shareable across different projects (framework agnostic).__

Components are now the most popular method of developing frontend applications. The popular frameworks and browsers themselves support splitting applications to individual components.  
Components let you split your UI into independent, reusable pieces, and think about each piece in isolation.  

![tweet](images/trek_tweet.png)

**Why me?**  

This series summarizes what I have learned in the last months working as head of Developer Experience at Bit. Bit is a components collaboration tool that helps developers build components in different applications and repositories and share them.  
During this period, I have seen components developed by many different teams and across different frameworks. The good parts and the bad parts have led me to define some guidelines that can help people build more independent, isolated, and hence reusable components.  

**Table of Contents**

- [Reusable Components Styleguide](#reusable-components-styleguide)
  - [Directory Structure](#directory-structure)
    - [One component -> One directory](#one-component---one-directory)
    - [Use Aliases](#use-aliases)
  - [APIs](#apis)
    - [Use Enums](#use-enums)
    - [Set Defaults](#set-defaults)
  - [Globals](#globals)
    - [Do not rely on global variables](#do-not-rely-on-global-variables)
    - [Provide fallbacks to globals](#provide-fallbacks-to-globals)
  - [NPM Packages](#npm-packages)
    - [Ensure versions compatibility](#ensure-versions-compatibility)
    - [Minimize packages](#minimize-packages)

## Directory Structure

Producer - create a directory structure that is easily consumable
Consumer - Ensure you get all parts of the component when consuming it

### One component -> One directory

✅ _Do_: Put all the component related files in a single directory, including component's code, stylings, tests, documentation, storybook stories, and testing snapshots. Sub-components (components that you can only use within the context of another component, such as List and ListItem), can also be included in the same directory.  

```bash
├── Button.spec.tsx
├── Button.stories.tsx
├── Button.style.tsx
├── Button.tsx
└── __snapshots__
    └── Button.spec.tsx.snap

1 directory, 5 files
```

❌ _Avoid_: Placing files according to their type in different directories.

```bash
├── components
│   ├── Button.style.tsx
│   └── Button.tsx
├── stories
│   └── Button.stories.tsx
└── tests
    ├── Button.spec.tsx
    └── __snapshots__
        └── Button.spec.tsx.snap

4 directories, 5 files
```  

❔**Why?**
Placing all the files related to a component makes it easier to reason about the items that are interconnected. File references are becoming shorter and easier to find the referenced item. Shorter file references make it easy to move the component to different directories. A typical reference pattern is:  
style <- code <- story <- test
The component code is importing the component's style (CSS or JS in CSS style). A story (supporting CSF format) is importing the code to build the story. And the test is importing and instancing a story to validate functionality. (It is also totally ok for the test is directly importing the code).  

### Use Aliases

✅ _Do_: Reference other components using aliases.

```js
import { } from '@utils'
```

❌ _Avoid_: using relative pathname to other components.

```js
import { } from '../../../src/utils';
```

❔**Why?**
Having a path like the above makes it hard to move the component files around in our project, as we need to keep the reference valid. Using backward relative paths couples the component to the specific file structure of the project and forces the consuming project to use a similar structure.
Webpack, Rollup, and Typescript provide methods for using fixed references instead of relative ones. Typescript uses the `paths` mapping to create a mapping to reference components. Rollup uses the `@rollup/plugin-alias` for creating similar aliases, and Webpack is using the setting of `resolve.alias` for the same purpose.

## APIs

Component's APIs are the data attributes it receives and the callbacks it exposes. The general rule is to try and minimize the APIs surface to the necessary minimum.  

Producer - prepare the APIs so they are logical and consistent for consumers to use
Consumer - receive flexible but simple to use APIs that cover most needs

### Use Enums

✅ _Do_: Use Enums instead of multiple booleans

```ts
type LocationProps = {
  position: 'TopLeft' | 'TopRight' | 'BottomLeft' | 'BottomRight'
}
```

❌ _Avoid_: using multiple booleans

```ts
type LocationProps = {
  isLeft: string,
  isTop: string,
}
```

❔**Why?**
Interdependencies between parameters make it harder for the consumer to use it. Creating more simplistic params paves a smoother way for developers to consume the components.  

### Set Defaults

✅ _Do_: Set reasonable defaults for most params

```ts
type LocationProps = {
  position: 'TopLeft' | 'TopRight' | 'BottomLeft' | 'BottomRight'
}

defaultProps = {
  position: 'TopLeft'
}
```

❌ _Avoid_: Making parameters required and expect user to fill in values for all of them.

❔**Why?**

Setting parameters makes it easy for the consumer to start using the component, rather than find fair values for all parameters. Once incorporating the component, tweaking it to the exact need is more tranquil. 

## Globals

### Do not rely on global variables

✅ _Do_: get globals in the component's APIs instead of accessing a global param

```js
export const Card = ({ title, paragraph, someGlobal }: CardProps) =>
<aside>
  <h2>{ title }</h2>
  <p>
    { paragraph + someGlobal.value }
  </p>
</aside>
```

❌ _Avoid_: Assuming a value

Components may rely on globals, such as window.someGlobal, assuming that the global variable already exists.  

❔**Why?**

Relying on parameters gives the consuming application greater flexibility in using the components and does not require it to adhere to the same structure that exists in the producing application.

### Provide fallbacks to globals

✅ _Do_: Use reasonable defaults when accessing globals that may not exist

```js
if (typeof window.someGlobal === 'function') {
  window.someGlobal()
} else {
  // do something else or set the global variable and use it
}
```

❌ _Avoid_: accessing global with no safe fallback

```js
window.someGlobal()
```

❔**Why?**
Fallbacks let the consuming application a way to build the application in a manner that is less coupled to the way the provider application. It also does not assumes that the global was set at the time it is consumed.  

## NPM Packages

Our code relies on third-party libraries for providing specific functionalities, such as scrolling, charting, animations, and more. Third-party libraries are important but take care when adding them.

### Ensure versions compatibility

✅ _Do_: Define packages that are likely to exist in the consuming app as peerDependency with relaxed versioning.  

"peerDependencies": {
    "my-lib": ">=1.0.0"
}

❌ _Avoid_: specifying very strict version as dependency

"dependencies": {
    "my-lib": "1.0.0"
}

❔**Why?**
To understand the problem, let's understand how package managers resolve dependencies. Assume we have two libraries with the following package.json files: 

```json
{
  "name": "library-a",
  "version": "1.0.0",
  "dependencies": {
    "library-b": "^1.0.0",
    "library-c": "^1.0.0"
  }
}

{
  "name": "library-b",
  "version": "1.0.0",
  "dependencies": {
    "library-c": "^2.0.0"
  }
}
```

The resulting node_modules tree will look as follow:  

```bash
- library-a/
  - node_modules/
    - library-c/
      - package.json <-- library-c@1.0.0
    - library-b/
      - package.json
      - node_modules/
        - library-c/
          - package.json <-- library-c@2.0.0
```

You can see that library-c is installed twice with two separate versions. In some cases, such as with React or Angular frameworks, this can even cause errors. However, if the configuration is kept as follow:  

```json
{
  "name": "library-a",
  "version": "1.0.0",
  "dependencies": {
    "library-b": "^1.0.0",
  },
  "peerDependencies": {
    "library-c": ">=1.0.0"
  },
}

{
  "name": "library-b",
  "version": "1.0.0",
  "dependencies": {
    "library-c": "^2.0.0"
  }
}
```

library-c is only installed once:

```bash
- library-a/
  - node_modules/
    - library-c/
      - package.json <-- library-c2@.0.0
    - library-b/
      - package.json
```

### Minimize packages

✅ _Do_: Revise package.json dependencies often to make sure they are all in use. Prefer language features over packages (e.g. lodash vs. built it functions).  

❌ _Avoid_: Using functionality duplicated across multiple packages.  

❔**Why?**
When reusing code, you also need to reuse the packages that use it. Relying on multiple packages makes it hard to move components between applications, but also increase bundle size for all the component consumers. 
