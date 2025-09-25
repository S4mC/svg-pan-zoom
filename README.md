# svg-pan-zoom

A JavaScript library based on https://github.com/bumbu/svg-pan-zoom that enables pan and zoom functionality for SVG elements in web browsers through mouse, touch, and keyboard interactions.

## Features

- **Interactive Controls**: Mouse drag to pan, mouse wheel to zoom, double-click to zoom, and touch gestures for mobile devices
- **Pinch-to-Zoom**: Multi-touch pinch gestures for zooming on touch devices
- **Optional UI Controls**: On-screen zoom in/out/reset buttons
- **Viewport Management**: Automatic fitting, centering, and containment options
- **Event Callbacks**: Hooks for `beforeZoom`, `onZoom`, `beforePan`, `onPan`, and `onUpdatedCTM` events
- **Cross-browser Compatibility**: Includes IE-specific workarounds and normalized wheel events


How To Use
----------

Reference the [svg-pan-zoom.js file](https://github.com/bumbu/svg-pan-zoom/blob/master/dist/svg-pan-zoom.min.js) from your HTML document. Then call the init method:

```js
var panZoomTiger = svgPanZoom('#demo-tiger');
// or
var svgElement = document.querySelector('#demo-tiger')
var panZoomTiger = svgPanZoom(svgElement)
```

First argument to function should be a CSS selector of SVG element or a DOM Element.

If you want to override the defaults, you can optionally specify one or more arguments:

```js
svgPanZoom('#demo-tiger', {
  viewportSelector: '.svg-pan-zoom_viewport'
, panEnabled: true
, controlIconsEnabled: false
, zoomEnabled: true
, dblClickZoomEnabled: true
, mouseWheelZoomEnabled: true
, preventMouseEventsDefault: true
, zoomScaleSensitivity: 0.2
, minZoom: 0.5
, maxZoom: 10
, fit: true
, contain: false
, center: true
, refreshRate: 'auto'
, beforeZoom: function(){}
, onZoom: function(){}
, beforePan: function(){}
, onPan: function(){}
, onUpdatedCTM: function(){}
, customEventsHandler: {}
, eventsListenerElement: null
});
```

If any arguments are specified, they must have the following value types:
* 'viewportSelector' can be querySelector string or SVGElement.
* 'panEnabled' must be true or false. Default is true.
* 'controlIconsEnabled' must be true or false. Default is false.
* 'zoomEnabled' must be true or false. Default is true.
* 'dblClickZoomEnabled' must be true or false. Default is true.
* 'mouseWheelZoomEnabled' must be true or false. Default is true.
* 'preventMouseEventsDefault' must be true or false. Default is true.
* 'zoomScaleSensitivity' must be a scalar. Default is 0.2.
* 'minZoom' must be a scalar. Default is 0.5.
* 'maxZoom' must be a scalar. Default is 10.
* 'fit' must be true or false. Default is true.
* 'contain' must be true or false. Default is false.
* 'center' must be true or false. Default is true.
* 'refreshRate' must be a number or 'auto'
* 'beforeZoom' must be a callback function to be called before zoom changes.
* 'onZoom' must be a callback function to be called when zoom changes.
* 'beforePan' must be a callback function to be called before pan changes.
* 'onPan' must be a callback function to be called when pan changes.
* 'customEventsHandler' must be an object with `init` and `destroy` arguments as functions.
* 'eventsListenerElement' must be an SVGElement or null.

`beforeZoom` will be called with 2 float attributes: oldZoom and newZoom.
If `beforeZoom` will return `false` then zooming will be halted.

`onZoom` callbacks will be called with one float attribute representing new zoom scale.

`beforePan` will be called with 2 attributes:
* `oldPan`
* `newPan`

Each of these objects has two attributes (x and y) representing current pan (on X and Y axes).

If `beforePan` will return `false` or an object `{x: true, y: true}` then panning will be halted.
If you want to prevent panning only on one axis then return an object of type `{x: true, y: false}`.
You can alter panning on X and Y axes by providing alternative values through return `{x: 10, y: 20}`.

> *Caution!* If you alter panning by returning custom values `{x: 10, y: 20}` it will update only current pan step. If panning is done by mouse/touch you have to take in account that next pan step (after the one that you altered) will be performed with values that do not consider altered values (as they even did not existed).

`onPan` callback will be called with one attribute: `newPan`.

> *Caution!* Calling zoom or pan API methods from inside of `beforeZoom`, `onZoom`, `beforePan` and `onPan` callbacks may lead to infinite loop.

`onUpdatedCTM` will get called after the CTM will get updated. That happens asynchronously from pan and zoom events.

`panEnabled` and `zoomEnabled` are related only to user interaction. If any of this options are disabled - you still can zoom and pan via API.

`fit` takes precedence over `contain`. So if you set `fit: true` then `contain`'s value doesn't matter.

Embedding remote files
---------------------

If you're embedding a remote file like this
```html
<embed type="image/svg+xml" src="/path/to/my/file.svg" />
<object type="image/svg+xml" data="/path/to/my/file.svg">Your browser does not support SVG</object>
```

or you're rendering the SVG after the page loads then you'll have to call svgPanZoom library after your SVG is loaded.

One way to do so is by listening to load event:
```html
<embed type="image/svg+xml" src="/path/to/my/file.svg" id="my-embed"/>

<script>
document.getElementById('my-embed').addEventListener('load', function(){
  // Will get called after embed element was loaded
  svgPanZoom(document.getElementById('my-embed'));
})
</script>
```


Using a custom viewport
-----------------------

You may want to use a custom viewport if you have more layers in your SVG but you want to _pan-zoom_ only one of them.

By default if:
  * There is just one top-level graphical element of type SVGGElement (`<g>`)
  * SVGGElement has no `transform` attribute
  * There is no other SVGGElement with class name `svg-pan-zoom_viewport`

then the top-level graphical element will be used as viewport.

To specify which layer (SVGGElement) should be _pan-zoomed_ set the `svg-pan-zoom_viewport` class name to that element:
`<g class="svg-pan-zoom_viewport"></g>`.

> Do not set any _transform_ attributes to that element. It will make the library misbehave.
> If you need _transform_ attribute for viewport better create a nested group element and set _transforms_ to that element:
```html
<g class="svg-pan-zoom_viewport">
  <g transform="matrix(1,0,0,1,0,0);"></g>
</g>
```

You can specify your own viewport selector by altering `viewportSelector` config value:
```js
svgPanZoom('#demo-tiger', {
  viewportSelector: '.svg-pan-zoom_viewport'
});
// or
var viewportGroupElement = document.getElementById('demo-tiger').querySelector('.svg-pan-zoom_viewport');
svgPanZoom('#demo-tiger', {
  viewportSelector: viewportGroupElement
});
```

Listening for pan/zoom events on a child SVG element
----------------------------------------------------

If you want to listen for user interaction events from a child SVG element then use `eventsListenerElement` option. An example is available in [demo/layers.html](http://bumbu.github.io/svg-pan-zoom/demo/layers.html).

Use with browserify
-------------------

To use with browserify, follow these steps:
* Add the package as node module `npm install --save bumbu/svg-pan-zoom`
* Require _svg-pan-zoom_ in your source file `svgPanZoom = require('svg-pan-zoom')`
* Use in the same way as you would do with global svgPanZoom: `instance = svgPanZoom('#demo-tiger')`

Use with Require.js (or other AMD libraries)
-------------------

An example of how to load library using Require.js is available in [demo/require.html](http://bumbu.github.io/svg-pan-zoom/demo/require.html)

Custom events support
---------------------

You may want to add custom events support (for example double tap or pinch).

It is possible by setting `customEventsHandler` configuration option.
`customEventsHandler` should be an object with following attributes:
* `haltEventListeners`: array of strings
* `init`: function
* `destroy`: function

`haltEventListeners` specifies which default event listeners should be disabled (in order to avoid conflicts as svg-pan-zoom by default supports panning using touch events).

`init` is a function that is called when svg-pan-zoom is initialized. An object is passed into this function.
Passed object has following attributes:
* `svgElement` - SVGSVGElement
* `instance` - svg-pan-zoom public API instance

`destroy` is a function called upon svg-pan-zoom destroy

An example of how to use it together with [Hammer.js](http://hammerjs.github.io):
```js
var options = {
  zoomEnabled: true
, controlIconsEnabled: true
, customEventsHandler: {
    // Halt all touch events
    haltEventListeners: ['touchstart', 'touchend', 'touchmove', 'touchleave', 'touchcancel']

    // Init custom events handler
  , init: function(options) {
      // Init Hammer
      this.hammer = Hammer(options.svgElement)

      // Handle double tap
      this.hammer.on('doubletap', function(ev){
        options.instance.zoomIn()
      })
    }

    // Destroy custom events handler
  , destroy: function(){
      this.hammer.destroy()
    }
  }
}

svgPanZoom('#mobile-svg', options);
```

You may find an example that adds support for Hammer.js pan, pinch and doubletap in demo/mobile.html

Keep content visible/Limit pan
------------------------------

You may want to keep SVG content visible by not allowing panning over SVG borders.

To do so you may prevent or alter panning from `beforePan` callback. For more details take a look at `demo/limit-pan.html` example.

Public API
----------

When you call `svgPanZoom` method it returns an object with following methods:
* enablePan
* disablePan
* isPanEnabled
* pan
* panBy
* getPan
* setBeforePan
* setOnPan
* enableZoom
* disableZoom
* isZoomEnabled
* enableControlIcons
* disableControlIcons
* isControlIconsEnabled
* enableDblClickZoom
* disableDblClickZoom
* isDblClickZoomEnabled
* enableMouseWheelZoom
* disableMouseWheelZoom
* isMouseWheelZoomEnabled
* setZoomScaleSensitivity
* setMinZoom
* setMaxZoom
* setBeforeZoom
* setOnZoom
* zoom
* zoomBy
* zoomAtPoint
* zoomAtPointBy
* zoomIn
* zoomOut
* setOnUpdatedCTM
* getZoom
* resetZoom
* resetPan
* reset
* fit
* contain
* center
* updateBBox
* resize
* getSizes
* destroy

To programmatically pan, call the pan method with vector as first argument:

```js
// Get instance
var panZoomTiger = svgPanZoom('#demo-tiger');

// Pan to rendered point x = 50, y = 50
panZoomTiger.pan({x: 50, y: 50})

// Pan by x = 50, y = 50 of rendered pixels
panZoomTiger.panBy({x: 50, y: 50})
```

To programmatically zoom, you can use the zoom method to specify your desired scale value:

```js
// Get instance
var panZoomTiger = svgPanZoom('#demo-tiger');

// Set zoom level to 2
panZoomTiger.zoom(2)

// Zoom by 130%
panZoomTiger.zoomBy(1.3)

// Set zoom level to 2 at point
panZoomTiger.zoomAtPoint(2, {x: 50, y: 50})

// Zoom by 130% at given point
panZoomTiger.zoomAtPointBy(1.3, {x: 50, y: 50})
```

> Zoom is relative to initial SVG internal zoom level. If your SVG was fit at the beginning (option `fit: true`) and thus zoomed in or out to fit available space - initial scale will be 1 anyway.

Or you can use the zoomIn or zoomOut methods:

```js
// Get instance
var panZoomTiger = svgPanZoom('#demo-tiger');

panZoomTiger.zoomIn()
panZoomTiger.zoomOut()
panZoomTiger.resetZoom()
```

If you want faster or slower zooming, you can override the default zoom increment with the setZoomScaleSensitivity method.

To programmatically enable/disable pan or zoom:

```js
// Get instance
var panZoomTiger = svgPanZoom('#demo-tiger');

panZoomTiger.enablePan();
panZoomTiger.disablePan();

panZoomTiger.enableZoom();
panZoomTiger.disableZoom();
```

To fit and center (you may try `contain` instead of `fit`):

```js
// Get instance
var panZoomTiger = svgPanZoom('#demo-tiger');

panZoomTiger.fit();
panZoomTiger.center();
```

If you want to fit and center your SVG after its container resize:

```js
// Get instance
var panZoomTiger = svgPanZoom('#demo-tiger');

panZoomTiger.resize(); // update SVG cached size and controls positions
panZoomTiger.fit();
panZoomTiger.center();
```

If you update SVG (viewport) contents so its border box (virtual box that contains all elements) changes, you have to call `updateBBox`:

```js
var panZoomTiger = svgPanZoom('#demo-tiger');
panZoomTiger.fit();

// Update SVG rectangle width
document.getElementById('demo-tiger').querySelector('rect').setAttribute('width', 200)

// fit does not work right anymore as viewport bounding box changed
panZoomTiger.fit();

panZoomTiger.updateBBox(); // Update viewport bounding box
panZoomTiger.fit(); // fit works as expected
```

If you need more data about SVG you can call `getSizes`. It will return an object that will contain:
* `width` - SVG cached width
* `height` - SVG cached height
* `realZoom` - _a_ and _d_ attributes of transform matrix applied over viewport
* `viewBox` - an object containing cached sizes of viewport boxder box
  * `width`
  * `height`
  * `x` - x offset
  * `y` - y offset

Destroy SvgPanZoom instance:

```js
// Get instance
var panZoomTiger = svgPanZoom('#demo-tiger');

panZoomTiger.destroy();
delete panZoomTiger;
```
