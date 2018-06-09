NB: the `???` are because these steps _used_ to cause the issue (maybe not 100% consistently, but with some level of opening/closing projects,
re-syncing, enabling/disabling the property, etc, at some point it created some kind of corrupted project definition). I only tested it on AS 3.2
canary 17 though, and I am not seeing it.  So it's possible that it has been fixed in a recent canary release.


---

In AS: File -> New -> Import Project, import lib-project
In AS: File -> New -> Import Project, import app-project
Build both projects in AS, everything works fine.

Note that the following files exist:

- app-project/app-project.iml
- app-project/app/app.iml
- lib-project/lib-project.iml  ??
- lib-project/lib/lib.iml      ??

Note: app-project is currently using the `repo/*` flat maven repo to get the lib-1.0.0.aar artifact. Assume that it is a normal maven artifact server.


Edit app-project/gradle.properties -> uncomment libPath=../lib-project

Gradle-Sync app-project.

See that :lib-project:lib now appears in the Android project explorer (as a "lib" module), and LibClass.java can be edited.
See that :app:assembleDebug works properly (and it uses the composite lib-project built library, matching the build type selected for :app).


See that the following attribute exists now in both lib*.iml files:

   external.root.project.path="$MODULE_DIR$/../../app-project"

???  Sync lib-project 

Edit app-project/gradle.properties -> re-comment libPath.

Gradle-sync app-project.

??? See that lib-project/lib-project.iml and  lib-project/lib/lib.iml have been deleted.


???  Sync lib-project

See that :lib is now removed from the app's project explorer.
See that LibClass.class is now accessed via the Gradle cached classes.jar file and cannot be edited.
See that :app:assembleDebug works properly still (and uses the lib-1.0.0.aar file from the `repo/` dir, or the Gradle cached version).

Now re-open the lib-project in AS.

??? some combination of syncing app-project, syncing lib-project, open/close projects, and/or enable/disable libPath property ???

eventually, what we used to see:

lib-project no longer shows the `:lib` module, but starts showing the `:app` module (!!), which should *never* be possible. The entire project
explorer becomes corrupted at this point, and you have no other option but to close the lib-project and re-import fresh.

Note that the app projects do not typically become corrupted (I think I saw this once in early 3.1 canaries, but have not seen it happen
recently). Only the library project becomes corrupt.
