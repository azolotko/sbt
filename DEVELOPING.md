Developer guide
===============

### Branch to work against

sbt uses two branches for development:

- Development branch: `develop` (this is also called "master")
- Stable branch: `1.$MINOR.x`, where `$MINOR` is current minor version (e.g. `1.1.x` during 1.1.x series)

### Instruction to build all modules from source

1. Install the current stable binary release of sbt (see [Setup]), which will be used to build sbt from source.
2. Get the source code.

   ```
   $ mkdir sbt-modules
   $ cd sbt-modules
   $ for i in sbt io util librarymanagement zinc; do \
     git clone https://github.com/sbt/$i.git && (cd $i; git checkout -b develop origin/develop)
   done
   $ cd sbt
   $ ./sbt-allsources.sh
   ```

3. To build and publish all components locally,

   ```
   $ ./sbt-allsources.sh
   sbt:sbtRoot> publishLocalAllModule
   ```

### Instruction to build just sbt

If the change you are making is contained in sbt/sbt, you could publishLocal on sbt/sbt:

```
$ sbt
sbt:sbtRoot> publishLocal
```

### Using the locally built sbt

The `publishLocal` above will build and publish version `1.$MINOR.$PATCH-SNAPSHOT` (e.g. 1.1.2-SNAPSHOT) to your local ivy repository.

To use the locally built sbt, set the version in `build.properties` file in your project to `1.$MINOR.$PATCH-SNAPSHOT` then launch `sbt` (this can be the `sbt` launcher installed in your machine).

```
$ cd $YOUR_OWN_PROJECT
$ sbt
> compile
```

### Using Jenkins sbt-snapshots nighties

There is a Jenkins instance for sbt that every night builds and publishes (if successful) a timestamped version
of sbt to http://jenkins.scala-sbt.org/sbt-snapshots and is available for 4-5 weeks. To use it do the following:

1. Set the `sbt.version` in `project/build.properties`

```bash
echo "sbt.version=1.2.0-bin-20180423T192044" > project/build.properties
```

2. Create an sbt repositories file (`./repositories`) that includes that Maven repository:

```properties
[repositories]
  local
  local-preloaded-ivy: file:///${sbt.preloaded-${sbt.global.base-${user.home}/.sbt}/preloaded/}, [organization]/[module]/[revision]/[type]s/[artifact](-[classifier]).[ext]
  local-preloaded: file:///${sbt.preloaded-${sbt.global.base-${user.home}/.sbt}/preloaded/}
  maven-central
  sbt-maven-releases: https://repo.scala-sbt.org/scalasbt/maven-releases/, bootOnly
  sbt-maven-snapshots: https://repo.scala-sbt.org/scalasbt/maven-snapshots/, bootOnly
  typesafe-ivy-releases: https://repo.typesafe.com/typesafe/ivy-releases/, [organization]/[module]/[revision]/[type]s/[artifact](-[classifier]).[ext], bootOnly
  sbt-ivy-snapshots: https://repo.scala-sbt.org/scalasbt/ivy-snapshots/, [organization]/[module]/[revision]/[type]s/[artifact](-[classifier]).[ext], bootOnly
  sbt-snapshots: https://jenkins.scala-sbt.org/sbt-snapshots
```

3. Start sbt with a stable launcher and the custom repositories file:

```bash
$ sbt -sbt-jar ~/.sbt/launchers/1.1.4/sbt-launch.jar -Dsbt.repository.config=repositories
Getting org.scala-sbt sbt 1.2.0-bin-20180423T192044  (this may take some time)...
downloading https://jenkins.scala-sbt.org/sbt-snapshots/org/scala-sbt/sbt/1.2.0-bin-20180423T192044/sbt-1.2.0-bin-20180423T192044.jar ...
	[SUCCESSFUL ] org.scala-sbt#sbt;1.2.0-bin-20180423T192044!sbt.jar (139ms)
...
[info] sbt server started at local:///Users/dnw/.sbt/1.0/server/936e0f52ed9baf6b6d83/sock
> show sbtVersion
[info] 1.2.0-bin-20180423T192044
```

