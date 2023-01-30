# Workflows to Build and Deploy a Lektor Website

Here you'll find example code on one way to build and deploy a [Lektor]
website using a [GitHub workflow].

Included here are two [reusable workflows] that show how to build and
deploy (using rsync) a Lektor website.

If you'd like, you may call these reusable workflows from your own
workflows.  (The code in [ci.yml] shows how to do that.)

## Reusable Workflows

### Building the HTML

See [build.yml].

This is a reusable workflow to build a website using Lektor.

Essentially, what is does is:


```bash
pip install -r requirements.txt
lektor build -O htdocs
```

then it saves the results from `htdocs` in a workflow [artifact].

As such, it expects a `requirements.txt` and your `.lektorproject`
file to be in the top-level of your repository.

#### Caching

This workflow also manages two workflow caches.

The contents of Lektor’s output directory is saved between runs. This
can reduce build time, since it let’s Lektor’s dependency resolution
system work out which output files need to be rebuilt.

Pip’s wheel cache is also cached between runs.  This cache is keyed on
the `requirements.txt` file. The wheel cache is re-used (and not
updated) so long as `requirements.txt` remains unchanged.

> **Warning** Since the wheel cache is reused (and not updated) so long
> as `requirements.txt` remains unchanged, for best results your
> `requirements.txt` should be quite explicit, with exact pins for
> every package installed.
> ([Pip-compile] can be used to generate a suitable `requirements.txt`
> from a minimal `requirements.in`.)

- Lektor’s output directory (`htdocs`)

#### Examples

```yaml
jobs:
  build:
    uses: dairiki/lektor-ci/.github/workflows/build.yml@v1
    with:
      python-version: "3.10"
```

#### Usage

See [the source][build.yml] for details on all inputs that are
accepted.

### Deploying the HTML using Rsync

See [deploy.yml].

This is a reusable workflow to publish a static website using rsync over SSH.

Minimal example usage, assuming that there is a repository or
organization [secret] named `RSYNC_SSH_KEY` that contains the SSH
private key to be used to authenticate when rsyncing to the server.


```yaml
on: push

jobs:

  build:
    # job to build the website, saving the resulting HTML
    # in an workflow artifact named "htdocs"
    # ...

  deploy:
    needs: build
    uses: dairiki/lektor-ci/.github/workflows/deploy.yml@v1
    with:
      rsync-dest: myuser@server.example.org:/srv/www/mysite/htdocs
    secrets:
      ssh-key: ${{ secrets.RSYNC_SSH_KEY }}
```

#### Using Environments

This workflow can pull variables and secrets from an environment.

```yaml
jobs:
  build:
    # ...

  deploy:
    needs: build
    uses: dairiki/lektor-ci/.github/workflows/deploy.yml@v1
    with:
      environment: staging
    secrets:
      # Get ssh-key from environment (or repo or org) secret
      ssh-key: ${{ secrets.RSYNC_SSH_KEY }}
```

Note that the handling of secrets in reusable workflows when using
environments is a bit counter-intuitive. See [this blog
post](https://colinsalmcorner.com/consuming-environment-secrets-in-reusable-workflows/)
for some insight.

#### Usage

See [the source][deploy.yml] for details on all inputs that are
accepted.



## Author

Jeff Dairiki <dairiki@dairiki.org>


[Lektor]: https://www.getlektor.com/

[GitHub workflow]: https://docs.github.com/en/actions/using-workflows/about-workflows
[reusable workflows]: https://docs.github.com/en/actions/using-workflows/reusing-workflows#calling-a-reusable-workflow
[artifact]: https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts
[secret]: https://docs.github.com/en/actions/security-guides/encrypted-secrets

[pip-compile]: https://github.com/jazzband/pip-tools

[ci.yml]: https://github.com/dairiki/lektor-ci/blob/master/.github/workflows/ci.yml
[build.yml]: https://github.com/dairiki/lektor-ci/blob/master/.github/workflows/build.yml
[deploy.yml]: https://github.com/dairiki/lektor-ci/blob/master/.github/workflows/deploy.yml
