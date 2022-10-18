---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
---

# TypeScript 4.9 Beta

---

# 本日のひとこと

## ワールドトリガー面白い！

---

# Agenda

- `satisfies` 演算子
- `in` 演算子での絞り込みの拡張
- `NaN` との等値比較
- （おまけ） TypeScript 4.8

---

# `satisfies` 演算子

今までの問題点

「ある式」が「ある型」にマッチすることを保証したい

推論のためにその式の最も具体的な型も残して欲しい

↑ が対応されていなかった

---

# `satisfies` 演算子

```ts
const palette = {
    red: [255, 0, 0],
    green: '#00ff00',
    bleu: [0, 0, 255] // typo: blue
}

// 'red' に配列メソッドを使う
const redComponent = palette.red.map((ele) => ele)

// 'green' に文字列メソッドを使う
const greenNormarized = palette.green.toUpperCase()
```

---

# `satisfies` 演算子

typo は、 `palette` の型アノテーションを使って検出できそう🤔

```ts
type Colors = 'red' | 'green' | 'blue'

type RGB = [red: number, green: number, blue: number]

const palette: Record<Colors, string | RGB> = {
    red: [255, 0, 0],
    green: '#00ff00',
    bleu: [0, 0, 255] // typo が検出される
}

// Property 'map' does not exist on type 'string | RGB'.
const redComponent = palette.red.map((ele) => ele)
```

---

# `satisfies` 演算子

`palette.red` は文字列かもしれないので、配列メソッドが使えない🤔

```ts {11,12}
type Colors = 'red' | 'green' | 'blue'

type RGB = [red: number, green: number, blue: number]

const palette: Record<Colors, string | RGB> = {
    red: [255, 0, 0],
    green: '#00ff00',
    bleu: [0, 0, 255] // typo が検出される
}

// Property 'map' does not exist on type 'string | RGB'.
const redComponent = palette.red.map((ele) => ele)
```

---

# `satisfies` 演算子

理想

`palette` のキーは `Colors` 型にマッチしていて、使えるメソッドについては、 `Type` に指定した型から正確な型を推論して欲しい

```ts
type Colors = 'red' | 'green' | 'blue'

type RGB = [red: number, green: number, blue: number]

const palette: Record<Colors, string | RGB> = {
    red: [255, 0, 0],
    green: '#00ff00',
    bleu: [0, 0, 255] // typo が検出される
}

// Property 'map' does not exist on type 'string | RGB'.
const redComponent = palette.red.map((ele) => ele)
```

---

# `satisfies` 演算子

この問題を解決できるのが `satisfies` 演算子 🚀

```ts
type Colors = 'red' | 'green' | 'blue'

type RGB = [red: number, green: number, blue: number]

const palette = {
    red: [255, 0, 0],
    green: '#00ff00',
    bleu: [0, 0, 255] // typo が検出される
} satisfies Record<Colors, string | RGB>

const redComponent = palette.red.map((ele) => ele)

const greenNormarized = palette.green.toUpperCase()
```

---

# `satisfies` 演算子

他の使用例

あるオブジェクトが、ある型のキーを全て持っていて、それ以上持っていないことを検証できる

```ts
type Colors = 'red' | 'green' | 'blue'

const favoriteColors = {
    'red': 'yes',
    'green': false,
    'blue': 'kinda'
} satisfies Record<Colors, unknown>

const g: boolean = favoriteColors.green
```

---

# `satisfies` 演算子

これを `Record` ユーティリティのみでやろうと思った場合

```ts
type Colors = 'red' | 'green' | 'blue'

const favoriteColors: Record<Colors, unknown> = {
    'red': 'yes',
    'green': false,
    'blue': 'kinda',
}

// Type 'unknown' is not assignable to type 'boolean'.
const g: boolean = favoriteColors.green
```

---

# `satisfies` 演算子

```ts
type Colors = 'red' | 'green' | 'blue'

const favoriteColors = {
    'red': 'yes',
    'green': false,
    // Type '{ red: string; green: boolean; }' does not satisfy the expected type 'Record<Colors, unknown>'
} satisfies Record<Colors, unknown>

const g: boolean = favoriteColors.green
```

```ts
type Colors = 'red' | 'green' | 'blue'

const favoriteColors = {
    'red': 'yes',
    'green': false,
    'blue': 'kinda',
    'orange': true // Type '{ red: string; green: boolean; blue: string; orange: boolean; }' does not satisfy the expected type 'Record<Colors, unknown>'.
} satisfies Record<Colors, unknown>

const g: boolean = favoriteColors.green
```

---

# `satisfies` 演算子

オブジェクトのキー部分はどうでも良いが、値の型は検証したいという場合にも使える

```ts
type RGB = [red: number, green: number, blue: number]

const palette = {
    red: [255, 0, 0],
    green: '#00ff00',
    blue: [0, 0] // Type '[number, number]' is not assignable to type 'string | RGB'.
} satisfies Record<string, string | RGB>
```

---

# `in` 演算子での絞り込みの拡張

おさらい: JavaScript の `in` 演算子