### Using Jenkins maven-snapshots nightlies

As an alternative you can request a build that publishes to https://repo.scala-sbt.org/scalasbt/maven-snapshots
and stays there forever by:

1. Logging into https://jenkins.scala-sbt.org/job/sbt-validator/
2. Clicking "Build with Parameters"
3. Making sure `deploy_to_bintray` is enabled
4. Hitting "Build"

Afterwhich start sbt with a stable launcher: `sbt -sbt-jar ~/.sbt/launchers/1.1.4/sbt-launch.jar`

### Clearing out boot and local cache

When you run a locally built sbt, the JAR artifacts will be now cached under `$HOME/.sbt/boot/scala-2.12.6/org.scala-sbt/sbt/1.$MINOR.$PATCH-SNAPSHOT` directory. To clear this out run: `reboot dev` command from sbt's session of your test application.

One drawback of `-SNAPSHOT` version is that it's slow to resolve as it tries to hit all the resolvers. You can workaround that by using a version name like `1.$MINOR.$PATCH-LOCAL1`. A non-SNAPSHOT artifacts will now be cached under `$HOME/.ivy/cache/` directory, so you need to clear that out using [sbt-dirty-money](https://github.com/sbt/sbt-dirty-money)'s `cleanCache` task.

### Running sbt "from source" - `sbtOn`

In addition to locally publishing a build of sbt, there is an alternative, experimental launcher within sbt/sbt
to be able to run sbt "from source", that is to compile sbt and run it from its resulting classfiles rather than
from published jar files.

Such a launcher is available within sbt/sbt's build through a custom `sbtOn` command that takes as its first
argument the directory on which you want to run sbt, and the remaining arguments are passed _to_ that sbt
instance. For example:

I have setup a minimal sbt build in the directory `/s/t`, to run sbt on that directory I call:

```bash
> sbtOn /s/t
[info] Packaging /d/sbt/scripted/sbt/target/scala-2.12/scripted-sbt_2.12-1.2.0-SNAPSHOT.jar ...
[info] Done packaging.
[info] Running (fork) sbt.RunFromSourceMain /s/t
Listening for transport dt_socket at address: 5005
[info] Loading settings from idea.sbt,global-plugins.sbt ...
[info] Loading global plugins from /Users/dnw/.dotfiles/.sbt/1.0/plugins
[info] Loading project definition from /s/t/project
[info] Set current project to t (in build file:/s/t/)
[info] sbt server started at local:///Users/dnw/.sbt/1.0/server/ce9baa494c7598e4d59b/sock
> show baseDirectory
[info] /s/t
> exit
[info] shutting down sbt server
[success] Total time: 19 s, completed 25-Apr-2018 15:04:58
```

Please note that this alternative launcher does _not_ have feature parity with sbt/launcher. (Meta)
contributions welcome! :-D

### Diagnosing build failures

Globally included plugins can interfere building `sbt`; if you are getting errors building sbt, try disabling all globally included plugins and try again.

### Running Tests

sbt has a suite of unit tests and integration tests, also known as scripted tests.

#### Unit / Functional tests

Various functional and unit tests are defined throughout the
project. To run all of them, run `sbt test`. You can run a single test
suite with `sbt testOnly`

#### Integration tests

Scripted integration tests reside in `sbt/src/sbt-test` and are
written using the same testing infrastructure sbt plugin authors can
use to test their own plugins with sbt. You can read more about this
style of tests [here](http://www.scala-sbt.org/1.0/docs/Testing-sbt-plugins).

You can run the integration tests with the `sbt scripted` sbt
command. To run a single test, such as the test in
`sbt/src/sbt-test/project/global-plugin`, simply run:

    sbt "scripted project/global-plugin"

### Random tidbits

#### Import statements

You'd need alternative DSL import since you can't rely on sbt package object.

```scala
// for slash syntax
import sbt.SlashSyntax0._

// for IO
import sbt.io.syntax._
```
