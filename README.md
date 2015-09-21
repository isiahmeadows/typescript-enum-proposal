# TypeScript Enum generalization proposal

(See [this bug](https://github.com/Microsoft/TypeScript/issues/1206))

-----

These are all amendments to [the enum part of the spec](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#9-enums).

## Enum Declaration

The enum syntax would be extended as follows, where *EnumName* is the enum's name, *Type* is the enum's type, and *EnumMembers* are the members and associated values of the enum type.

&emsp;&emsp;*EnumDeclaration:*
&emsp;&emsp;&emsp;`enum`&emsp;*EnumName*&emsp;`{`&emsp;*EnumBody<sub>opt</sub>*&emsp;`}`
&emsp;&emsp;&emsp;`enum`&emsp;*EnumName*&emsp;`:`&emsp;*Type*&emsp;`{`&emsp;*EnumBody<sub>opt</sub>*&emsp;`}`

&emsp;&emsp;*EnumName:*
&emsp;&emsp;&emsp;*Identifier*

An enum type of the form above is a subtype of *Type*, and implicitly declares a variable of the same name, with its type being an anonymous object containing all the type's names as keys and *Type* as the value type of each of them. Moreover, if *Type* is the Number or Symbol primitive types, then the object's signature includes a numeric index signature with the signature `[x: number]: string` or `[x: symbol]: string`, respectively. In the first variation, *Type* is inferred to be the type of the *EnumValue* of the first *EnumEntry* of *EnumBody* if it has an *EnumValue* (i.e. it's initialized), or the Number primitive type otherwise. In the second variation, *Type* is explicitly given.

&emsp;&emsp;*EnumDeclaration:*
&emsp;&emsp;&emsp;`const`&emsp;*EnumBody<sub>opt</sub>*&emsp;`enum`&emsp;*EnumBody<sub>opt</sub>*&emsp;*EnumName*&emsp;*EnumBody<sub>opt</sub>*&emsp;`{`&emsp;*EnumBody<sub>opt</sub>*&emsp;`}`
&emsp;&emsp;&emsp;`const`&emsp;*EnumBody<sub>opt</sub>*&emsp;`enum`&emsp;*EnumBody<sub>opt</sub>*&emsp;*EnumName*&emsp;*EnumBody<sub>opt</sub>*&emsp;`:`&emsp;*EnumBody<sub>opt</sub>*&emsp;*PrimitiveEnumType*&emsp;*EnumBody<sub>opt</sub>*&emsp;`{`&emsp;*EnumBody<sub>opt</sub>*&emsp;`}`

&emsp;&emsp;*PrimitiveEnumType:*
&emsp;&emsp;&emsp;`boolean`
&emsp;&emsp;&emsp;`string`
&emsp;&emsp;&emsp;`number`

An enum type of the form above is a subtype of *PrimitiveEnumType*. It declares a variable of the same name, with its type being an anonymous object containing all the type's names as keys and *PrimitiveEnumType* as the value type of each of them. It is said to also be a constant enum type. In the first variation, *Type* is inferred to be the type of the *EnumValue* of the first *EnumEntry* of *EnumBody* if it has an *EnumValue* (i.e. it's initialized), or the Number primitive type otherwise. In the second variation, *Type* is explicitly given. For constant enum types, for the sake of simplicity below, *Type* references *PrimitiveEnumType*.

The example

```ts
enum Color: string { Red, Green, Blue }
```

declares a subtype of the String primitive type called `Color`, and introduces a variable 'Color' with a type that corresponds to the declaration

```ts
var Color: {
    [x: string]: string;  
    Red: Color;  
    Green: Color;  
    Blue: Color;  
};
```

The example

```ts
enum Color: Type { Red, Green, Blue }
```

declares a subtype of the type `Type` called `Color`, and introduces a variable 'Color' with a type that corresponds to the declaration

```ts
var Color: {
    Red: Color;  
    Green: Color;  
    Blue: Color;  
};
```

## Enum Members

Each enum member has an associated value of *Type* specified by the enum declaration.

&emsp;&emsp;*EnumBody:*
&emsp;&emsp;&emsp;*EnumMemberList*&emsp;`,`*<sub>opt</sub>*

&emsp;&emsp;*EnumMemberList:*
&emsp;&emsp;&emsp;*EnumMember*<br>
&emsp;&emsp;&emsp;*EnumMemberList*&emsp;`,`&emsp;*EnumMember*

&emsp;&emsp;*EnumMember:*
&emsp;&emsp;&emsp;*PropertyName*<br>
&emsp;&emsp;&emsp;*PropertyName*&emsp;`=`&emsp;*EnumValue*

&emsp;&emsp;*EnumValue:*
&emsp;&emsp;&emsp;*AssignmentExpression*

If in an ambient context: 

- An error occurs if any *EnumMember* of *EnumMemberList* has an *EnumValue*.
- Skip the rest of this section, as it does not apply.

If *Type* is explicitly given, an error occurs if a given *EnumValue* of any *EnumMember* of *EnumMemberList* is not of type *Type*.

For each *EnumMember*, if *EnumValue* is not given, then *EnumValue* is defined in the first of the following to apply:

1.  If *Type* is the String primitive type, then let *EnumValue* be *PropertyName*.
2.  Else, if *Type* is the Number primitive type, then:
    1.  If the member is the first in the declaration, then let *EnumValue* be the primitive number 0.
    2.  Else, if the previous member's *EnumValue* can be classified as a constant numeric enum member, then let *EnumValue* be the *EnumValue* of the previous member plus one.
    3.  Else, an error occurs.
3.  Else, if *Type* is the Symbol primitive type, then let *EnumValue* be *PropertyName*.
4.  Else, if the constructor for *Type* accepts a single parameter of the String primitive type, then let *EnumValue* be a newly constructed instance of *Type* with a sole argument *PropertyName*.

    *Non-normative:*

    In other words, *Type*'s constructor must implement

    ```ts
    interface TypeConstructorWithString {
        new (value: string): <Type>;
    }
    ```

5.  Else, if the constructor for *Type* accepts a single parameter of the Number primitive type, then:
    1.  If the member is the first in the declaration, then let *EnumValue* be let *EnumValue* be a newly constructed instance of *Type* with a sole argument of the primitive number 0.
    2.  Else, if the previous member's *EnumValue* is a newly constructed instance of *Type*, and the constructor is called with a sole argument that can be classified as a constant numeric enum member, then let *EnumValue* be that constructor call's argument plus one.
    3.  Else, an error occurs.
    
    *Non-normative:*
    
    In other words, *Type*'s constructor must implement
    
    ```ts
    interface TypeConstructorWithNumber {
        new (value: number): <Type>;
    }
    ```
    
6.  Else, if the constructor for *Type* accepts zero parameters, then let *EnumValue* be a newly constructed instance of *Type*, with zero arguments.

    *Non-normative:*

    In other words, *Type*'s constructor must implement

    ```ts
    interface TypeConstructorWithString {
        new (): <Type>;
    }
    ```

7.  Else, an error occurs.

A few examples:

*EnumValue* for `Foo` is the number 0.

```ts
enum E {
    Foo = 0,
}
```

*EnumValue* for `Foo` is the string "Foo".

```ts
enum E: string {
    Foo = "Foo",
}
```

*EnumValue* for `Foo` is a symbol wrapping the string "Foo".

```ts
enum E: symbol {
    Foo = Symbol("Foo"),
}
```

*EnumValue* for `Foo` is the number 0.

```ts
enum E {
    Foo,
}
```

*EnumValue* for `Foo` is the string "Foo".

```ts
enum E: string {
    Foo,
}
```

*EnumValue* for `Foo` is `new Type("Foo")`.

```ts
class Type {
    constructor(public value: string);
}

enum E: Type {
    Foo,
}
```

*EnumValue* for `Foo` is `new Type()`.

```ts
class Type {
    constructor();
}

enum E: Type {
    Foo,
}
```

An enum member is classified as follows:

- If the member declaration specifies no value, and *Type* is either the String primitive type or the Number primitive type, the member is considered a constant enum member, inferred according to their member name.
- If *Type* is the Boolean primitive type, and the member declaration specifies a value that can be classified as a constant boolean enum expression (as defined below), the member is considered a constant enum member.
- If *Type* is the Number primitive type, and the member declaration specifies a value that can be classified as a constant numeric enum expression (as defined below), the member is considered a constant enum member.
- If *Type* is the String primitive type, and the member declaration specifies a value that can be classified as a constant string enum expression (as defined below), the member is considered a constant enum member.
- Otherwise, the member is considered a computed enum member.

A ***constant enum expression*** is a constant boolean enum expression, a constant numeric enum expression, or a constant string enum expression.

A ***constant boolean enum expression*** is a subset of the expression grammar that can be evaluated fully at compile time, and returns only booleans. A constant boolean enum expression is one of the following:

- A boolean literal
- An identifier or property access that denotes a previously declared member in the same constant enum declaration, if *Type* is the Boolean primitive type.
- A parenthesized constant boolean enum expression.
- A `!` unary operator applied to a constant enum expression.

A ***constant numeric enum expression*** is a subset of the expression grammar that can be evaluated fully at compile time, and returns only numbers. An expression is considered a constant numeric enum expression if it is one of the following:

- A numeric literal.
- An identifier or property access that denotes a previously declared member in the same constant enum declaration, if *Type* is the Number primitive type.
- A parenthesized constant numeric enum expression.
- A `+`, `–`, or `~` unary operator applied to a constant enum expression.
- A `+` operator applied to two constant numeric enum expressions.
- A `–`, `*`, `/`, `%`, `<<`, `>>`, `>>>`, `&`, `^`, or `|` operator applied to two constant enum expressions.

A ***constant string enum expression*** is a subset of the expression grammar that can be evaluated fully at compile time, and returns only strings. An expression is considered a constant numeric enum expression if it is one of the following:

- A string literal.
- A template string literal whose expressions are all constant enum expressions.
- An identifier or property access that denotes a previously declared member in the same constant enum declaration, if *Type* is the String primitive type.
- A parenthesized constant string enum expression.
- A + operator applied to two constant enum expressions, one of which is a constant string enum expression.

## Proposed emit

[Section 9.1](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#95-code-generation), applying to non-constant enums only, is amended to the following:

```js
var <EnumName>;
(function (<EnumName>) {
    <EnumMemberAssignments>  
})(<EnumName>||(<EnumName>={}));
```

where *EnumName* is the name of the enum, and *EnumMemberAssignments* is a sequence of assignments, one for each enum member, in order they are declared. *EnumMemberAssignments* is defined in the first applicable section below.

1. If *Type* is the Number or Symbol primitive types, let the following be *EnumMemberAssignments*.

    ```js
    <EnumName>[<EnumName>["<PropertyName>"] = <EnumValue>] = "<PropertyName>";
    ```

2. If *Type* is of any other type, let the following be *EnumMemberAssignments*.

    ```js
    <EnumName>["<PropertyName>"] = <Value>;
    ```

### Examples

For either of these sources,

```ts
enum Color { Red, Green, Blue }
enum Color: number { Red, Green, Blue }
```

the following should be emitted:

```js
var Color;
(function (Color) {
    Color[Color["Red"] = 0] = "Red";
    Color[Color["Green"] = 1] = "Green";
    Color[Color["Blue"] = 2] = "Blue";
})(Color||(Color={}));
```

For this source,

```ts
enum Color: string { Red, Green, Blue }
```

the following should be emitted:

```js
var Color;
(function (Color) {
    Color["Red"] = "Red";
    Color["Green"] = "Green";
    Color["Blue"] = "Blue";
})(Color||(Color={}));
```

For this source,

```ts
enum Color: symbol { Red, Green, Blue }
```

the following should be emitted:

```js
var Color;
(function (Color) {
    Color[Color["Red"] = Symbol("Red")] = "Red";
    Color[Color["Green"] = Symbol("Green")] = "Green";
    Color[Color["Blue"] = Symbol("Blue")] = "Blue";
})(Color||(Color={}));
```

For this source,

```ts
class Type {
    constructor(public value: String);
}

enum Color {
  Red = new Type("Red"),
  Green,
  Blue,
}
enum Color: Type { Red, Green, Blue }
```

the following should be emitted for either version of the `Color` enum:

```js
var Color;
(function (Color) {
    Color["Red"] = new Type("Red");
    Color["Green"] = new Type("Green");
    Color["Blue"] = new Type("Blue");
})(Color||(Color={}));
```

For this source,

```ts
class Type {
    constructor();
}

enum Color: Type { Red, Green, Blue }
```

the following should be emitted for the `Color` enum:

```js
var Color;
(function (Color) {
    Color["Red"] = new Type();
    Color["Green"] = new Type();
    Color["Blue"] = new Type();
})(Color||(Color={}));
```

For this source,

```ts
class Type {
    constructor(public value: number);
}

enum Color: Type { Red, Green, Blue }
```

the following should be emitted for the `Color` enum:

```js
var Color;
(function (Color) {
    Color["Red"] = new Type(0);
    Color["Green"] = new Type(1);
    Color["Blue"] = new Type(2);
})(Color||(Color={}));
```

For this source,

```ts
class Type {
    constructor(public value: number);
}

enum Color: Type { Red = new Type(1), Green, Blue }
```

the following should be emitted for the `Color` enum:

```js
var Color;
(function (Color) {
    Color["Red"] = new Type(1);
    Color["Green"] = new Type(2);
    Color["Blue"] = new Type(3);
})(Color||(Color={}));
```

For this source,

```ts
class Type {
    constructor(public value: string, public index: number);
}

enum Color: Type {
    Red = new Type("Red", 1),
    Green = new Type("Green", 2),
    Blue = new Type("Blue", 3),
}
```

the following should be emitted for the `Color` enum:

```js
var Color;
(function (Color) {
    Color["Red"] = new Type("Red", 1);
    Color["Green"] = new Type("Green", 2);
    Color["Blue"] = new Type("Blue", 3);
})(Color||(Color={}));
```

For this source,

```ts
class Type {
    constructor(public value: string, public index: number);
}

enum Color: string {
    Red = "Red",
    Green = "Blue",
    Blue = "Purple",
}
```

the following should be emitted for the `Color` enum:

```js
var Color;
(function (Color) {
    Color["Red"] = "Red";
    Color["Green"] = "Blue";
    Color["Blue"] = "Purple";
})(Color||(Color={}));
```

*End of proposal.*

---

Examples from Microsoft/TypeScript#1206

```ts
interface EmscriptenEnumEntry {
  value: number; /* some more properties ... */
}
declare enum Month: EmscriptenEnumEntry {Jan, Feb, Mar}
// Month.Jan.value == 0, Month.Feb.value == 1, ...

interface IFoo {
    id: string;
    code: number;
}
enum Foo: IFoo {
    BAR = { id: "BAR", code: 123 },
    BAZ = { id: "", code: 0 }
}

enum Audience: String {
    Public,
    Friends,
    Private,
}

enum Test: string {
    Foo = "Bar",
    Bar = "Baz",
    Baz = "Foo"
}

// const string enums now exist.
const enum A: string {x, y, z}

enum Foo2: boolean {
    BAR = true, // assigning required
    BAZ = false
}

enum TokenTypes: TokenType {
    OpenCurved   = type('(', 1),
    CloseCurved  = type(')', 1),
    OpenBracket  = type('[', 1),
    CloseBracket = type(']', 1),
    OpenCurly    = type('{', 1),
    CloseCurly   = type('}', 1),
    Identifier   = type('Identifier', 0),
    String       = type('String', 0),
    EOF          = type('EOF', 0),
}

enum tuples: [number, number] {
  a = [0, 1],
  b = [0, 2],
  // tuples...
}

/*
 * The Planets Java example
 */
class PlanetType {
    constructor(
        private label: string,
        private size: string,
        private orbit: number);
    
    canSwallow(planet: Planet): bool {
       return planet.size < this.size;
    }

    isCloserToSun(planet: Planet): bool {
       return planet.orbit < this.orbit;
    }

    static fromOr(nameindexOrType: string, defVal: Planet = null): Planet {
       var e = from(nameindexOrType);
       return e == null ? defVal : e;
    }

    static from(nameindexOrType: string): Planet {
        if (nameindexOrType == null) {
            return null;
        } else if (typeof nameindexOrType === 'Number') {
            switch(nameindexOrType){
            case 0: return Planet.Mercury;
            case 1: return Planet.Venus;
            case 2: return Planet.Earth;
            // ...
            }
        } else if (typeof nameindexOrType === 'String') {
            nameindexOrType = nameindexOrType.ToUpperCase();
            switch(nameindexOrType){
            case 'MERCURY': return Planet.Mercury;
            case 'VENUS': return Planet.Venus;
            case 'EARTH': return Planet.Earth;
            // ...
            }
        } else {
            return null;
        }
    }
}

enum Planet: PlanetType {
    Mercury = new PlanetType("Mercury", 1, 1),
    Venus = new PlanetType("Venus", 2.8, 2),
    Earth = new PlanetType("Home", 3, 3),
    // ...
}

/*
 * The DeploymentStatus Java example
 */
class DeploymentStatusType implements InternationalizedEnumType {
    constructor(
        private value: number,
        private code: string,
        private messageCode: string);

    getCode() { return code; }

    getValue() { return value; }

    getMessageCode() { return messageCode; }

    static parse(id: number) {
        if (id != null) {
            switch (id) {
            case 100: return DeploymentStatus.PENDING;
            case 110: return DeploymentStatus.QUEUED_FOR_RELEASE;
            case 120: return DeploymentStatus.READY_FOR_RELEASE;
            case 130: return DeploymentStatus.RELEASING;
            case 140: return DeploymentStatus.RELEASED;
            case 200: return DeploymentStatus.QUEUED;
            case 300: return DeploymentStatus.READY;
            case 400: return DeploymentStatus.STARTED;
            case 500: return DeploymentStatus.SUCCEEDED;
            case 600: return DeploymentStatus.FAILED;
            case 700: return DeploymentStatus.UNKNOWN;
            }
        }
        return DeploymentStatus.UNDEFINED;
    }

    isFinalStatus() {
        return this === DeploymentStatus.SUCCEEDED ||
            this === DeploymentStatus.FAILED ||
            this === DeploymentStatus.UNKNOWN;
    }
}

export default enum DeploymentStatus {
    PENDING = new DeploymentStatusType(100, "STATUS_PENDING", "status.pending"),
    QUEUED_FOR_RELEASE = new DeploymentStatusType(110, "STATUS_QUEUED_FOR_RELEASE", "status.queuedrelease"),
    READY_FOR_RELEASE = new DeploymentStatusType(120, "STATUS_QUEUED_RELEASE","status.readyrelease"),
    RELEASING = new DeploymentStatusType(130, "STATUS_RELEASING", "status.startedrelease"),
    RELEASED = new DeploymentStatusType(140, "STATUS_RELEASED", "status.suceededrelease"),
    QUEUED = new DeploymentStatusType(200, "STATUS_QUEUED", "status.queued"),
    READY = new DeploymentStatusType(300, "STATUS_READY", "status.ready"),
    STARTED = new DeploymentStatusType(400, "STATUS_STARTED", "status.started"),
    SUCCEEDED = new DeploymentStatusType(500, "STATUS_SUCCEEDED", "status.succeeded"),
    FAILED = new DeploymentStatusType(600, "STATUS_FAILED", "status.failed"),
    UNKNOWN = new DeploymentStatusType(700, "STATUS_UNKNOWN", "status.unknown"),
    UNDEFINED = new DeploymentStatusType(0, "", "status.undefined");
}
```