---
layout: post
title: Choo and js libraries
---

I think i have **fallen in love with choo**. It works just how I want it to and you need to [check it out](https://github.com/yoshuawuyts/choo).
But it is still in its infancy and not everything has been worked out yet. One of those things are how to use **external libraries in choo**.

### So this is my take on including external js libs:

choo uses template strings, vanilla es2015 awesomeness.
I want only to call one simple function.
```javascript
const html = require('choo/html')

const view = (state, prev, send) => html`
  <main>
    <h1>Chart</h1>
    // one simple function
    ${ chart(state) }
  </main>
`
```

We are going to create it inside a closure and return a func. This is a [d3](https://d3js.org/) chart.
```javascript
const chart = (function (){
  // this is a closure, here we put internal stuff and init things
  const element = document.createElement('div')
  const svg = svg = d3.select(element).append("svg") // ++ height, width
  cont rect = svg.append("rect").attr("x", 0).attr("y", 0) // ++ more attrs.
  
  // this is the function that will be called with chart(state)
  return function (state) {
    // we set and update values by calling functions on the d3 lib
    rect.transition().duration(750).attr('width', state.value)
  
    // always return the same element
    return element
  }
})()
```

### The result
<iframe width="560" height="315" src="http://requirebin.com/embed?gist=d3477f03bcd3293cfa632bbd43c4985a" frameborder="0" allowfullscreen></iframe>

### Diffing and Dispatch

We can also do some more fancy stuff, like diffing and dispatching choo actions from the external library code.
This is a full integration of the [leaflet](http://leafletjs.com/) map client:
```javascript
const html = require('choo/html')
const L = require('leaflet')

const view = (state, prev, send) => html`
  <main>
    <h1>Map</h1>
    ${ map(state, prev, send) }
  </main>
`

const map = (function (){
  const element = document.createElement('div')
  const map = L.map(element).setView([60, 10], 6)
  var markers = {}
  
  // this is the function that will be called with map(state, prev, send)
  return function (state, prev, send) {
    // diff and find new markers
    /*psuedocode*/diff(state.points, prev.points).forEach((newId)=>{
      markers[newId] = L.circle(points[newId],10000, circleProps).addTo(map)
      marker.on('click',()=>send('remove',{ id: newId }))
    })
    // diff and find removed markers
    /*psuedocode*/diff(prev.points, state.points).forEach((remId)=>{
      map.removeLayer(markers[remId])
      delete markers[remId]
    })
  
    // always return the same element
    return element
  }
})()
```

### The result
<iframe width="560" height="315" src="http://requirebin.com/embed?gist=970e917827aa636e0c9b4de784927d2d" frameborder="0" allowfullscreen></iframe>

### Afterword
This method works really well when you have **one instance** of a library you want to include into the choo view.
But if you need multiple charts, you need to instanciate all the charts you need outside of the choo view lifecycle.

I have some ideas on how to solve this, but that is a post for another day.

