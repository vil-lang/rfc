# 🍃 AVT
This document specifies the core _AVTree_ node types that support one of two _VIL_ grammars.

## Node objects
_AVT_ nodes are represented as `Node` objects with an additional `type` field. They implement the following interface:

```js
interface Node {
	type: string;
	loc: SourceLocation | null;
}
```

The `type` field is a string representing the _AVT_ variant type. Each subtype of `Node` is documented below with the specific string of its `type` field. You can use this field to determine which interface a node implements.

The `loc` field represents the source location information of the node. If the node contains no information about the source location, the field is `null`; otherwise it is an object that references the source node, its parent and children:

```js
interface SourceLocation {
  node: Object | null;
	parent: Object | null;
	children: [ Object ] | null;
}
```

Basically it maps an _AVT_ node to a node belonging to the source tree.

## Identifier
An `Identifier` defines a unique name for the entire _AVT_.

```js
interface Identifier <: Node {
	type: 'Identifier';
	name: string;
}
```

## Layer objects
A `Layer` is a rectangular area that is part of a nested structure made of other layers. `opacity` is a `number` between `0` and `1`.

```js
interface Layer <: Node {
	id: Identifier;
	name: string | null;
	parent: Layer;
	children: [ Layer ];
	position: PointAttribute;
	size: PointAttribute;
	rotation: number;
 	opacity: number;
	visible: boolean;
}
```

## Documents
A `Document` is the root of an _AVT_.

```js
interface Document <: Layer {
  version: '1.0.0',
  type: 'document';
  source: SourceType;
	parent: null;
  children: [ Artboard ];
}
```

The `version` field tells which type of  _AVT_ is used in order to adapt to potential future additions to _VIL_.

The `source` field tells what type of document this _AVT_ was built from:

```js
enum SourceType {
	'sketch' | 'psd' | 'none' | 'unknown'
}
```

## Artboards
An `Artboard` is the root a subtree of layers that represents a meaningful unit of design.

```js
interface Artboard <: Layer {
	type: 'artboard';
	parent: Document;
	children: [ Layer ];
}
```

## Elements
An `Element` is a specialised layer that contains `DataLayer`  as children. Each type of `Element` can only have one type of `DataLayer` associated.

```js
interface Element <: Layer {
	fills: [ FillAttribute ] | null;
	borders: [ BorderAttribute ] | null;
	shadows: [ ShadowAttribute ] | null;
	children: [ DataLayer ];
}
```

### ImageElement

An `Image` contains one or more `BitmapData` layers.

```js
interface ImageElement <: Element {
	type: 'ImageElement';
  children: [ BitmapData ];
}
```

### ShapeElement

A `Shape` contains one or more `PathData` layers.

```js
interface ShapeElement <: Element {
	type: 'ShapeElement';
	children: [ PathData ];
}
```

### TextElement

A `TextElement` contains one or more `TextData` layers.

```js
interface TextElement <: Element {
	type: 'TextElement';
	font: FontAttribute;
	children: [ TextData ];
}
```

### SymbolElement

A `SymbolElement` references a `Symbol`. Note that a `SymbolElement` has no children.

```js
interface SymbolElement <: Element {
	type: 'SymbolElement';
	ref: Identifier;
	children: null;
}
```

## Attributes
```js
interface Attribute <: Node { }
```

### PointAttribute

A `PointAttribute` references a `PointValue`.

```js
interface PointAttribute <: Attribute {
	type: 'PointAttribute';
	point: PointValue;
}
```

### FillAttribute

A `FillAttribute` references either a `ColorValue` or `GradientValue`.

```js
interface FillAttribute <: Attribute {
	type: 'FillAttribute';
	fill: ColorValue | GradientValue;
}
```

### BorderAttribute

A `BorderAttribute` references a `ColorValue` and defines the `thickness` of the border; `thickness` can only be a positive `number`.

```js
interface BorderAttribute <: Attribute {
	type: 'BorderAttribute';
	color: ColorValue;
	thickness: number;
}
```

### ShadowAttribute

A `ShadowAttribute` references a `ColorValue`, defines its orientation with `x` and `y`, and its shape with `blur` and `spread`. All must be a positive `number`.

```js
interface ShadowAttribute <: Attribute {
	type: 'ShadowAttribute';
	color: ColorValue;
	x: number;
	y: number;
	blur: number;
	spread: number;
}
```

### FontAttribute

A `FontAttribute` references a `ColorValue`, #todo split this

```js
interface FontAttribute <: Attribute {
	type: 'FontAttribute';
	typeface: string;
	weight: number;
	style: "normal" | "italic";
	size: number;
	color: ColorValue;
	decoration: "none" | "underline" | "overline" | "line-through";
	transform: "none" | "uppercase" | "lowercase";
	alignement: "left" | "center" | "right" | "justify";
	spacing: FontSpacing | null;
}
```

#### FontSpacing

```js
interface FontSpacing {
	letter: number;
	line: number;
	paragraph: number;
}
```

## Data Layers
```js
interface DataLayer <: Layer { }
```

### BitmapData

A `BitmapData` references a `PixelsValue`.

```js
interface BitmapData <: DataLayer {
	type: 'BitmapData';
	pixels: PixelsValue;
}
```

### PathData

A `PathData` references one or more `PointValue`.

```js
interface PathData <: DataLayer {
	type: 'PathData';
	points: [ PointValue ];
}
```

### TextData

A `TextData` defines text content.

```js
interface TextData <: DataLayer {
	type: 'TextData';
	text: string;
}
```

## Value
```js
interface Value <: Node { }
```

### ColorValue

A `ColorValue` defines `r`, `g`, `b` and `a` channels of a color; channels must be integers between `0` and `255`.

```js
interface ColorValue <: Value {
	type: 'ColorValue';
	r: number;
	g: number;
	b: number;
	a: number;
}
```

### GradientValue

A `GradientValue` references one or more `ColorStop`.

```js
interface GradientValue <: Value {
	type: 'GradientValue';
	stops: [ ColorStop ];
}
```

#### ColorStop

```js
interface ColorStop {
	type: 'ColorStop';
	position: PointValue;
	color: ColorValue;
}
```

### PixelsValue

A `PixelsValue` defines an array of `32-bits RGBA` encoded pixels.

```js
interface PixelsValue <: Node {
	type: 'PixelsValue';
	pixels: Uint32Array
}
```

### PointValue

A `PointValue` defines  `x` and `y` coordinates.

```js
interface PointValue <: Node {
	type: 'PointValue';
	x: number;
	y: number;
}
```

### SizeValue

A `SizeValue` defines `width` and `height`.

```js
interface SizeValue <: Node {
	type: 'SizeValue';
	width: number;
	height: number;
}
```

## Symbols
A `Symbol` is an invisible layer that defines a reusable subtree of layers.

```js
interface Symbol <: Layer {
	type: 'Symbol';
	children: [ Layer ];
}
```
