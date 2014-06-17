# copy-deps-scripts

Until Eastwood version 0.1.2, Eastwood depended directly upon several
other projects, which in several cases had their own dependencies.

For example, tools.analyzer.jvm version 0.1.0-beta13 depended upon
these things (this is an excerpt of the output of the commend 'lein
deps :tree' for a project that depends directly upon
tools.analyzer.jvm):

 [org.clojure/tools.analyzer.jvm "0.1.0-beta13"]
   [org.clojure/core.memoize "0.5.6"]
     [org.clojure/core.cache "0.6.3"]
       [org.clojure/data.priority-map "0.0.2"]
   [org.ow2.asm/asm-all "4.1"]

Whenever Eastwood has such dependencies, it can cause errors if a
project being linted depends upon different versions of those
dependencies, especially if vars are added to or removed from the API.

Goal: Make Eastwood properly lint such projects, regardless of which
version of Clojure contrib libraries they depend upon.

One method to achieve this (somewhat tedious, but correct): Copy the
source code of Eastwood's dependencies into Eastwood itself, and
change the namespace names.

The bash script clone.sh in this directory does 'git clone' for
particular versions of what once were Eastwood dependencies.

I hope to automate the process of then copying these into the
appropriate subdirectory beneath eastwood/src/eastwood/copieddeps, and
then changing all of their namespace names to make them have the
prefix 'eastwood.copieddeps.dep<n>.' for the value of <n> I have
chosen to make their 'root namespaces' unique.

Until then, I have done this copying and editing manually.  Such
manual work is best (IMO) for figuring exactly what needs to be
automated, anyway.

These copied-and-renamed versions will not conflict with any version
chosen by a project being linted.


## Checking for updated Eastwood dependencies in the future

Subdirectory deps contains a Leiningen project.clj file.  That project
is not intended to be used or built in any way, except to see what
Eastwood's dependencies would be if it depended upon them in the
normal way, rather than copying their source code into its own tree.

It is intended merely for running commands like the following, to see
what the transitive dependencies are, and whether newer versions are
available:

    lein deps :tree

    lein ancient


### Current dependencies of project deps

As of June 16 2014:

    % lein deps :tree
     [clojure-complete "0.2.3" :exclusions [[org.clojure/clojure]]]
     [leinjacker "0.4.1"]
       [org.clojure/core.contracts "0.0.1"]
         [org.clojure/core.unify "0.5.3"]
     [org.clojars.brenton/google-diff-match-patch "0.1"]
     [org.clojure/clojure "1.6.0"]
     [org.clojure/tools.analyzer.jvm "0.2.2"]
       [org.clojure/core.memoize "0.5.6"]
         [org.clojure/core.cache "0.6.3"]
           [org.clojure/data.priority-map "0.0.2"]
       [org.ow2.asm/asm-all "4.1"]
     [org.clojure/tools.analyzer "0.2.2"]
     [org.clojure/tools.macro "0.1.2"]
     [org.clojure/tools.namespace "0.2.4"]
     [org.clojure/tools.nrepl "0.2.3" :exclusions [[org.clojure/clojure]]]
     [org.clojure/tools.reader "0.8.4"]

    % lein ancient
    [leinjacker "0.4.2"] is available but we use "0.4.1"
    [org.clojure/tools.macro "0.1.5"] is available but we use "0.1.2"

Latest versions of some transitive dependencies as of May 29 2014:

core.contracts 0.0.5 > 0.0.1 above, but leave at 0.0.1 since
  leinjacker has been tested with it
core.unify 0.5.6 > 0.5.3 above, but leave at 0.5.3 since leinjacker
  has been tested with it
core.memoize 0.5.6 = version above above, but see patch I applied below
core.cache 0.6.3 = version above
data.priority-map 0.0.5 > 0.0.2 above.  See below.
tools.namespace 0.2.4 = version above
tools.reader 0.8.4 = version above


### Current versions copied into Eastwood

data.priority-map 0.0.2 has a reflection warning that has been
eliminated as of version 0.0.5 (and perhaps also in an earlier
version).  The API should not have changed from 0.0.2 to 0.0.5, so
copy in 0.0.5.

core.memoize 0.5.6 has a reflection warning that can be eliminated, I
believe without introducing any bugs, by a simple patch attached to
JIRA ticket CMEMOIZE-13.

    http://dev.clojure.org/jira/browse/CMEMOIZE-13

I have copied in core.memoize 0.5.6, renamed the necessary namespaces,
and then applied that one-line patch by hand to eliminate the
reflection warning.  If a future version of core.memoize is later
copied into Eastwood, it would be good to verify that reflection
warning has been eliminated, or apply that one-line patch by hand
again.

tools.namespace 0.2.4 uses an exponential run time algorithm to find
transitive dependencies, where a linear time algorithm is possible.  I
have applied the patch named linear-time-no-debug.patch attached to
this ticket after copying it:

    http://dev.clojure.org/jira/browse/TNS-18

For all others, the versions output by 'lein deps :tree' are copied in
with no changes other than editing the namespace names.  See clone.sh
for details.