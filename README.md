all-repos-incantations
======================

Documented commands run with [`all-repos`][all-repos]
for doing mass rewrites.

[all-repos]: https://github.com/asottile/all-repos


### format

```markdown
### title (YYYY-mm-DD)

```bash
all-repos-...
```

Description of command.

Additional links and/or references.
```

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
