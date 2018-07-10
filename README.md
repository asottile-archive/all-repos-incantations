all-repos-incantations
======================

Documented commands run with [`all-repos`][all-repos]
for doing mass rewrites.

[all-repos]: https://github.com/asottile/all-repos

### appveyor python3.6 -> python3.7 (2018-07-10)

```bash
all-repos-sed 's/36/37/g' appveyor.yml \
    --commit-msg 'Use python3.7 in appveyor'
```

This applies the following diff

```diff
diff --git a/appveyor.yml b/appveyor.yml
index 8df64a5..395320b 100644
--- a/appveyor.yml
+++ b/appveyor.yml
@@ -1,10 +1,10 @@
 environment:
     matrix:
         - TOXENV: py27
-        - TOXENV: py36
+        - TOXENV: py37

 install:
-    - "SET PATH=C:\\Python36;C:\\Python36\\Scripts;%PATH%"
+    - "SET PATH=C:\\Python37;C:\\Python37\\Scripts;%PATH%"
     - pip install tox

 # Not a C# project
```

[appveyor/ci#2475](https://github.com/appveyor/ci/issues/2475)

### `sha:` to `rev:` in pre-commit mirrors (2018-06-21)

```bash
all-repos-sed 's|sha:|rev:|g;s|sha you want|sha / tag you want|g' README.md \
    --commit-msg 'Replace sha: with rev: in documentation'
```

This applies the following diff

```diff
--- a/README.md
+++ b/README.md
@@ -12,6 +12,6 @@ For scss-lint: see https://github.com/causes/scss-lint
 Add this to your `.pre-commit-config.yaml`:

     -   repo: https://github.com/pre-commit/mirrors-scss-lint
-        sha: ''  # Use the sha you want to point at
+        rev: ''  # Use the sha / tag you want to point at
         hooks:
         -   id: scss-lint
```

[`sha` is soft deprecated for `rev`][pre-commit-repo-yaml]

[pre-commit-repo-yaml]: https://pre-commit.com/#pre-commit-configyaml---repos

### remove mentions of autopep8 E309 code (2018-06-01)

```bash
all-repos-sed 's/E309,//g' tox.ini \
    --commit-msg 'E309 is no longer rewritten by autopep8'
```

This applies the following diff

```diff
--- a/tox.ini
+++ b/tox.ini
@@ -21,4 +21,4 @@ commands =
 max-line-length=131

 [pep8]
-ignore=E265,E309,E501
+ignore=E265,E501
```

E309 was removed [here](https://github.com/hhatto/autopep8/pull/294).

### remove redundant `--show-missing` in `tox.ini` (2018-05-12)

```bash
all-repos-grep --repos -- show-missing -- tox.ini |
    xargs --replace grep -l show_missing {}/.coveragerc |
    cut -d'/' -f1-3 |
    xargs all-repos-sed 's/ --show-missing//g' tox.ini \
        --commit-msg 'Remove --show-missing when it is set in .coveragerc' \
        --branch-name show-missing \
        --repos
```

When the `[report]` section of `.coveragerc` already contains
`show_missing = True`, it is redundant to call
`coverage report --show-missing`.


Here is a [sample pr][sample-pr-show-missing].

[sample-pr-show-missing]: https://github.com/asottile/add-trailing-comma/pull/46

### replace legacy `wheel` metadata with `bdist_wheel` (2018-04-30)

```bash
all-repos-sed \
    's/\[wheel\]/[bdist_wheel]/g' setup.cfg \
    --commit-msg 'Replace legacy wheel metadata'
```

This applies the following diff:

```diff
--- a/setup.cfg
+++ b/setup.cfg
@@ -1,2 +1,2 @@
-[wheel]
+[bdist_wheel]
 universal = True
```

The `[wheel]` format is considered [legacy][legacy-wheel].

Here is a [sample pr][sample-pr-wheel].


[legacy-wheel]: https://bitbucket.org/pypa/wheel/src/54ddbcc9cec25e1f4d111a142b8bfaa163130a61/wheel/bdist_wheel.py?fileviewer=file-view-default#bdist_wheel.py-119:125
[sample-pr-wheel]: https://github.com/asottile/pyupgrade/pull/30

### find all flask routes (2018-04-22)

```bash
all-repos-grep '@.*\.route('
```

I was looking for `@app.route('/<path:path_var>')`

### normalize pre-commit config git urls (2018-03-07)

```bash
all-repos-sed \
    's|\.git$||g;s|git://|https://|g;s|git@github.com:|https://github.com/|g' .pre-commit-config.yaml \
    --interactive \
    --commit-msg "$(echo -en 'Normalize git urls in pre-commit config\n\nImproves cache performance')" \
    --repos $(all-repos-grep --repos -E '(git@|git://|\.git)' -- .pre-commit-config.yaml)
```

This normalizes git urls in `.pre-commit-config.yaml` to `https://` urls.

Normalizations:
- Remove unnecessary trailing `.git`
- `git://` to `https://`
- `git@github.com:` to `https://`

Normalization away from ssh urls is done for a few reasons:

- `ssh` credentials are not available on travis-ci.
- `SSH_AUTH_SOCK` is not passed through when running `tox`.

Here is a [sample pr](https://github.com/pre-commit/pre-commit/pull/721).
