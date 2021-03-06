# 32.1. Running the Tests

The regression tests can be run against an already installed and running server, or using a temporary installation within the build tree. Furthermore, there is a “parallel” and a “sequential” mode for running the tests. The sequential method runs each test script alone, while the parallel method starts up multiple server processes to run groups of tests in parallel. Parallel testing adds confidence that interprocess communication and locking are working correctly.

#### 32.1.1. Running the Tests Against a Temporary Installation

To run the parallel regression tests after building but before installation, type:

```text
make check
```

in the top-level directory. \(Or you can change to `src/test/regress` and run the command there.\) At the end you should see something like:

```text
=======================
 All 115 tests passed.
=======================
```

or otherwise a note about which tests failed. See [Section 32.2](https://www.postgresql.org/docs/10/static/regress-evaluation.html) below before assuming that a “failure” represents a serious problem.

Because this test method runs a temporary server, it will not work if you did the build as the root user, since the server will not start as root. Recommended procedure is not to do the build as root, or else to perform testing after completing the installation.

If you have configured PostgreSQL to install into a location where an older PostgreSQL installation already exists, and you perform `make check` before installing the new version, you might find that the tests fail because the new programs try to use the already-installed shared libraries. \(Typical symptoms are complaints about undefined symbols.\) If you wish to run the tests before overwriting the old installation, you'll need to build with `configure --disable-rpath`. It is not recommended that you use this option for the final installation, however.

The parallel regression test starts quite a few processes under your user ID. Presently, the maximum concurrency is twenty parallel test scripts, which means forty processes: there's a server process and a psql process for each test script. So if your system enforces a per-user limit on the number of processes, make sure this limit is at least fifty or so, else you might get random-seeming failures in the parallel test. If you are not in a position to raise the limit, you can cut down the degree of parallelism by setting the `MAX_CONNECTIONS` parameter. For example:

```text
make MAX_CONNECTIONS=10 check
```

runs no more than ten tests concurrently.

#### 32.1.2. Running the Tests Against an Existing Installation

To run the tests after installation \(see [Chapter 16](https://www.postgresql.org/docs/10/static/installation.html)\), initialize a data area and start the server as explained in [Chapter 18](https://www.postgresql.org/docs/10/static/runtime.html), then type:

```text
make installcheck
```

or for a parallel test:

```text
make installcheck-parallel
```

The tests will expect to contact the server at the local host and the default port number, unless directed otherwise by `PGHOST` and `PGPORT` environment variables. The tests will be run in a database named `regression`; any existing database by this name will be dropped.

The tests will also transiently create some cluster-wide objects, such as roles and tablespaces. These objects will have names beginning with `regress_`. Beware of using `installcheck`mode in installations that have any actual users or tablespaces named that way.

#### 32.1.3. Additional Test Suites

The `make check` and `make installcheck` commands run only the “core” regression tests, which test built-in functionality of the PostgreSQL server. The source distribution also contains additional test suites, most of them having to do with add-on functionality such as optional procedural languages.

To run all test suites applicable to the modules that have been selected to be built, including the core tests, type one of these commands at the top of the build tree:

```text
make check-world
make installcheck-world
```

These commands run the tests using temporary servers or an already-installed server, respectively, just as previously explained for `make check` and `make installcheck`. Other considerations are the same as previously explained for each method. Note that `make check-world` builds a separate temporary installation tree for each tested module, so it requires a great deal more time and disk space than `make installcheck-world`.

Alternatively, you can run individual test suites by typing `make check` or `make installcheck` in the appropriate subdirectory of the build tree. Keep in mind that `make installcheck`assumes you've installed the relevant module\(s\), not only the core server.

The additional tests that can be invoked this way include:

* Regression tests for optional procedural languages \(other than PL/pgSQL, which is tested by the core tests\). These are located under `src/pl`.
* Regression tests for `contrib` modules, located under `contrib`. Not all `contrib` modules have tests.
* Regression tests for the ECPG interface library, located in `src/interfaces/ecpg/test`.
* Tests stressing behavior of concurrent sessions, located in `src/test/isolation`.
* Tests of client programs under `src/bin`. See also [Section 32.4](https://www.postgresql.org/docs/10/static/regress-tap.html).

When using `installcheck` mode, these tests will destroy any existing databases named `pl_regression`, `contrib_regression`, `isolation_regression`, `ecpg1_regression`, or `ecpg2_regression`, as well as `regression`.

The TAP-based tests are run only when PostgreSQL was configured with the option `--enable-tap-tests`. This is recommended for development, but can be omitted if there is no suitable Perl installation.

#### 32.1.4. Locale and Encoding

By default, tests using a temporary installation use the locale defined in the current environment and the corresponding database encoding as determined by `initdb`. It can be useful to test different locales by setting the appropriate environment variables, for example:

```text
make check LANG=C
make check LC_COLLATE=en_US.utf8 LC_CTYPE=fr_CA.utf8
```

For implementation reasons, setting `LC_ALL` does not work for this purpose; all the other locale-related environment variables do work.

When testing against an existing installation, the locale is determined by the existing database cluster and cannot be set separately for the test run.

You can also choose the database encoding explicitly by setting the variable `ENCODING`, for example:

```text
make check LANG=C ENCODING=EUC_JP
```

Setting the database encoding this way typically only makes sense if the locale is C; otherwise the encoding is chosen automatically from the locale, and specifying an encoding that does not match the locale will result in an error.

The database encoding can be set for tests against either a temporary or an existing installation, though in the latter case it must be compatible with the installation's locale.

#### 32.1.5. Extra Tests

The core regression test suite contains a few test files that are not run by default, because they might be platform-dependent or take a very long time to run. You can run these or other extra test files by setting the variable `EXTRA_TESTS`. For example, to run the `numeric_big` test:

```text
make check EXTRA_TESTS=numeric_big
```

To run the collation tests:

```text
make check EXTRA_TESTS='collate.icu.utf8 collate.linux.utf8' LANG=en_US.utf8
```

The `collate.linux.utf8` test works only on Linux/glibc platforms. The `collate.icu.utf8` test only works when support for ICU was built. Both tests will only succeed when run in a database that uses UTF-8 encoding.

#### 32.1.6. Testing Hot Standby

The source distribution also contains regression tests for the static behavior of Hot Standby. These tests require a running primary server and a running standby server that is accepting new WAL changes from the primary \(using either file-based log shipping or streaming replication\). Those servers are not automatically created for you, nor is replication setup documented here. Please check the various sections of the documentation devoted to the required commands and related issues.

To run the Hot Standby tests, first create a database called `regression` on the primary:

```text
psql -h primary -c "CREATE DATABASE regression"
```

Next, run the preparatory script `src/test/regress/sql/hs_primary_setup.sql` on the primary in the regression database, for example:

```text
psql -h primary -f src/test/regress/sql/hs_primary_setup.sql regression
```

Allow these changes to propagate to the standby.

Now arrange for the default database connection to be to the standby server under test \(for example, by setting the `PGHOST` and `PGPORT` environment variables\). Finally, run `make standbycheck` in the regression directory:

```text
cd src/test/regress
make standbycheck
```

Some extreme behaviors can also be generated on the primary using the script `src/test/regress/sql/hs_primary_extremes.sql` to allow the behavior of the standby to be tested.

