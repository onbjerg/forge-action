## `foundry-toolchain` Action

This GitHub Action installs [Foundry](https://github.com/foundry-rs/foundry), the blazing fast, portable and modular
toolkit for Ethereum application development.

### Example workflow

```yml
on: [push]

name: test

jobs:
  check:
    name: Foundry project
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: recursive

      - name: Install Foundry
        uses: foundry-rs/foundry-toolchain@v1

      - name: Run tests
        run: forge test -vvv

      - name: Run snapshot
        run: forge snapshot
```

### Inputs

| **Name**             | **Required** | **Default**                           | **Description**                                                                                              | **Type** |
| -------------------- | ------------ | ------------------------------------- | ------------------------------------------------------------------------------------------------------------ | -------- |
| `cache`              | No           | `true`                                | Whether to cache RPC responses or not.                                                                       | bool     |
| `version`            | No           | `nightly`                             | Version to install, e.g. `nightly` or `1.0.0`. **Note:** Foundry only has nightly builds for the time being. | string   |
| `cache-key`          | No           | `${{ github.job }}-${{ github.sha }}` | The cache key to use for caching.                                                                            | string   |
| `cache-restore-keys` | No           | `${{ github.job }}-`                  | The cache key to use for restoring the cache.                                                                | string[] |

### RPC Caching

By default, this action matches Forge's behavior and caches all RPC responses in the `~/.foundry/cache/rpc` directory.
This is done to speed up the tests and avoid hitting the rate limit of your RPC provider.

The logic of the caching is as follows:

- Always load the latest valid cache, and always create a new one with the updated cache.
- When there are no changes to the fork tests, the cache does not change but the key does, since the key is based on the
  commit hash.
- When the fork tests are changed, both the cache and the key are updated.

If you would like to disable the caching (e.g. because you want to implement your own caching mechanism), you can set
the `cache` input to `false`, like this:

```yml
- name: Install Foundry
  uses: foundry-rs/foundry-toolchain@v1
  with:
    cache: false
```

### Custom Cache Keys

You can also set custom cache keys using the `cache-key` and `cache-restore-keys` inputs. This is useful if you want to
customize how the cache may be shared between jobs. Note that cache-key must be unique for each run to successfully save
without conflicts.

For example, to share the cache across two different jobs, you could do something like this:

```yml
- name: Install Foundry
  uses: foundry-rs/foundry-toolchain@v1
  with:
    cache-key: custom-seed-test-${{ github.sha }}
    cache-restore-keys: |-
      custom-seed-test
      custom-seed-
---
- name: Install Foundry
  uses: foundry-rs/foundry-toolchain@v1
  with:
    cache-key: custom-seed-coverage-${{ github.sha }}
    cache-restore-keys: |-
      custom-seed-coverage
      custom-seed-
```

This will create two different caches, one for each job, but they will share the same cache key prefix. This means that
when restoring the cache in the second job, it will look for caches with keys that start with `custom-seed-`. This
allows you to share the cache across different jobs, or across different workflows if desired.

#### Deleting Caches

You can delete caches via the GitHub Actions user interface. Just go to your repo's "Actions" page:

```text
https://github.com/<OWNER>/<REPO>/actions/caches
```

Then, locate the "Management" section, and click on "Caches". You will see a list of all of your current caches, which
you can delete by clicking on the trash icon.

For more detail on how to delete caches, read GitHub's docs on
[managing caches](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#managing-caches).

#### Fuzzing

Note that if you are fuzzing in your fork tests, the RPC cache strategy above will not work unless you set a
[fuzz seed](https://book.getfoundry.sh/reference/config/testing#seed). You might also want to reduce your number of RPC
calls by using [Multicall](https://github.com/mds1/multicall).

### Summaries

You can add the output of Forge and Cast commands to GitHub step summaries. The summaries support GitHub flavored
Markdown.

For example, to add the output of `forge snapshot` to a summary, you would change the snapshot step to:

```yml
- name: Run snapshot
  run: NO_COLOR=1 forge snapshot >> $GITHUB_STEP_SUMMARY
```

See the official
[GitHub docs](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#adding-a-job-summary)
for more information.

### Building

When opening a PR, you must build the action exactly following the below steps for CI to pass:

```console
$ npm ci
$ npm run build
```

You **have** to use Node.js 20.x.
