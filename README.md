# Workflows to Build and Deploy a Lektor Website

Here you'll find example code on one way to build and deploy a [Lektor]
website using a [GitHub workflow].

Included here are a few [reusable workflows] that show how to build and
deploy (using rsync) a Lektor website.

If you'd like, you may call these reusable workflows from your own
workflows.  (The code in [ci.yml] shows how to do that.)

## Reusable Workflows

### Build and Deploy a Lektor Website

See [build-and-deploy.yml].

This is a reusable workflow to build a website using Lektor, and then
(optionally) deploy the site to a remote server using rsync.

#### Building the Lektor Website

Essentially, what is does to build the HTML is:


```bash
pip install -r requirements.txt
lektor build -O /tmp/htdocs
```

As such, it expects a `requirements.txt` file to be in the top-level
of your repository.  By default, your `.lektorproject` file is also
expected to be in the top-level directory, but you can set the
`lektor-project` input to the path to either the `.lektorproject` file
to use, or to its directory.

This workflow also manages two workflow caches to help reduce
(re)build times.

The contents of Lektor’s output directory is saved between runs in one
cache. This can reduce build time, since it let’s Lektor’s dependency
resolution system work out which output files need to be rebuilt.

Pip’s wheel cache is also cached between runs.  This cache is keyed on
the `requirements.txt` file. The wheel cache is re-used so long as
`requirements.txt` remains unchanged.

> **Warning** Since the wheel cache is reused (and not updated) so
> long as `requirements.txt` remains unchanged, for best results your
> `requirements.txt` should be quite explicit, with exact pins for
> every package installed.  ([Pip-compile] can be used to generate a
> suitable `requirements.txt` from a minimal
> `requirements.in`. [Pipenv] is another viable way to manage a pinned
> requirements.txt.)

#### Deployment

To deploy the built HTML, the workflow does, approximately:

```bash
rsync --recursive --delete-delay --exclude=.lektor /tmp/htdocs/ $rsync-dest
```

The `$rsync-dest` is configured via the `rsync-dest` input to the
workflow. If that is not set, the `RSYNC_DEST` workflow variable is used.
An SSH private key should be passed the the `ssh-key` secret
of the workflow.
(If either `rsync-dest` or `ssh-key` is missing, no attempt to deploy
the built HTML Will be made.)

> **Note** This workflow currently disables SSH’s
> `StrictHostKeyChecking` option.  (Since we’re just sending our HTML
> out, what's the worst that can happen if we are mistakenly talking
> to a man-in-the-middle?)

#### Example Usage


```yaml
name: Build and publish Lektor Website

on: push

jobs:
  build:
    uses: dairiki/lektor-ci/.github/workflows/build-and-deploy.yml@v1
    with:
      python-version: '3.10'
      rsync-dest: 'myuser@myserver.example.org:/path/to/htdocs'
    secrets:
      ssh-key: ${{ secrets.STAGING_SSH_KEY }}
```

For full details, see [the source][build-and-deploy.html].


#### Using Environments

This workflow can pull variables and secrets from an environment.

```yaml
name: Build and publish Lektor Website

on: push

jobs:
  build:
    uses: dairiki/lektor-ci/.github/workflows/build-and-deploy.yml@v1
    with:
      python-version: '3.10'
      environment: staging
      # rsync-dest defaults to the value of the RSYNC_DEST
      # variable set for the environment (or repo or org).
    secrets:
      # Get ssh-key from environment (or repo or org) secret
      ssh-key: ${{ secrets.RSYNC_SSH_KEY }}
```

Note that the handling of secrets in reusable workflows when using
environments is a bit counter-intuitive. See [this blog
post](https://colinsalmcorner.com/consuming-environment-secrets-in-reusable-workflows/)
for some insight.

### Separate *Build* and *Deploy* Workflows

See [build.yml], and [deploy.html] for reusable workflows that
separate the building from the deployment.

The `build.yml` workflow save the built HTML in a workflow artifact.
The `deploy.yml` workflow retrieves that artifact and rsync’s to the
target server.

Initially, I used these to do the building and deploying in two
separate workflow jobs. However, I found that the creation and
retrieval of the artifact — necessary to pass the built HTML between
the jobs — takes a considerable amount of time. Doing [the build and
deploy](#build-and-deploy-a-lektor-website) in a single job is
considerably more efficient.


## Author

Jeff Dairiki <dairiki@dairiki.org>


[Lektor]: https://www.getlektor.com/

[GitHub workflow]: https://docs.github.com/en/actions/using-workflows/about-workflows
[reusable workflows]: https://docs.github.com/en/actions/using-workflows/reusing-workflows#calling-a-reusable-workflow
[artifact]: https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts
[secret]: https://docs.github.com/en/actions/security-guides/encrypted-secrets

[pip-compile]: https://github.com/jazzband/pip-tools
[pipenv]: https://pipenv-fork.readthedocs.io/en/latest/advanced.html#generating-a-requirements-txt

[ci.yml]: https://github.com/dairiki/lektor-ci/blob/master/.github/workflows/ci.yml
[build-and-deploy.yml]: https://github.com/dairiki/lektor-ci/blob/master/.github/workflows/build-and-deploy.yml
[build.yml]: https://github.com/dairiki/lektor-ci/blob/master/.github/workflows/build.yml
[deploy.yml]: https://github.com/dairiki/lektor-ci/blob/master/.github/workflows/deploy.yml
