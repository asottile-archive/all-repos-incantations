all-repos-incantations
======================

Documented commands run with [`all-repos`][all-repos]
for doing mass rewrites.

[all-repos]: https://github.com/asottile/all-repos

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

Here is a [sample-pr][sample-pr-wheel].


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

Here is a [sample PR](https://github.com/pre-commit/pre-commit/pull/721).
