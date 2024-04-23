---
title: Server side SVG generation using D3.js
permalink: /content/server-side-svg-generation-using-d3js
---

Dynamic graphics generation for displaying charts in a web browser from data passed from the server is a nifty trick.
Sometimes however it may be desirable to generate SVG content on the server side, e.g. to lessen the demand on the
client or to store the chart on the server's file system.

I had written some JavaScript modules which used [D3.js](http://d3js.org/) to generate some graphs in the clients web browser. I then
realised that it would be nice to generate these same graphs on the server side without rewriting the code in Java. So
basically I wanted to run the same JavaScript code in the client's web browser and on the server's JVM.

I soon found Mozilla's [Rhino](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Rhino) JavaScript interpreter for Java. The problem however was that Rhino doesn't provide a full
browser like environment, it is lacking any APIs for manipulating the [Document Object Model](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model) (DOM). I found several
implementations of the DOM for platforms such as Node.js but they weren't compatible with Rhino. What I needed was a
pure JavaScript implementation.

I initially started working with [Envjs](http://www.envjs.com/) but found it was outdated and didn't properly support SVG elements. I did get it
to work eventually but it was just a real mess so I looked for alternatives and discovered Felix Gnass' domino project.
Domino is based on work by Mozilla's Andreas Gal and appears to be currently maintained. While it didn't work out of the
box, it was easy to patch. The modifications which I made were as follows:

* Change files to use AMD style loading instead of CommonJS (so I can load them in Rhino and the browser using [RequireJS](http://requirejs.org/))
* Add xmlns attribute to SVG element when serializing
* Modified the CSS parser to workaround a issue in Rhino with case insensitive regular expressions
* Added a SVG element type which has a style property like the HTML elements
* Added SVG specific CSS properties to the CSS parser
* Several bug fixes

You can find the patched version of domino on my GitHub account - [https://github.com/jazdw/domino](https://github.com/jazdw/domino).

Here's some sample code to get you started.

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.Reader;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.mozilla.javascript.Context;
import org.mozilla.javascript.Scriptable;
import org.mozilla.javascript.ScriptableObject;
import org.mozilla.javascript.tools.shell.Global;

import com.google.common.base.Charsets;

public abstract class RunJS {
    static final Log LOG = LogFactory.getLog(RunJS.class);
    
    static final int JS_VERSION = Context.VERSION_1_8;
    static final int OPTIMIZATION_LEVEL = 9;
    
    private static ScriptableObject ENVIRONMENT = null;
    
    static {
        ENVIRONMENT = createNewEnvironment();
    }
    
    /**
     * Creates a top level scope with the shell globals and requirejs then loads
     * bootstrap.js
     * 
     * The bootstrap script can then load other modules to be included in the top level
     * scope using requirejs
     */
    public static ScriptableObject createNewEnvironment() {
        Global global = null;
        
        Context cx = Context.enter();
        try {
            cx.setOptimizationLevel(OPTIMIZATION_LEVEL);
            cx.setLanguageVersion(JS_VERSION);
            
            global = new Global();
            global.setSealedStdLib(true);
            global.init(cx);

            Scriptable argsObj = cx.newArray(global, new Object[] {});
            global.defineProperty("arguments", argsObj, ScriptableObject.DONTENUM);
            
            // Enable RequireJS
            loadJS(cx, global, "/path/to/r.js");

            // Load the bootstrap file
            loadJS(cx, global, "/path/to/bootstrap.js");
            
            global.sealObject();
        } catch (Exception e) {
            LOG.error("Error setting up Javascript environment", e);
        }
        finally {
            Context.exit();
        }
        return global;
    }
    
    public static void loadJS(Context cx, Scriptable scr, String fileName) throws Exception {
        Reader reader;
        try {
            reader = new InputStreamReader(new FileInputStream(new File(fileName)), Charsets.UTF_8);
        } catch (FileNotFoundException e) {
            throw new Exception("Could not find file", e);
        };
        
        try {
            cx.evaluateReader(scr, reader, fileName, 0, null);
        } catch (IOException e) {
            throw new Exception("IO error reading file", e);
        }
        finally {
            try { reader.close(); } catch (IOException e) {
                throw new Exception("IO error closing file", e);
            }
        }
    }
    
    protected static Scriptable getScope(Context cx) {
        Scriptable scope = cx.newObject(RunJS.ENVIRONMENT);
        scope.setPrototype(RunJS.ENVIRONMENT);
        scope.setParentScope(null);
        return scope;
    }
}
```

boostrap.js

```javascript
require.config({
    baseUrl: '/base/path/to/packages'
});

var window, document, d3;

require(['domino/index'], function(domino) {
    window = domino.createWindow('');
    document = window.document;

    require(['d3/d3'], function(_d3) {
        d3 = _d3;
    });
});

// preload your modules
require(["mypackage/mymodule1", "mypackage/mymodule2"]);
```

Subclass RunJS and load your JavaScript file like this:

```java
Context cx = Context.enter();
try {
    cx.setOptimizationLevel(OPTIMIZATION_LEVEL);
    cx.setLanguageVersion(JS_VERSION);
    
    Scriptable scope = getScope(cx);
    scope.put("myvar", scope, myvar);
    loadJS(cx, scope, "/path/to/yourcode.js");
}
catch (Exception e) {
    LOG.error(e);
}
finally {
    Context.exit();
}
```
