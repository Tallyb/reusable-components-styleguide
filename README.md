# Reusable Components Styleguide

Tips and tricks for making components shareable across different projects (framework agnostic)

Components are now the most popular method of developing frontend applications. The popular frameworks and browsers themselves support splitting applications to individual components. 
Components let you split your UI into independent, reusable pieces, and think about each piece in isolation.  

![tweet](images/trek_tweet.png)

**Why me?**  

This series summarizes what I have learned in the last months working as head of Developer Experience at Bit. Bit is a components collaboration tool that helps developers build components in different applications and repositories and share them.  
During this period, I have seen components developed by many different teams and across different frameworks. The good parts and the bad parts have led me to define some guidelines that can help people build more independent, isolated, and hence reusable components.  

**Table of Contents**

  - [Directory Structure](#directory-structure)
    - [One component -> One directory](#one-component---one-directory)
    - [Use Aliases](#use-aliases)
  - [APIs](#apis)

## Directory Structure

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
, consumers of the components c

Do - Set reasonable defaults.
Avoid - Making the parameters required.
Why? 
Setting parameters makes it easy for the consumer to start using the component, rather than find fair values for all parameters. Once incorporating the component, tweaking it to the exact need is more tranquil. 

