# XSS Discovery

## General

So in general, XSS can be found in any value I control (parameters, path, headers, cookies).

There are three types of XSS:

## DOM XSS

DOM-based XSS is an XSS vulnerability that occurs when user input goes into a field (source) and passes into sinks without sanitization.

#### SOURCE

Any place where attacker-controlled data enters JavaScript.

Example sources:

##### URL Based

```
location.href
location.search // ?q=...
document.URL
```

#### Others

```
document.cookie
window.name
```

Example:

```
https://site.com/page?input=
```

```
let input = location.search;
```

### SINK

A sink is the execution point where the data is being passed (it's the function that executes the attacker-controlled input).

#### HTML Sinks

Executes JS directly:

```
element.innerHTML
document.write()
outerHTML
insertAdjacentHTML()
attr() // jQuery library
```

Sometimes executable (e.g. `javascript:`):

```
element.src
element.href
```

Example:

```
document.body.innerHTML = userInput;
```

For the `attr` sink, jQuery library uses it to modify DOM attributes, so I can find:

```
href="myinput"
```

which I can use:

```
javascript:mycode
```

to execute JS.

### Remark

- The `innerHTML` sink doesn't accept the `script` tag, nor will SVG `onload` events fire. That means I can use the alternative `/` tag (with `onload` and `onerror`).

## Full DOM XSS Flow

```
let input = location.search;
document.body.innerHTML = input;
```

1. Browser loads page.
    
2. JS reads `location.search`.
    
3. Passes it into `innerHTML`.
    
4. Browser parses HTML → executes the input.
    

## How to Identify DOM XSS

### 1. Find Sources

Search in the source code (DevTools) for:

```
location
document.cookie
referrer
localStorage
```

Find where user input enters JS.

### 2. Track the Data Flow

Example:

```
let q = location.search;
let clean = decodeURIComponent(q);
document.write(clean);
```

Flow:

```
location.search -> clean -> document.write
```

### 3. Find Sinks

Search for dangerous functions like:

```
innerHTML
document.write
eval
setTimeout
```

Then check whether any of them receive data from the source.

## Remarks

```
location.search
```