```ts
const car = {
    make: 'Honda',
    model: 'Acoord',
    year: 1998
}

console.log('make' in car) // => true
console.log('foo' in car) // => false
```

---

# `in` 演算子での絞り込みの拡張

これまで TypeScript では、プロパティを明示的に列挙していない型の絞り込みができた

```ts
type RGB = {
    red: number
    green: number
    blue: number
}

type HSV = {
    hue: number
    saturation: number
    value: number
}
```

---

# `in` 演算子での絞り込みの拡張

これまで TypeScript では、プロパティを明示的に列挙していない型の絞り込みができた

```ts
const setColor = (color: RGB | HSV) => {
  if ('hue' in color) {
    console.log(true)
  } else {
    console.log(false)
  }
}

const rgbColor: RGB = {
  red: 0,
  green: 0,
  blue: 0
}

const hsvColor: HSV = {
  hue: 0,
  saturation: 0,
  value: 0
}

setColor(rgbColor) // => false
setColor(hsvColor) // => true
```

---

# `in` 演算子での絞り込みの拡張

例えば JavaScript で、引数の型が明記されていない場合、コードが意味不明になってくる

```ts
function tryGetPackageName(context) {
  const packageJSON = context.packageJSON
  
  if (packageJSON && typeof packageJSON === 'object') {
    if ('name' in packageJSON && typeof packageJSON.name === 'string') {
      return packageJSON.name
    }
  }
  return undefined
}
```

---

# `in` 演算子での絞り込みの拡張

例えば JavaScript で、引数の型が明記されていない場合、コードが意味不明になってくる

```ts
const foo = {
  packageJSON: {
    name: 'name'
  }
}

const bar = {
  packageJSON: {
    bar: 'string'
  }
}

console.log(tryGetPackageName(foo)) // => 'name'
console.log(tryGetPackageName(bar)) // => undefined
```

---

# `in` 演算子での絞り込みの拡張

TypeScript(v4.8.4) で型を明示するとわかりやすくなるけど...

```ts
type Context = {
    // unknown などの型を安易に使用すると問題が発生する
    packageJSON: unknown
}

function tryGetPackageName(context: Context) {
  const packageJSON = context.packageJSON
  
  if (packageJSON && typeof packageJSON === 'object') {
    // Property 'name' does not exist on type 'object'.
    if ('name' in packageJSON && typeof packageJSON.name === 'string') {
      // Property 'name' does not exist on type 'object'.
      return packageJSON.name
    }
  }
  return undefined
}
```

---

# `in` 演算子での絞り込みの拡張

`unknown` から `object` に絞られる一方、 `in` 演算子はチェック対象のプロパティを実際に定義している方に厳密に絞るため、 `packageJSON` の型は、 `object` として扱われる

```ts{9,11}
type Context = {
    // unknown などの型を安易に使用すると問題が発生する
    packageJSON: unknown
}

function tryGetPackageName(context: Context) {
  const packageJSON = context.packageJSON
  
  if (packageJSON && typeof packageJSON === 'object') {
    // Property 'name' does not exist on type 'object'.
    if ('name' in packageJSON && typeof packageJSON.name === 'string') {
      // Property 'name' does not exist on type 'object'.
      return packageJSON.name
    }
  }
  return undefined
}
```

---

# `in` 演算子での絞り込みの拡張

TypeScript(v4.9.4) では、全く列挙していない型を絞り込む際に、 `in` 演算子が強力になった

`Record<'property-key-being-checked', unknown>` で型を交差させるようになる

```ts{9,10,11,12,13}
type Context = {
    // unknown などの型を安易に使用すると問題が発生する
    packageJSON: unknown
}

function tryGetPackageName(context: Context) {
  const packageJSON = context.packageJSON
  
  if (packageJSON && typeof packageJSON === 'object') {
    // `unknown` 型から `object` に絞り込む
    if ('name' in packageJSON && typeof packageJSON.name === 'string') {
      // `object & Record<'name', unknown>` となるのでアクセス可能に！
      return packageJSON.name
    }
  }
  return undefined
}
```

---

# `NaN` との等値比較

JavaScript で度々問題になる `NaN` のチェック

`NaN` は、 `not a number` を意味する特殊な数値なので、 `NaN` と等しいものは存在しない


```ts
console.log(NaN == 0) // false
console.log(NaN === 0) // false

console.log(NaN == NaN) // false
console.log(NaN === NaN) // false

console.log(NaN != 0) // true
console.log(NaN !== 0) // true

console.log(NaN != NaN) // true
console.log(NaN !== NaN) // true
```

---

# `NaN` との等値比較

JavaScript で度々問題になる `NaN` のチェック

- この挙動は JavaScript 固有の問題ではなく、 IEEE-754 の浮動小数点数の仕様を含む全ての言語で同じ挙動をする
  - Java や C# など仕様に明記している例もあるが、ほとんどの言語は暗黙的にこの仕様に則っているらしい

---

# `NaN` との等値比較

しかし、 `NaN` のチェックはしたい！

そんな時は `Number.isNaN`

```ts
console.log(Number.isNaN(0)) // false
console.log(Number.isNaN(NaN)) // true
console.log(Number.isNaN('foo')) // false

// ちなみに Number コンストラクターを付けないと
console.log(isNaN(0)) // false
console.log(isNaN(NaN)) // true
console.log(isNaN('foo')) // true
```

