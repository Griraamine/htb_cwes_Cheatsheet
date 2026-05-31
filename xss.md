# xss discovery

## general

so in general, xss can be found in any value I control( parameters, path , headers, cookies)

There are three types of xss:

## reflected xss

DOM based xss is an xss vulnerability that occurs when userinput goes into a field(source) and passes into sinks without sanitization.

SOURCE: any place where attacker-controlled data enters JavaScript
example sources: URL BASED: location.href
                 location.search  // ?q=...
                 document.URL

         OTHERS: document.cookie
             window.name 

Example:
https://site.com/page?input=<payload>

let input = location.search;

SINK: is the execution point, where the data is being passed(its the function that executes the attacker-controlled input)  

HTML sinks:
    executes JS directly:   element.innerHTML
                    document.write()
                    outerHTML
                    insertAdjacentHTML()
                attr(jQuery library)

    sometimes executable(ex: javascript:) : element.src
                        element.href

Example:  document.body.innerHTML = userInput;
      for the attr sink, JQuery library uses it to modify DOM attributes, so I can find " href='myinput' " which I can use javascript:mycode on it to execute JS

#remark: the innerHTML sink doesnt accept script tag, nor will svg onload events fire, that means I can use the alternative <img> / <iframe> tag(with onload and onerror)

FULL DOM XSS FLOW: 

let input = location.search;
document.body.innerHTML = input;

1 browser loads page
2 JS reads location.search 
3 passes it into innerHMTL
4 browser parses HTML -> executes the input 

HOW TO IDENTIFY DOM XSS: 

1 FIND SOURCES:
search in the source code(Dev Tools) for location, document.cookie, referrer, localStorage -> find where userinput enters JS 

2 TRACK THE DATA FLOW 
example: 
let q = location.search; 
let clean = decodeURIComponent(q)
document.write(clean);

location.search -> clean -> document.write

3 FIND SINKS: 
search for dangerous functions like innerHTML, document.write, eval, setTimeout then check does any of them receive data from the source

# Remarks

 location.search 
