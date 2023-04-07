Const Enums
In most cases, enums are a perfectly valid solution. However sometimes requirements are tighter. To avoid paying the cost of extra generated code and additional indirection when accessing enum values, it’s possible to use const enums. Const enums are defined using the const modifier on our enums:

const enum Enum {
  A = 1,
  B = A * 2,
}
Try
Const enums can only use constant enum expressions and unlike regular enums they are completely removed during compilation. Const enum members are inlined at use sites. This is possible since const enums cannot have computed members.

const enum Direction {
  Up,
  Down,
  Left,
  Right,
}
 
let directions = [
  Direction.Up,
  Direction.Down,
  Direction.Left,
  Direction.Right,
];
Try
in generated code will become

"use strict";
let directions = [
    0 /* Direction.Up */,
    1 /* Direction.Down */,
    2 /* Direction.Left */,
    3 /* Direction.Right */,
];
 
Try
Const enum pitfalls
Inlining enum values is straightforward at first, but comes with subtle implications. These pitfalls pertain to ambient const enums only (basically const enums in .d.ts files) and sharing them between projects, but if you are publishing or consuming .d.ts files, these pitfalls likely apply to you, because tsc --declaration transforms .ts files into .d.ts files.

For the reasons laid out in the isolatedModules documentation, that mode is fundamentally incompatible with ambient const enums. This means if you publish ambient const enums, downstream consumers will not be able to use isolatedModules and those enum values at the same time.
You can easily inline values from version A of a dependency at compile time, and import version B at runtime. Version A and B’s enums can have different values, if you are not very careful, resulting in surprising bugs, like taking the wrong branches of if statements. These bugs are especially pernicious because it is common to run automated tests at roughly the same time as projects are built, with the same dependency versions, which misses these bugs completely.
importsNotUsedAsValues: "preserve" will not elide imports for const enums used as values, but ambient const enums do not guarantee that runtime .js files exist. The unresolvable imports cause errors at runtime. The usual way to unambiguously elide imports, type-only imports, does not allow const enum values, currently.
Here are two approaches to avoiding these pitfalls:

A. Do not use const enums at all. You can easily ban const enums with the help of a linter. Obviously this avoids any issues with const enums, but prevents your project from inlining its own enums. Unlike inlining enums from other projects, inlining a project’s own enums is not problematic and has performance implications. B. Do not publish ambient const enums, by deconstifying them with the help of preserveConstEnums. This is the approach taken internally by the TypeScript project itself. preserveConstEnums emits the same JavaScript for const enums as plain enums. You can then safely strip the const modifier from .d.ts files in a build step.

This way downstream consumers will not inline enums from your project, avoiding the pitfalls above, but a project can still inline its own enums, unlike banning const enums entirely.

Ambient enums
Ambient enums are used to describe the shape of already existing enum types.

declare enum Enum {
  A = 1,
  B,
  C = 2,
}
Try
One important difference between ambient and non-ambient enums is that, in regular enums, members that don’t have an initializer will be considered constant if its preceding enum member is considered constant. By contrast, an ambient (and non-const) enum member that does not have an initializer is always considered computed.

Objects vs Enums
In modern TypeScript, you may not need an enum when an object with as const could suffice:

const enum EDirection {
  Up,
  Down,
  Left,
  Right,
}
 
const ODirection = {
  Up: 0,
  Down: 1,
  Left: 2,
  Right: 3,
} as const;
 
EDirection.Up;
           
(enum member) EDirection.Up = 0
 
ODirection.Up;
           
(property) Up: 0
 
// Using the enum as a parameter
function walk(dir: EDirection) {}
 
// It requires an extra line to pull out the values
type Direction = typeof ODirection[keyof typeof ODirection];
function run(dir: Direction) {}
 
walk(EDirection.Left);
run(ODirection.Right);