---

# `NaN` との等値比較

TypeScript(v4.9.4) では、エラー出しと、 `Number.isNaN` への切り替え提案をするように！

```ts
function validate(someValue: number) {
  // This condition will always return 'true'.
  //　　Did you mean '!Number.isNaN(someValue)'?
  return someValue !== NaN
}
```

---

# `NaN` との等値比較

他にエラー出してくれるのって、こういう例？

```ts
// This condition will always return 'false' since JavaScript compares objects by reference, not value.
const foo = [0, 0, 0] === [0, 0, 0] // このこと？
const bar = { hoge: 'hoge' } === { hoge: 'hoge' }
```

---

# （おまけ） TypeScript 4.8

4.8 で入った型の絞り込みの改善について

`{}` 型はプロパティを持たないオブジェクトを表す型

プロパティを持ちうる値なら何でも代入できるが、 `null` と `undefined` 以外は代入できる

```ts
// Type 'null' is not assignable to type '{}'.
let foo: {}
foo = null
```

---

# （おまけ） TypeScript 4.8

4.7までは任意の型引数が `{}` 型に代入可能であった

```ts
function someFunc<T>(x: T) {
  // エラーにならない
  const some: {} = x
}

// null が代入できてしまう
someFunc(null)
```

---

# （おまけ） TypeScript 4.8

4.8では

```ts
function someFunc<T>(x: T) {
  // Type 'T' is not assignable to type '{}'.
  // This type parameter might need an `extends {}` constraint.
  const some: {} = x
}

// ここはエラーにならない
someFunc(null)
```

---

# （おまけ） TypeScript 4.8

エラーを修正するには `T` に制約をつける

```ts
// T は {} の型であると制約をつける
function someFunc<T extends {}>(x: T) {
  // ここはエラーにならない
  const some: {} = x
}

// Argument of type 'null' is not assignable to parameter of type '{}'.
someFunc(null)
```

---

# （おまけ） TypeScript 4.8

一方で、 `unknown` 型は、 `null` や `undefined` も取りうる型

```ts
let foo: unknown

foo = undefined
```

---

# （おまけ） TypeScript 4.8

`unknown` 型の値を使いたい場合は、 `null` や `undefined` は絞り込んで除外したいが、今までは `unknown` 型から `null` と `undefined` だけを除外することができなかった

```ts
function someFunc(x: unknown) {
    if (x !== null && x !== undefined) {
        // Type 'unknown' is not assignable to type '{}'.
        const y: {} = x
    }
}
```

---

# （おまけ） TypeScript 4.8

4.8 からは下記で絞り込みができる

```ts
function someFunc(x: unknown) {
    if (x !== null && x !== undefined) {
        const y: {} = x
        console.log(y)
    } else {
        console.log('else')
    }
}

someFunc(1) // => 1
someFunc(undefined) // => 'else'
```

---

# （おまけ） TypeScript 4.8

なので、4.8からは `unknown` 型は、 `{} | null | undefined` のように扱われるようになったと言える

下記のように引数の真偽値をチェックするだけで、絞り込みが可能になった

```ts
function someFunc(x: unknown) {
    if (x) {
        const y: {} = x
        console.log(y)
    } else {
        console.log('else')
    }
}

someFunc(1) // => 1
someFunc(undefined) // => 'else'
```

---

# （おまけ） TypeScript 4.8

この挙動のおかげで、 `NonNullable<T>` が便利になった

`NonNullable<T>` 型は、 `T` が `null | undefined` の可能性を排除した型のこと

```ts
// 4.7までの型定義
type NonNullable<T> = T extends null | undefined ? never : T;
```

---

# （おまけ） TypeScript 4.8

`T` に `unknown` 型が指定された場合、さっきの通り `{}` との挙動で問題が生じる

```ts
type Foo = NonNullable<unknown>

let x: Foo
let y: {}

// Type 'unknown' is not assignable to type '{}'.
y = x
```

---

# （おまけ） TypeScript 4.8

型引数に対しても、型引数の中身がわからないため未解決になる

```ts
function someFunc<T>(x:T) {
    // type Foo = T extends null | undefined ? never : T
    type Foo = NonNullable<T>

    if (x !== null && x !== undefined) {
        // Type 'T' is not assignable to type 'NonNullable<T>'.
        const y: Foo = x
    }
}
```

---

# （おまけ） TypeScript 4.8

しかし、4.8からは `NonNullable<T>` 型の定義が変更された

```ts
// 4.7までの型定義
type NonNullable<T> = T extends null | undefined ? never : T;

// 4.8
type NonNullable<T> = T & {};
```

---

# （おまけ） TypeScript 4.8

つまり、 `{}` に関する絞り込みが改善されたため、 `NonNullable<T>` 型も改善されたことになる


```ts
function someFunc<T>(x:T) {
    // type Foo = T & {}
    type Foo = NonNullable<T>

    if (x !== null && x !== undefined) {
        // 型エラーにならない
        const y: Foo = x
    }
}
```