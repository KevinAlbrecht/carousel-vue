# Carousel vue

This component is a simple carousel, inspired by the netflix's movies one.
The carousel can be finite or looping endlessly in both ways.

## Demo
(available soon)
## Add to your project
```bash
npm i @kalbrecht/carousel-vue
```
## Project setup from sources
```bash
npm install
npm run build && npm pack
```

**/!\ Attention, the "bp" script use the windows command "move", switching to shelljs is in TODOs**

## Doc
### Props
- **items**: the actual list of items to use in the carousel ( a complete list is needed for the moment ) you can use like the following : 
```html
<KalCarousel :items="latest">
	<template v-slot:default="props">
		{{props.item}}
	</template>
</KalCarousel>
```
- **title**: A title you want to show (will be removed)
- **chunkSize**: The size of elements you want to show
- **loop**: let you decide if the carousel has to loop endlessly or stop when you reach the last chunk./!\ note that , by design, when the carousel is build it act as a "finite" carousel for the first action, so you have to scroll once at right for "activating" the loop. ( That will be a customization option later ).
- **translationDuration**: the time in miliseconds for the slide animation ( 700 by default )

### Events
**/!\ All those event will be reworked ( name and payload ).**

- **slide_left**: when the user slide left, to see previous items. the payload represent as following
```js
{
currentChunk:number,
directionLeft:boolean
}
```
- **slide_right**: when the user slide right, to see next items. The payload is the same as the slide_left.
- **i_click**: when the user click on an item, the payload is the current item.
## Roadmap
- manage empty input
- allow async items input
- portal for items
- add ShellJS for packaging command.
- customising arrows
- vue3

## Todos
- improve styles
- fix package dependencies
- rework events
- refactoring