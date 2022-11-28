# Minimal fullstack project
<https://www.youtube.com/watch?v=islMjv55cN8>

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

## new commit!
At this point we are using deps.edn to manage the dependencies, both frontend and backend, 
thus shadow-cljs will start via clojure command.

And to get rid of the warning:

```
------------------------------------------------------------------------------

   WARNING: shadow-cljs not installed in project.
   See https://shadow-cljs.github.io/docs/UsersGuide.html#project-install

------------------------------------------------------------------------------
```

We need to create the package.json, and install the dependency:

``bash
npm init -y
npm i -D shadow-cljs
``

Now for development you could create two remote nREPL, both connecting to the shadow-cljs nREPL started.
You promote one to cljs and select the :app, `(shadow/repl :app)`, and the other just connect and use it
for backend development.

Note: Although you could start a local repl within cursive with right click on the deps.edn file and
"Run REPL for...", this would be a waste of resources (but we could mess with the jvm runtime used by shadow-cljs), 
cuz we can use the JVM runtime of shadow-cljs that is used also for compiling the cljs files to js, and other stuff.

This is a limitation in Calva because you cannot open two nREPL connections in the same project window, you need to use
workspaces. We have two options to have both front and backend nREPL sessions in one window:

1. Start the clojure repl using the deps.edn tooling, the clojure program, then that one in turn can start shadow-cljs.
   The Jack-In option "deps.edn + shadow-cljs" which automate the steps in <https://github.com/pez/shadow-w-backend>.
   You will see that the command is clojure and inject dependencies and middleware, one of the is the one that intercepts
   cljs code and sent it to the shadow-cljs runtime.
   We could copy the Jack-In command for deps.edn + shadow-cljs and run it manually so the process (the clojure process
   that we started at the terminal, the jack-in connection it is killed when the window is reloaded) won't get killed
   if we reload the VS Code window (when usin jack-in command and prompt), and the use "Calva: Connect to a Running Server 
   in the Project".
2. Start everything with the shadow-cljs executable, the npm module, and it can in turn choose to start the clojure 
   programing using clojure, instead of starting itself it will say tell clojure to start, and then will also tell
   clojure to handle the dependencies and the "sort pass???" we (shadow-cljs) don't have nothing to do with it.
   Having now deps.edn managing the dependencies `:deps true` in `shadow-cljs.edn`.

In both options Calva will use two sessions, one when you evaluate code from clj files and the cljs session for
executing code from the cljs files.

Difference in Calva of the two options to start the full-stack project.

- First option, when you use shadow-cljs and the in turn shadow-cljs uses clojure to
  start it. Then shadow-cljs output will be in the "jack-in terminal" (the terminal window where we run the
 `npx shadow-cljs watch :app` for cursive), great when you have a shadow-cljs test runner you will have the results there.

- Second, "deps.edn + shadow-cljs", then all this output will go to the
  "Calva output window" (the window where the code evaluation are printed) 
  and mix up with your results.

**So the first is prefered.**

For Cursive if we decide to use clojure command in the root dir project:

```bash
clojure -Sdeps '{:deps {nrepl/nrepl {:mvn/version,"1.0.0"}}}' -M -m nrepl.cmdline --middleware "[shadow.cljs.devtools.server.nrepl/middleware]"
```

We could use cider-nrepl as a dependecy but Cursive do not use it.

```bash
clojure -Sdeps '{:deps {nrepl/nrepl {:mvn/version,"1.0.0"},cider/cider-nrepl {:mvn/version,"0.28.5"}}}'  -m nrepl.cmdline --middleware "[cider.nrepl/cider-middleware,shadow.cljs.devtools.server.nrepl/middleware]"
```

We needed to inject nrepl dependency cuz we don't have it in our deps.edn, and shadow-cljs we do have it because is
needed for development. The deps passed in the terminal get merged with the one in the deps.edn file.

We connect with nREPL remote with the .nrepl-port file created by the above command.

and then we need to follow <https://github.com/pez/shadow-w-backend>, in order to start:
1. start shadow-cljs server `(do (require 'shadow.cljs.devtools.server) (shadow.cljs.devtools.server/start!))`
2. start watcher on the build `(do (require 'shadow.cljs.devtools.api) (shadow.cljs.devtools.api/watch :app))`
3. Start the clojurescript app, open <http://localhost:8700> and confirm thw shadow-cljs watcher is running.
4. attach the ClojureScript REPL on the build `(do (require 'shadow.cljs.devtools.api) (shadow.cljs.devtools.api/nrepl-select :app))`
5. you can evaluate something like `(js/alert "hey!")`

This is where the video ends.

## new changes
We added shadow-cljs.edn with a deps alias, in this way when we start shadow-cljs with
the npm bin it will fire clojure command and use that alias with the extra dependency.

But if we start a repl without the cljs alias, it will be for our backend and it will not
include that shadow-cljs dependency not needed for backed development.
