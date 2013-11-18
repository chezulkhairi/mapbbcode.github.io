---
layout: default
title: Плагины для Leaflet
needmap: true
---

В процессе разработки MapBBCode было написано немало плагинов для библиотеки Leaflet. Здесь находится документация по ним всем, вместе с примерами и прямыми ссылками на скачивание. Исходники находятся там же, где и остальной MapBBCode: в [репозитории GitHub](https://github.com/MapBBCode/mapbbcode).

## L.LetterIcon

Круглая иконка с белой рамкой и текстом внутри. Полезно для отображения маркеров, которые не требуют клика или наведения курсора для того, чтобы прочитать их текст.

| Опция | Тип | По умолчанию | Описание
|---|---|---|---
| `color` | String | `'black'` | Цвет CSS фона иконки.
| `radius` | Number | `11` | Радиус иконки.

[Скачать](https://raw.github.com/MapBBCode/mapbbcode/master/src/LetterIcon.js)

```javascript
L.marker([11, 22], { icon: L.letterIcon('Big', { radius: 20 }), clickable: false }).addTo(map);
```

<div class="map" id="mapli"></div>

## L.PopupIcon

Иконка, которая выглядит как всплывающая панель, но значительно меньше её. Надпись нужно указывать в конструкторе объекта. Опция только одна: `width` с максимальной шириной иконки.

[Скачать](https://raw.github.com/MapBBCode/mapbbcode/master/src/PopupIcon.js)

```javascript
L.marker([11, 22], { icon: L.popupIcon("Don't click me"), clickable: false }).addTo(map);
```

<div class="map" id="mappi"></div>

## L.FunctionButtons

This control simplifies creating simple Leaflet controls that invoke javascript functions when clicked. It supports multiple actions on one control: in that case they are stacked vertically, like on a standard zoom control.

Class contructor accepts two parameters: an array of labels for actions (strings, image URLs or html elements) and an options object. The latter may contain `titles` array of strings, which would be used as title attributes, and `bgPos` array of [dx, dy] arrays that define offsets for image content. Other options (one option, actually: `position`) are inherited from the [L.Control](http://leafletjs.com/reference.html#control) class.

An action inside a function button can be updated with `setContent(id, content)` and `setTitle(id, title)` methods. Image content offset can be altered with `setBgPos(id, bgPos)` method.

When clicked, the control emits a Leaflet event `clicked` with a single data property, `idx`: zero-based index of an action that was selected.

[Скачать](https://raw.github.com/MapBBCode/mapbbcode/master/src/FunctionButton.js)

```javascript
var btn = L.functionButtons(['Saint-Petersburg', 'Hide buttons']);
btn.on('clicked', function(data) {
	if( data.idx == 0 ) {
		map.setView([59.939, 30.315], 13);
	} else {
		map.removeControl(btn);
	}
});
map.addControl(btn);
```

<div class="map" id="mapfb"></div>

### L.FunctionButton

This is a subclass of `L.FunctionButtons` designed to work with a single action. It accepts a single object with options in the contructor, its `setContent()`, `setTitle()` and `setBgPos()` methods accept a single parameter.

## L.StaticLayerSwitcher

This control can replace the standard Leaflet layers control. It allows a user to switch layers and, if `editable` property is set, to remove layers and change their order. This control differs from the stanard one in two ways:

* It does not support overlay layers.
* It fully controls layers included in a map, to the point of removing any layer that it doesn't have listed.

Each layer has to have a label (id). It is either specified on adding, or included in the layer's `options` object as a `name` property. The plugin can be used together with `window.layerList` module: in this case layers can be managed as a list of ids.

The constructor accepts two parameters. The first one is a layer list, either an array of ids or an object `{ id: layer, ... }`. If a layer is not specified, it is requested from `window.layerList` for a given id. The second optional parameter is an options object with the following properties:

| Опция | Тип | По умолчанию | Описание
|---|---|---|---
| `maxLayers` | Number | `7` | Maximum number of layers on a map.
| `bgColor` | String | `'white'` | Background CSS color of a layer switcher.
| `selectedColor` | String | `#ddd` | Background CSS color of a selected layer line.
| `editable` | Boolean | `false` | Whether a user can move and delete layers.

Those methods can be used to get and manipulate layers in a layer switcher:

* `<ILayer[]> getLayers()`: returns an array of layers included in a control, in their displayed order.
* `<String[]> getLayersIds()`: returns an array of layer identifiers included in a control, in their displayed order.
* `<ILayer>   getSelectedLayer()`: returns a currently selected layer.
* `<String>   getSelectedLayerId()`: returns an identifier of a currently selected layer.
* `<ILayer>   addLayer( <String> id, <ILayer> layer )`: appends a layer with given id to the end of the layer list. If `layer` is omitted, `window.layerList` is queried for a given `id`. Returns a layer that was added, or `null` if operation failed.
* `<ILayer>   updateLayer( <ILayer> layer, <String> id )`: updated an identifier for a layer. In case the layer was created by `window.layerList`, recreates it. Returns a layer, an id for which was updated, or `null` if operation failed.
* `<ILayer>   removeLayer( <ILayer> layer )`: Removes a layer from the list. Returns a layer that was removed, or `null` if operation failed.
* `           moveLayer( <ILayer> layer, <Boolean> moveDown )`: moves a layer in the list either up or down.

When active, the layer switcher emits following Leaflet events:

| Event | Data | When fired
|---|---|---
| `selectionchanged` | `{ <ILayer> selected, <String> selectedId }` | A user has changed active layer.
| `layerschanged` | `{ <String[]> layerIds }` | List of layers has been changed (only when it is editable).

[Скачать](https://raw.github.com/MapBBCode/mapbbcode/master/src/StaticLayerSwitcher.js)

``` javascript
map.addControl(L.staticLayerSwitcher([
    'OpenMapSurfer', 'CycleMap', 'Humanitarian'
], { editable: true }));
```

<div class="map" id="mapls"></div>

## window.layerList

This object (not a class!) holds a list of layers that can be used on a leaflet map, and more specifically, as a MapBBCode base layer. It is by no means complete and includes only the most popular and distinctive layers.

The list itself is in the `list` property: it is an array of strings, which have to be passed to `eval()` to be converted into a Leaflet `ILayer` object. Some of entries contain `{key:<url>}` substring. It means that the layer requires a developer key (and the link is for its website). This substring has to be replaced with an actual key.

The object has some methods to simplify working with the layer list:

* `<String[]> getSortedKeys()`: returns a sorted list of layer keys (every key is essentially a human-readable label).
* `<Boolean>  requiresKey( <String> id )`: checks if the layer for a given id requires a developer key.
* `<String>   getKeyLink( <String> id )`: returns an URL for a developer key required for a layer, or an empty string if there is no URL or the layer does not need a key.
* `<ILayer[]> getLeafletLayers( <String[]> ids, <Leaflet> L )`: converts an array of ids to array of layers ready to be added to a Leaflet map.

[Скачать](https://raw.github.com/MapBBCode/mapbbcode/master/src/LayerList.js)

``` javascript
map.addLayer(window.layerList.getLeafletLayer('OpenStreetMap'));
```

<select size="1" id="llselect"><input type="button" id="lladd" value="Show layer">
<div class="map" id="mapll"></div>

## L.Control.Search

Every search control on the Leaflet plugins page has flaws. This is an attempt on making a simple, good-looking (though not as good as MapBox's closed-source one) search control. You just click a button, type a string and press Enter key. There are only two configurable options:

* `title`: title text on the control.
* `email`: e-mail address that will be sent to Nominatim server. It can be used for determining a source of suspicious activity. You should use this option, especially if the control is installed on a popular website.

[Скачать](https://raw.github.com/MapBBCode/mapbbcode/master/src/Leaflet.Search.js)

``` javascript
map.addControl(L.control.search());
```

<div class="map" id="mapcs"></div>

## L.ExportControl

An export button for maps downloaded from an external service. Gets the supported formats list from the server.

| Опция | Тип | По умолчанию | Описание
|---|---|---|---
| `name` | String | 'Export' | Text written on a button.
| `title` | String | `''` | Title text for the button.
| `endpoint` | String | 'http://share.mapbbcode.org/' | URL of the map sharing service.
| `codeid` | String | `''` | Identifier of a map to export.
| `types` | String[] | `[]` | List of supported formats (if empty, then it is downloaded).
| `titles` | String[] | `[]` | Titles for supported formats (if empty, then it is downloaded).

[Скачать](https://raw.github.com/MapBBCode/mapbbcode/master/src/ExportButton.js)

``` javascript
map.addControl(L.exportControl({ codeid: 'nwrxs' }));
```

<div class="map" id="mapec"></div>

## L.Control.PermalinkAttribution

Replaces `L.Control.Attribution` with itself, so you won't have to do anything besides including the plugin script. Makes OpenStreetMap links in attribution into permanent links to the displayed place on osm.org website.

An option `attributionEditLink` is added to `L.Map` class. If it is set to `true`, OSM links will be followed by an edit link.

[Скачать](https://raw.github.com/MapBBCode/mapbbcode/master/src/PermalinkAttribution.js)

<div class="map" id="mappa"></div>

{% include plugins-demo.html %}