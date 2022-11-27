# Minimal fullstack project

Shadow cljs can manage clj dependencies for the fullstack project, but deps.edn is recommended.

But if we only want to do frontend, then we could use this min template, manages cljs dependencies
with shadow-cljs and if we need node.js dependencies then create a package.json.

Integrate with code editors:
<https://shadow-cljs.github.io/docs/UsersGuide.html#_editor_integration>

## Using with calva

Calva has built-in support for shadow-cljs, so just use the Jack-In command, follow the prompts, and you're all set up.

## Using with cursive

We could start the shadow-cljs server, the nREPL server, and watch the build (and the shadow-cljs.edn config has hot reload).

`npx shadow-cljs watch :app`

or install it globally `npm i -g shadow-cljs`.

And open the browser at http://localhost:4000, and you should see in the js console "init client".

### Connect to the clj nrepl

Create a remote nREPL and connect to it, and type:

`(tap> "hello from idea clj remote repl")`

Go to the shadow-cljs server and inspect the JVM runtime in the "inspect stream" tab,
you should see the tap that we sent from the idea remote repl.

Note that the Cursive REPL when first connected always starts out as a CLJ REPL. You can switch it to CLJS by calling 
`(shadow/repl :your-build-id)`. This will automatically switch the Cursive option as well. You can type `:cljs/quit` 
to drop back down to the CLJ REPL.

NOTE:
You cannot switch from CLJ→CLJS via the Cursive select box. Make sure you use the call above to switch.

### Cursive doesn't resolve dependencies via shadow-cljs.edn

Alternatively you can create a dummy deps.edn/project.clj or use the full Deps/Leiningen integration.
<https://shadow-cljs.github.io/docs/UsersGuide.html#_build_tool_integration>

```clojure
{:deps {thheller/shadow-cljs {:mvn/version "2.20.12"}}}
```

```clojure
(defproject your/project "0.0.0"
  :dependencies
  [[thheller/shadow-cljs "X.Y.Z"]]

  :source-paths
  ["src"])
```
