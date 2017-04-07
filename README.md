# api documentation for  [yo-yo (v1.4.0)](https://github.com/maxogden/yo-yo#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-yo-yo.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-yo-yo) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-yo-yo.svg)](https://travis-ci.org/npmdoc/node-npmdoc-yo-yo)
#### A tiny library for building modular UI components using DOM diffing and ES6 tagged template literals

[![NPM](https://nodei.co/npm/yo-yo.png?downloads=true)](https://www.npmjs.com/package/yo-yo)

[![apidoc](https://npmdoc.github.io/node-npmdoc-yo-yo/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-yo-yo_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-yo-yo/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-yo-yo/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-yo-yo/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Max Ogden"
    },
    "bugs": {
        "url": "https://github.com/maxogden/yo-yo/issues"
    },
    "dependencies": {
        "bel": "^4.0.0",
        "morphdom": "^2.1.0"
    },
    "description": "A tiny library for building modular UI components using DOM diffing and ES6 tagged template literals",
    "devDependencies": {
        "browserify": "^13.0.1",
        "tape": "^4.5.1",
        "wzrd": "^1.3.1"
    },
    "directories": {},
    "dist": {
        "shasum": "c3291e512bb4743f9eb514b9b6dbb264757e3257",
        "tarball": "https://registry.npmjs.org/yo-yo/-/yo-yo-1.4.0.tgz"
    },
    "gitHead": "b4e1e8fe2e1081464c1fbdd1ad6c7a0ae7e24ad1",
    "homepage": "https://github.com/maxogden/yo-yo#readme",
    "keywords": [],
    "license": "ISC",
    "main": "index.js",
    "maintainers": [
        {
            "name": "freeman-lab",
            "email": "the.freeman.lab@gmail.com"
        },
        {
            "name": "maxogden",
            "email": "max@maxogden.com"
        },
        {
            "name": "shama",
            "email": "kyle@dontkry.com"
        },
        {
            "name": "yoshuawuyts",
            "email": "i@yoshuawuyts.com"
        }
    ],
    "name": "yo-yo",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+https://github.com/maxogden/yo-yo.git"
    },
    "scripts": {
        "test": "wzrd test.js"
    },
    "version": "1.4.0"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module yo-yo](#apidoc.module.yo-yo)
1.  [function <span class="apidocSignatureSpan">yo-yo.</span>createElement (tag, props, children)](#apidoc.element.yo-yo.createElement)
1.  [function <span class="apidocSignatureSpan">yo-yo.</span>default (strings)](#apidoc.element.yo-yo.default)
1.  [function <span class="apidocSignatureSpan">yo-yo.</span>update (fromNode, toNode, opts)](#apidoc.element.yo-yo.update)



# <a name="apidoc.module.yo-yo"></a>[module yo-yo](#apidoc.module.yo-yo)

#### <a name="apidoc.element.yo-yo.createElement"></a>[function <span class="apidocSignatureSpan">yo-yo.</span>createElement (tag, props, children)](#apidoc.element.yo-yo.createElement)
- description and source-code
```javascript
function belCreateElement(tag, props, children) {
  var el

  // If an svg tag, it needs a namespace
  if (SVG_TAGS.indexOf(tag) !== -1) {
    props.namespace = SVGNS
  }

  // If we are using a namespace
  var ns = false
  if (props.namespace) {
    ns = props.namespace
    delete props.namespace
  }

  // Create the element
  if (ns) {
    el = document.createElementNS(ns, tag)
  } else if (tag === COMMENT_TAG) {
    return document.createComment(props.comment)
  } else {
    el = document.createElement(tag)
  }

  // If adding onload events
  if (props.onload || props.onunload) {
    var load = props.onload || function () {}
    var unload = props.onunload || function () {}
    onload(el, function belOnload () {
      load(el)
    }, function belOnunload () {
      unload(el)
    },
    // We have to use non-standard 'caller' to find who invokes 'belCreateElement'
    belCreateElement.caller.caller.caller)
    delete props.onload
    delete props.onunload
  }

  // Create the properties
  for (var p in props) {
    if (props.hasOwnProperty(p)) {
      var key = p.toLowerCase()
      var val = props[p]
      // Normalize className
      if (key === 'classname') {
        key = 'class'
        p = 'class'
      }
      // The for attribute gets transformed to htmlFor, but we just set as for
      if (p === 'htmlFor') {
        p = 'for'
      }
      // If a property is boolean, set itself to the key
      if (BOOL_PROPS[key]) {
        if (val === 'true') val = key
        else if (val === 'false') continue
      }
      // If a property prefers being set directly vs setAttribute
      if (key.slice(0, 2) === 'on') {
        el[p] = val
      } else {
        if (ns) {
          if (p === 'xlink:href') {
            el.setAttributeNS(XLINKNS, p, val)
          } else if (/^xmlns($|:)/i.test(p)) {
            // skip xmlns definitions
          } else {
            el.setAttributeNS(null, p, val)
          }
        } else {
          el.setAttribute(p, val)
        }
      }
    }
  }

  function appendChild (childs) {
    if (!Array.isArray(childs)) return
    for (var i = 0; i < childs.length; i++) {
      var node = childs[i]
      if (Array.isArray(node)) {
        appendChild(node)
        continue
      }

      if (typeof node === 'number' ||
        typeof node === 'boolean' ||
        typeof node === 'function' ||
        node instanceof Date ||
        node instanceof RegExp) {
        node = node.toString()
      }

      if (typeof node === 'string') {
        if (el.lastChild && el.lastChild.nodeName === '#text') {
          el.lastChild.nodeValue += node
          continue
        }
        node = document.createTextNode(node)
      }

      if (node && node.nodeType) {
        el.appendChild(node)
      }
    }
  }
  appendChild(children)

  return el
}
```
- example usage
```shell
...

Running this produces 'h1 {} [ 'hello world' ]', which aren't yet DOM elements but have all the data you need to build your own
DOM elements however you like. These three arguments, 'tagName, attrs, children' are a sort of pseudo-standard used by various DOM
 building libraries such as [virtual-dom](https://www.npmjs.com/package/virtual-dom), [hyperscript](https://www.npmjs.com/package
/hyperscript) and [react](https://facebook.github.io/react/docs/glossary.html#react-elements), and now 'hyperx' and 'bel'.

You can also use DOM elements not created using 'hyperx' and 'bel':

'''js
var yo = require('yo-yo')
var vanillaElement = document.createElement('h3')
vanillaElement.textContent = 'Hello'

var app = yo'<div class="app">${vanillaElement} World</div>'
'''

Running the above sets 'app' to an element with this HTML:
...
```

#### <a name="apidoc.element.yo-yo.default"></a>[function <span class="apidocSignatureSpan">yo-yo.</span>default (strings)](#apidoc.element.yo-yo.default)
- description and source-code
```javascript
default = function (strings) {
  var state = TEXT, reg = ''
  var arglen = arguments.length
  var parts = []

  for (var i = 0; i < strings.length; i++) {
    if (i < arglen - 1) {
      var arg = arguments[i+1]
      var p = parse(strings[i])
      var xstate = state
      if (xstate === ATTR_VALUE_DQ) xstate = ATTR_VALUE
      if (xstate === ATTR_VALUE_SQ) xstate = ATTR_VALUE
      if (xstate === ATTR_VALUE_W) xstate = ATTR_VALUE
      if (xstate === ATTR) xstate = ATTR_KEY
      p.push([ VAR, xstate, arg ])
      parts.push.apply(parts, p)
    } else parts.push.apply(parts, parse(strings[i]))
  }

  var tree = [null,{},[]]
  var stack = [[tree,-1]]
  for (var i = 0; i < parts.length; i++) {
    var cur = stack[stack.length-1][0]
    var p = parts[i], s = p[0]
    if (s === OPEN && /^\//.test(p[1])) {
      var ix = stack[stack.length-1][1]
      if (stack.length > 1) {
        stack.pop()
        stack[stack.length-1][0][2][ix] = h(
          cur[0], cur[1], cur[2].length ? cur[2] : undefined
        )
      }
    } else if (s === OPEN) {
      var c = [p[1],{},[]]
      cur[2].push(c)
      stack.push([c,cur[2].length-1])
    } else if (s === ATTR_KEY || (s === VAR && p[1] === ATTR_KEY)) {
      var key = ''
      var copyKey
      for (; i < parts.length; i++) {
        if (parts[i][0] === ATTR_KEY) {
          key = concat(key, parts[i][1])
        } else if (parts[i][0] === VAR && parts[i][1] === ATTR_KEY) {
          if (typeof parts[i][2] === 'object' && !key) {
            for (copyKey in parts[i][2]) {
              if (parts[i][2].hasOwnProperty(copyKey) && !cur[1][copyKey]) {
                cur[1][copyKey] = parts[i][2][copyKey]
              }
            }
          } else {
            key = concat(key, parts[i][2])
          }
        } else break
      }
      if (parts[i][0] === ATTR_EQ) i++
      var j = i
      for (; i < parts.length; i++) {
        if (parts[i][0] === ATTR_VALUE || parts[i][0] === ATTR_KEY) {
          if (!cur[1][key]) cur[1][key] = strfn(parts[i][1])
          else cur[1][key] = concat(cur[1][key], parts[i][1])
        } else if (parts[i][0] === VAR
        && (parts[i][1] === ATTR_VALUE || parts[i][1] === ATTR_KEY)) {
          if (!cur[1][key]) cur[1][key] = strfn(parts[i][2])
          else cur[1][key] = concat(cur[1][key], parts[i][2])
        } else {
          if (key.length && !cur[1][key] && i === j
          && (parts[i][0] === CLOSE || parts[i][0] === ATTR_BREAK)) {
            // https://html.spec.whatwg.org/multipage/infrastructure.html#boolean-attributes
            // empty string is falsy, not well behaved value in browser
            cur[1][key] = key.toLowerCase()
          }
          break
        }
      }
    } else if (s === ATTR_KEY) {
      cur[1][p[1]] = true
    } else if (s === VAR && p[1] === ATTR_KEY) {
      cur[1][p[2]] = true
    } else if (s === CLOSE) {
      if (selfClosing(cur[0]) && stack.length) {
        var ix = stack[stack.length-1][1]
        stack.pop()
        stack[stack.length-1][0][2][ix] = h(
          cur[0], cur[1], cur[2].length ? cur[2] : undefined
        )
      }
    } else if (s === VAR && p[1] === TEXT) {
      if (p[2] === undefined || p[2] === null) p[2] = ''
      else if (!p[2]) p[2] = concat('', p[2])
      if (Array.isArray(p[2][0])) {
        cur[2].push.apply(cur[2], p[2])
      } else {
        cur[2].push(p[2])
      }
    } else if (s === TEXT) {
      cur[2].push(p[1])
    } else if (s === ATTR_EQ || s === ATTR_BREAK) {
      // no-op
    } else {
      throw new Error('unhandled: ' + s)
    }
  }

  if (tree[2].length > 1 && /^\s*$/.test(tree[2][0])) {
    tree[2].shift()
  }

  if (tree[2].length > 2
  || (tree[2].length === 2 && /\S/.test(tree[2][1]))) {
    throw new Error(
      'multiple root elements must be wrapped in an enclosing tag'
    )
  }
  if (Array.isArray(tree[2][0]) && typeof tree[2][0][0] === 'string'
  && Array.isArray(tree[2][0][2])) {
    tree[2][0] = h(tree[2][0][0], tree[2][0][1], tree[2][0][2])
  }
  return tree[2][0]

  function parse (str) {
    var res = []
    if (state === ATTR_VALUE_W) sta ...
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.yo-yo.update"></a>[function <span class="apidocSignatureSpan">yo-yo.</span>update (fromNode, toNode, opts)](#apidoc.element.yo-yo.update)
- description and source-code
```javascript
update = function (fromNode, toNode, opts) {
  if (!opts) opts = {}
  if (opts.events !== false) {
    if (!opts.onBeforeElUpdated) opts.onBeforeElUpdated = copier
  }

  return morphdom(fromNode, toNode, opts)

  // morphdom only copies attributes. we decided we also wanted to copy events
  // that can be set via attributes
  function copier (f, t) {
    // copy events:
    var events = opts.events || defaultEvents
    for (var i = 0; i < events.length; i++) {
      var ev = events[i]
      if (t[ev]) { // if new element has a whitelisted attribute
        f[ev] = t[ev] // update existing element
      } else if (f[ev]) { // if existing element has it and new one doesnt
        f[ev] = undefined // remove it from existing element
      }
    }
    var oldValue = f.value
    var newValue = t.value
    // copy values for form elements
    if ((f.nodeName === 'INPUT' && f.type !== 'file') || f.nodeName === 'SELECT') {
      if (!newValue) {
        t.value = f.value
      } else if (newValue !== oldValue) {
        f.value = newValue
      }
    } else if (f.nodeName === 'TEXTAREA') {
      if (t.getAttribute('value') === null) f.value = t.value
    }
  }
}
```
- example usage
```shell
...

Returns the 'yo' function. There is also a method on 'yo' called 'yo.update'.

### yo\'template\'

'yo' is a function designed to be used with [tagged template literals](#tagged-template-literals). If your template produces a string
 containing an HTML element, the 'yo' function will take it and produce a new DOM element that you can insert into the DOM.

### yo.update(targetElement, newElement, [opts])

Efficiently updates the attributes and content of an element by [diffing and morphing](#morphdom) a new element onto an existing
 target element. The two elements + their children should have the same 'shape', as the diff between 'newElement' will replace nodes
 in 'targetElement'. 'targetElement' will get efficiently updated with only the new DOM nodes from 'newElement', and 'newElement
' can be discarded afterwards.

Note that many properties of a DOM element **are ignored** when elements are updated. [morphdom](#morphdom) only copies the following
 properties:

- 'node.firstChild'
- 'node.tagName'
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
