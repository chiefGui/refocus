![Virtualform hero](https://i.imgur.com/OAgybA4.png)

<div align="center">

### The documentation for `@virtualform/grid`

[Home](https://github.com/chiefGui/virtualform) · [Quick Start](#-quick-start) · [API](#gear-api) · [Recipes](#-recipes) · [Demo](https://virtualform.vercel.app) · [Playground](https://stackblitz.com/edit/virtualform-grid?file=App.tsx)

</div>

## 🏁 Quick Start

As usual, you have to install it first.

```shell
yarn add @virtualform/grid
```

Then have fun!

```tsx
import { useGrid } from '@virtualform/grid'

const App = () => {
  const { getRootProps, getWrapperProps, cells } = useGrid({
    cells: {
      amount: 1000,
      width: 100,
      height: 100,
    },
  })

  const { style, ...rootProps } = getRootProps()

  return (
    <div style={{ ...style, width: '100vw', height: 600 }} {...rootProps}>
      <div {...getWrapperProps()}>
        {cells.map((cell) => {
          return (
            <div {...cell.getProps()}>
              <div>I am cell {cell.index}</div>
            </div>
          )
        })}
      </div>
    </div>
  )
}
```

A few quick good-to-knows:

1. The code above will give you a grid of `1000` cells,
2. Each cell will have the min width of `100px` and the max width of `100px`,
3. Each cell will have the fixed height of `100px`,
4. You must be explicit in the `height` of the root div (the one spreading `...rootProps`).

## :gear: API

### Inputs

Below we are going to see what `useGrid()` must and can receive.

#### `cells` · _Required_

```tsx
useGrid({
  cells: {
    amount: 1000,
    width: 100,
    height: 100,
  },
})
```

- `amount`: `number` · _Required_
  The absolute amount of cells to render in the grid. A cell is a single element. A gallery displaying 5000 pictures has a `cells.amount` of `5000`.

- `width`: `number` · _Required_
  The `width` of the cells. To be completely honest with you, this is an _estimation_ rather than an _exact size_. The reason for an estimation is because of the responsive nature of Virtualform: depending on the screen size, respecting the width passed through this variable, mathematically speaking, is not possible. So in order to achieve pixel perfection, **Virtualform** will take the `width` only as a reference of what you want your cells to look like, and will do its best to be as close to it as possible.

- `height`: `number` · _Required_
  The `height` of the cells. This is an exact size and won't ever change unless you manually do so. Just in case: the height of the cells are the height of the rows as well. Meaning, whatever the value you set as `height`, expect that to be the height of your rows.

### `gap`: `number`, Optional

Represents the distance, in pixels, between each cell, vertically and horizontally.

```tsx
useGrid({
  gap: 10,
})
```

> ⚠️ Currently you cannot specify gaps at specific diretions. I.e. gap only at the top, or only at the bottom, etc.

> ⚠️ Gaps won't space the surroundings of your grid. Use [`gutter`](#gutter) for that.

### `gutter`: `number`, Optional

Represents the space, in pixels, around your virtualized grid.

```tsx
useGrid({
  gutter: 50,
})
```

> ⚠️ Currently you cannot specify gutter at specific diretions. I.e. gutter only at the top, or only at the bottom, etc.

### `overscan`: `number`, Optional

Represents how many _rows_ not visible at the viewport will be mounted.

```tsx
useGrid({
  overscan: 2,
})
```

This property is useful when you want a smoother experience when the user is scrolling around. In other words, **Virtualform** will mount not only the cells the user is seeing, but also the cells at the _rows_ specified at `overscan` above and below the topmost and the bottommost visible rows.

See the examples below:

![Instructions for overscan at 0](https://i.imgur.com/7OqqCka.png)

The image above demonstrates what happens when your `overscan` is at `0` (the default value). As you can see, only the visible cells are mounted on the DOM and this is the fastest way to use **Virtualform**.

Now, let's look at what happens when you have an `overscan` of `2`:

![Instructions for overscan](https://i.imgur.com/ZXBwYR2.png)

As you can see, the two rows above and below the visible row will be mounted as well, meaning that when the user scrolls through these overscanned rows, they won't see that blink of components mounting.

To recap:

- A "visible" cell is the concept of a mounted cell, except it's present at the visible portion of your grid, so it's expected the user is seeing it.
- An "overscanned" cell is a mounted cell that's not visible at the viewport.
- A mounted cell is present at the DOM and will cost to the CPU to compute it, even though it's not necessarily explictly visible to the human eye.
- Not mounted cells don't cost CPU computation and only exist in memory. These are the virtualized cells.

#### A note of caution

Although overscan is a good technique to make the user experience even better, it's important you don't overabuse it (pun intended) because it may cause the contrary effect: making the experience worse.

1. Overscan is impercetible when scrolling too fast. In this case, because of how fast the scroll occurs, it doesn't give enough time for **Virtualform** to compute the overscanned rows. If you expect your users to scroll very fast, you can completely skip the `overscan` property.
2. Overscan occurs at row-level, not cell-level. Meaning that it'll mount all cells in the `x` rows above and below the topmost and bottommost visible rows, where `x` is the number you passed to `overscan`.
3. **Too big numbers at `overscan` will make your grid slower.** For example, if you pass to `overscan` the same amount of `cells.amount`, is as if nothing is virtualized, thus **Virtualform** is completely needless.

My recommendation for `overscan` is: while developing, start with `0` and play with your grid. If it doesn't feel completely good yet, increase it by `1`. Play with your grid again. Still think it can be smoother? Then repeat the process until you find the sweet spot. Also, a quick tip: the less columns (yes, columns, not cells), the greater the number for `overscan` can safely be.

---

### Outputs

Below we are going to see what `useGrid()` returns.

#### `getRootProps`

A function that returns the necessary props the root `<div />` must have in order to proper virtualize things. It's nothing scary, really, just some styles and a `ref`.

```tsx
const { getRootProps, ...etc } = useGrid({
  // ...
})

const { style, ...rootProps } = getRootProps()

return <div style={{ ...style, height: '100vh' }} {...rootProps} />
```

> ⚠️ The root `<div />` necessarily needs an explicit `height`.

#### `getWrapperProps`

A function that returns the necessary props a `<div />` inside the root div needs to properly virtualize things. `styles` only, and we recommend that you don't tweak position-related styles too much otherwise your virtualization may break.

```tsx
const { getRootProps, getWrapperProps } = useGrid({
  // ...
})

const { style, ...rootProps } = getRootProps()

return (
  <div style={{ ...style, height: '100vh' }} {...rootProps}>
    <div {...getWrapperProps()} />
  </div>
)
```

#### `cells`

An array containing the mounted cells (the visible ones and the ones used by [overscan](#overscan-number-optional)). You must render these cells inside the wrapper div.

```tsx
const { getRootProps, getWrapperProps, cells } = useGrid({
  // ...
})

const { style, ...rootProps } = getRootProps()

return (
  <div style={{ ...style, height: '100vh' }} {...rootProps}>
    <div {...getWrapperProps()}>
      {cells.map((cell) => (
        <div {...cell.getProps()}>Item {cell.index}</div>
      ))}
    </div>
  </div>
)
```

Let's take a look at the properties of a single cell, shall we?

- `index`: `number`
  Use this index to get data from your actual array of data you want to display. For example, if you have an array `pictures` and you want to display your pictures in order, you have to do something like `pictures[cell.index].url`.

- `getProps`: `function`
  A function that returns only the necessary props to use in the firstmost div of a cell. Returns a `key` so you don't have to worry about it, as well as a `style`.

#### `recompute`

The function that does the virtualization magic. The same one used internally by **Virtualform**, exposed to you.

```tsx
const { getRootProps, getWrapperProps, cells, recompute } = useGrid({
  // ...
})

const { style, ...rootProps } = getRootProps()

useEffect(() => {
  recompute()
}, [somethingThatMayAffectTheGrid])

return (
  <div style={{ ...style, height: '100vh' }} {...rootProps}>
    <div {...getWrapperProps()}>
      {cells.map((cell) => (
        <div {...cell.getProps()}>Item {cell.index}</div>
      ))}
    </div>
  </div>
)
```

I won't call it unlikely, but maybe you don't need to call `recompute`, ever. Whenever an [input](#inputs) property change, **Virtualform** will recompute your grid in a very optimized way. This is only exposed because under certain circumstances, you may benefit from it. That said, use wisely.

> ⚠️ This is an expensive task. Using it with caution it's okay and won't damage the experience, but overabusing it can freeze your app.

#### `mountedRowsIndices`

Returns an array containing the indices of all the mounted rows (`number[]`).

```tsx
const { getRootProps, getWrapperProps, cells, mountedRowsIndices } = useGrid({
  // ...
})

const { style, ...rootProps } = getRootProps()

useEffect(() => {
  alert(`There are ${mountedRowsIndices.length} in the DOM!`)
}, [mountedRows.length])

return (
  <div style={{ ...style, height: '100vh' }} {...rootProps}>
    <div {...getWrapperProps()}>
      {cells.map((cell) => (
        <div {...cell.getProps()}>Item {cell.index}</div>
      ))}
    </div>
  </div>
)
```

It's very important that _you don't trust the order_ of the indices. So:

❌ **Don't:**

```ts
if (mountedRowsIndices[10]) {
  // do something when row 10 is mounted
}
```

✅ **Do:**

```ts
if (mountedRowsIndices.includes(10)) {
  // do something when row 10 is mounted
}
```

**PRO TIP:** It's very likely that you don't need this property at all. However, it's useful if you are into [infinite loading](/packages/grid/recipes/Infinite-Loading.md).

#### `rowsAmount`

Returns the total amount (`number`) of rows computed by the grid, regardless whether they are mounted or not.

#### `colsPerRow`

Returns the computed amount (`number`) of columns/cells per row.

#### `colWidth`

Returns the computed width (`number`), in pixels, of the columns/cells. All columns/cells have the same width.

## 👩‍🍳 Recipes

- [Infinite Loading](/packages/grid/recipes/Infinite-Loading.md)

---

#### Brought to you by 🇧🇷 [Guilherme "chiefGui" Oderdenge](https://github.com/chiefGui) and [Starchive](https://starchive.io/).

---

The MIT License (MIT)

Copyright (c) 2022 Guilherme Oderdenge

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
