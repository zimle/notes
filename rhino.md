# Rhino

Rhino is an implementation of javascript in java and allows switching between java and javascript and sharing objects

## rhino from java

1. define a context
2. within this context, define a scope (interface javscript must implement, java classes accessible from javascript)
3. run context.evaluateString:

  ```java
  * Evaluate a JavaScript source string.
     *
     * The provided source name and line number are used for error messages
     * and for producing debug information.
     *
     * @param scope the scope to execute in
     * @param source the JavaScript source
     * @param sourceName a string describing the source, such as a filename
     * @param lineno the starting line number
     * @param securityDomain an arbitrary object that specifies security
     *        information about the origin or owner of the script. For
     *        implementations that don't care about security, this value
     *        may be null.
     * @return the result of evaluating the string
     * @see org.mozilla.javascript.SecurityController
     */
    public final Object evaluateString(Scriptable scope, String source,
                                       String sourceName, int lineno,
                                       Object securityDomain)
  ```

## ECMA standards supported

- the `ECMA` standard supported can be viewed in <https://mozilla.github.io/rhino/compat/engines.html>.
- the _partial_ (?) support for es6 can be enabled via  `Context.VERSION_ES6` or the command line flag -version 200
- `ECMA` features added with version can be found on [medium](https://madasamy.medium.com/javascript-brief-history-and-ecmascript-es6-es7-es8-features-673973394df4) and [wiki](https://en.wikipedia.org/wiki/ECMAScript)
