# Reusable Workflows

Thanks to the new GitHub Actions feature called "Reusable Workflows" we can now reference an existing workflow with a single line of configuration rather than copying and pasting from one workflow to another.

This massively reduces the amount of boilerplate setup in each plugin to the bare minimum and also means that we can maintain and revise our definition of best practice in one place and have all the Moodle plugins inherit improvements in lock step. Mostly these shared actions in turn wrap the Moodle plugin CI scripts:

https://moodlehq.github.io/moodle-plugin-ci/

## :rocket: Quick start

For most plugins, you'll want to go through this checklist:

1. Ensure branch section in `README.md` describes correct versions supported
2. Supported range in `version.php` has been configured correctly
3. Workflow file has been added `.github/workflows/ci.yml` and configured
4. CI badge has been properly added in the `README.md`

Some examples of usage: [tool_mfa](https://github.com/catalyst/moodle-tool_mfa#branches), [tool_dataflows](https://github.com/catalyst/moodle-tool_dataflows/#branches)


### Configure support range

Each plugin should have a measurable range of versions supported. It's recommended and ensures a predictable test range.

1. Open `version.php` in your plugin repository
2. Set the `$plugin->supported`* as the range the plugin supports, which then determines the versions the workflow tests for.

```php
# version.php
$plugin->supported = [35, 402];
```
This example will run a matrix of tests from `MOODLE_35_STABLE` to `MOODLE_402_STABLE` - [see full test matrix here](.github/actions/matrix/matrix_includes.yml).

Any number _greater_ than the [latest available stable branch](https://github.com/moodle/moodle/branches/active) will automatically include the [main branch](https://github.com/moodle/moodle/tree/main) for testing

\* For more info on the `$plugin->supported` field, please see https://docs.moodle.org/dev/version.php


### Add the workflow

For most cases, this following demonstrates how your workflow file would typically look.

Create `.github/workflows/moodle-plugin-ci.yml` with the below template in your plugin
repository.
```yaml
# .github/workflows/moodle-plugin-ci.yml
name: ci

on: [push, pull_request]

jobs:
  ci:
    uses: Wunderbyte-GmbH/catalyst-moodle-workflows/.github/workflows/ci.yml@main
    # Required if you plan to publish (uncomment the below)
    # secrets:
      # moodle_org_token: ${{ secrets.MOODLE_ORG_TOKEN }}
    with:
      # Any further options in this section
```
For how to set up the secret, please see the [_How does this automate releases_](#how-does-this-automate-releases) section below.

You can add extra options to disable checks that you might not want, or to add additional dependencies under the `with` field. For example:
```yaml
    with:
      disable_behat: true
      disable_grunt: true
      extra_plugin_runners: 'moodle-plugin-ci add-plugin catalyst/moodle-local_aws'
```

#### `with` options

Below lists the available inputs which are _all optional_:

| Inputs                    | Description                                                                                                                                                                              |
|---------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| codechecker_max_warnings  | To fail on warnings, set this to 0                                                                                                                                                       |
| extra_plugin_runners      | Command to install more dependencies                                                                                                                                                     |
| disable_release           | Set `true` Use true if using the tag-based release workflow.                                                                                                                             |
| disable_behat             | Set `true` to disable behat tests.                                                                                                                                                       |
| disable_phpdoc            | Set `true` to disable phpdoc tests.                                                                                                                                                      |
| disable_phpcs             | Set `true` to disable phpcs (codechecker) tests.                                                                                                                                         |
| disable_phplint           | Set `true` to disable phplint tests.                                                                                                                                                     |
| disable_phpunit           | Set `true` to disable phpunit tests.                                                                                                                                                     |
| disable_grunt             | Set `true` to disable grunt.                                                                                                                                                             |
| disable_mustache          | Set `true` to disable mustache.                                                                                                                                                          |
| disable_master            | If `true`, this will skip testing against moodle/master branch                                                                                                                           |
| disable_release           | If `true`, this will skip the release job                                                                                                                                                |
| disable_phpcpd            | If `true`, this will skip phpcpd checks                                                                                                                                                  |
| disable_ci_validate       | If `true`, this will skip moodle-plugin-ci validate checks                                                                                                                               |
| enable_phpmd              | If `true`, to enable phpmd                                                                                                                                                               |
| release_branches          | Name of the non-standardly named branch which should run the release job                                                                                                                 |
| moodle_branches           | Specify the MOODLE_XX_STABLE branch you specifically want to test against. This is _not_ recommended, and instead you should configuring a supported range.                              |
| min_php                   | The minimum php version to test. Set this to support the minimum php version supported by the plugin. Defaults to '7.4', however more recent Moodle branches only test higher versions.  |
| ignore_paths              | Specify custom paths for CI to ignore. Third party libraries are ignored by default.                                                                                                     |
| ignore_names              | Specify custom names for CI to ignore. Third party libraries are ignored by default.                                                                                                     |
| mustache_ignore_names     | Specify mustache templates to ignore during the mustache lint check. Useful for complex templates that might trigger false positives.                                                    |
| codechecker_ignore_paths  | Specify paths to ignore during the PHP CodeSniffer check. Useful for third-party libraries or code that doesn't follow Moodle coding standards.                                          |
| phpdocchecker_ignore_paths| Specify paths to ignore during the PHPDoc check. Useful for third-party libraries or legacy code.                                                                                        |


#### Secrets
These secrets are passed to the environment variables with the same name (in uppercase), making them available during tests.

| Secret                    | Description                                                                    | Required                                                                                                  |
|---------------------------|--------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| moodle_org_token          | Token for publishing to the Moodle plugin directory                            | Only if publishing                                                                                        |
| brandname                 | Used for plugins that require a brand name for testing                         | No                                                                                                        |
| clientid                  | Used for plugins that require a client ID for testing                          | No                                                                                                        |
| payone_secret             | Used for payment gateway plugins that require a PAYONE secret for testing      | No                                                                                                        |
| mpay_secret               | Used for payment gateway plugins that require a MPAY secret for testing        | No                                                                                                        |
| ssh_private_key           | Used for installing dependency plugins that are hosted in private repositories | No                                                                                                        |

Example of using secrets:
```yaml
# .github/workflows/moodle-plugin-ci.yml
name: ci

on: [push, pull_request]

jobs:
  ci:
    uses: Wunderbyte-GmbH/catalyst-moodle-workflows/.github/workflows/ci.yml@main
    secrets:
      brandname: ${{ secrets.BRANDNAME }}
      clientid: ${{ secrets.CLIENTID }}
      payone_secret: ${{ secrets.PAYONE_SECRET }}
      # moodle_org_token: ${{ secrets.MOODLE_ORG_TOKEN }}
    with:
      # Other options here
```

### Running with dependencies
If you'd require to run your workflow against specific versions of a plugin you depend on, then you can specify a branch you'd like to run each job against. Like following:   

```yaml
jobs:
  moodle41:
    uses: Wunderbyte-GmbH/catalyst-moodle-workflows/.github/workflows/ci.yml@main
    with:
      disable_phpunit: true
      moodle_branches: MOODLE_401_STABLE
      extra_plugin_runners:
        moodle-plugin-ci add-plugin danmarsden/moodle-mod_attendance --branch MOODLE_401_STABLE

  moodle42:
    uses: Wunderbyte-GmbH/catalyst-moodle-workflows/.github/workflows/ci.yml@main
    with:
      disable_phpunit: true
      moodle_branches: MOODLE_402_STABLE
      extra_plugin_runners:
        moodle-plugin-ci add-plugin danmarsden/moodle-mod_attendance --branch MOODLE_402_STABLE
```

An example can be found here - https://github.com/danmarsden/moodle-block_attendance/blob/main/.github/workflows/ci.yml

### Add CI badge

With badges, we will be able to see at a glance from the plugin's `README.md` whether or not the plugin is in a good state for usage.

```
[![ci](https://github.com/[user]/[plugin]/actions/workflows/ci.yml/badge.svg?branch=[branch])](https://github.com/[user]/[plugin]/actions/workflows/ci.yml?branch=[branch])
```

Please update `[USER]`, `[PLUGIN]` and `[BRANCH]` in the example above. This goes under the plugin title. Here is an example:

```
[![ci](https://github.com/catalyst/moodle-tool_excimer/actions/workflows/ci.yml/badge.svg?branch=MOODLE_35_STABLE)](https://github.com/catalyst/moodle-tool_excimer/actions/workflows/ci.yml?branch=MOODLE_35_STABLE)
</a>
```

which renders as:

[![ci](https://github.com/catalyst/moodle-tool_excimer/actions/workflows/ci.yml/badge.svg?branch=MOODLE_35_STABLE)](https://github.com/catalyst/moodle-tool_excimer/actions/workflows/ci.yml?branch=MOODLE_35_STABLE)

## How does this automate tests?
When you call the reusable ci, it will:
1. a check to see what versions of moodle should run, based on the `version.php` file included in the plugin repository.
2. This will then build out the combination of tests to run, performing the tests based on the __MOODLE_XX_STABLE__ version affected and will handle any version specific caveats and run a more optimally configured test suite for you.

To view or modify the full matrix, please see it here: [.github/actions/matrix/matrix_includes.yml](.github/actions/matrix/matrix_includes.yml)

## How does this automate releases?

Whenever a change is made to version.php, the workflow is triggered on a release branch (e.g. __main__ / __MOODLE_XX_STABLE__), and tests pass, will it attempt to run the plugin/release action `.github/plugin/release/action.yml`. Doing so will automatically publish a release to the Moodle plugin directory at https://moodle.org/plugins.

In the prepare step of the CI, it will have resolved the component name for you so you don't need to enter one manually, and the `MOODLE_ORG_TOKEN` secret should be set otherwise the plugin won't be published.

To configure the secret:
* Please check `MOODLE_ORG_TOKEN` is available in your plugin's **Settings > Secrets** section. If not, please create using below steps:
  * Log in to the Moodle Plugins directory at https://moodle.org/plugins/
  * Locate the **Navigation block > Plugins > API access**.
    * Use that page to generate your personal token for the `plugins_maintenance` service.
  * Go back to your plugin repository at Github. Locate your plugin's **Settings > Secrets** section. Use the 'New repository secret' button to define a new repository secret to hold your access token. Use name `MOODLE_ORG_TOKEN` and set the value to the one you generated in previous step.
* For the latest branch/stable releases in plugin directory we **MUST** bump plugin version (e.g. by date).
* For the __older stables__ with closed groups, the version **MUST** be a <ins>**micro bump**</ins>.

## Common concerns / issues

### Core patches
If you need to apply core patches to the moodle code to allow the plugin to work on older versions missing API support, this can be achieved by outputting
an appliable diff to a file, in the `patch` directory in the top level of the repo. The version for a particular branch should be named `MOODLE_XX_STABLE.diff` in line with the naming of the branch. To generate a diff, these patches can be manually applied to the target branch, and a diff from the remote generated by doing `git format-patch MOODLE_XX_STABLE --stdout > my/plugin/path/patch/MOODLE_XX_STABLE.diff`.

### amd / grunt bundling issues

Depending on your supported range, you might encounter an issue which outputs something along the lines of `File is stale and needs to be rebuilt`. The brief reason for this, is that along the way, Moodle has updated the way it bundles JS/CSS files, which results in different outputs across Moodle versions.

To fix this, you'll need to rebundle the relevant files, on the <ins>highest supported version of Moodle</ins> for your plugin's support range. For example, if the plugin supports up to __Moodle 4.0__, you'll need to bundle the changes, while on the `MOODLE_400_STABLE` branch of Moodle, and then commit those changes.

__NOTE:__ This may involve having a clean copy of Moodle and installing the plugin code to run the necessary commands to rebuild the _stale_ files.

Grunt docs: https://docs.moodle.org/dev/Grunt#Running_grunt

## Workarounds

The `workarounds` parameter allows you to run custom commands before the main CI process starts. This is useful for setting environment variables, fixing common issues, or implementing temporary fixes for known problems. You can define your workaround commands in a YAML block, and they will be executed at the beginning of the CI workflow.


### Common Use Cases

- **Environment Variables**: Set custom environment variables that will be available to all subsequent steps in the workflow.
- **AMD Module Workarounds**: Fix the "File is stale and needs to be rebuilt" error that occurs when AMD modules import Moodle core dependencies.
- **Custom Path Ignores**: Specify files or directories to ignore during various testing steps.
- **Dependency Fixes**: Install or configure dependencies needed before the main tests run.

### Notes

- Environment variables set using `>> $GITHUB_ENV` will be available to all subsequent steps in the workflow.
- You can verify environment variables are set correctly by adding debug `echo` statements.
- The workarounds run **before** any of the standard Moodle plugin tests.
- This is the ideal place to put **temporary fixes** that should be removed when upstream issues are resolved.

### Example: AMD Module Workaround

```yaml
workarounds: |
  # Set additional environment variables
  # echo "SOME_PATHS=OpenTBS, TinyButStrong" >> $GITHUB_ENV

  # WORKAROUND: Fix for "File is stale and needs to be rebuilt" error
  # See issue: https://github.com/moodlehq/moodle-plugin-ci/issues/350
  export NVM_DIR="$HOME/.nvm"
  [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
  cd moodle
  nvm use
  npm install
  npm install -g grunt-cli
  cd ../plugin
  echo "=== Building AMD files before CI check ==="
  grunt --gruntfile ../moodle/Gruntfile.js amd
  cd ..
```
## Custom Steps

You can add custom steps to run at the end of the CI workflow by using the `custom_steps` parameter. This is useful for running additional tests or tasks specific to your plugin.

Example:

```yaml
custom_steps: |
  # Run Vue.js tests
  if [ -d "plugin/vue3" ]; then
    cd plugin/vue3
    npm install
    npm test
    cd ../..
  fi
  
  # Run any other custom commands
  echo "Running additional custom verification"
  php plugin/verify.php
```
These steps will be executed after all the standard Moodle plugin tests have completed.

### Benefits of This Approach

1. **Flexibility**: Users can add any custom steps they need
2. **Isolation**: Custom steps run at the end, so they don't interfere with the standard tests
3. **Optional**: The custom steps section is entirely optional
4. **Context**: The steps run in the same environment/workspace as the standard tests

This solution gives your users the flexibility to add their Vue.js testing or any other custom verification steps while keeping the core workflow standardized.

## Using the Labeled Issue Workflow in Your Moodle Plugin

This feature allows you to automatically create tasks in ERPNext when issues in your GitHub repository are labeled. Here's how to set it up:

### 🛠️ Setup Instructions

### Add the Workflow File

Create a new file in your Moodle plugin repository at: .github/workflows/erpnext.yml

```yaml
name: Create ERPNext Task from Labeled Issue

on:
  issues:
    types:
      - labeled

jobs:
  call-labeled-issue-workflow:
    uses: Wunderbyte-GmbH/catalyst-moodle-workflows/.github/workflows/labeled_issue.yml@main
    secrets:
      webhook_token: ${{ secrets.WEBHOOK_TOKEN }}
```

### Configure the Secret

In your Moodle plugin's GitHub repository:

1. Go to **Settings > Secrets and variables > Actions**
2. Click **New repository secret**
3. Set the name to `WEBHOOK_TOKEN`
4. Set the value to your ERPNext API token
5. Click **Add secret**

---

### How It Works

When someone adds a label to an issue in your repository:

- The workflow triggers automatically
- It creates a new task in **ERPNext** with the following details:

  - **Subject**: `GH#[issue-number] - [issue-title]`
  - **Project**: Set to the label name
  - **Status**: `Open`
  - **Description**: Link to the GitHub issue + issue description

---

### ✅ Testing

To test the workflow:

1. Create a new issue in your repository
2. Add any label to it
3. Check **ERPNext** for the newly created task

## Tag-based Release Workflow

If you prefer to trigger releases via git tags instead of version.php changes, you can use our tag-based release workflow:

1. Create a file in your repository at `.github/workflows/moodle-release.yml`
2. Add the following content:

```yaml
name: Release to Moodle Plugins Directory

on:
  push:
    tags:
      - v*-stable

  workflow_dispatch:
    inputs:
      tag:
        description: 'Tag to be released'
        required: true

jobs:
  release:
    uses: Wunderbyte-GmbH/catalyst-moodle-workflows/.github/workflows/tag-release.yml@main
    with:
      plugin_name: YOUR_PLUGIN_NAME_HERE  # Change this to your plugin's frankenstyle name (e.g., local_shopping_cart)
    secrets:
      moodle_org_token: ${{ secrets.MOODLE_ORG_TOKEN }}
```
3. Replace YOUR_PLUGIN_NAME_HERE with your plugin's frankenstyle name (e.g., local_shopping_cart)
4. Set up the MOODLE_ORG_TOKEN secret in your repository settings
5. Make sure to set disable_release: true when using the CI workflow to avoid duplicate releases

### Triggering a release
You can release your plugin in one of two ways:

- Push a tag matching the pattern `v*-stable` (e.g., v3.2-stable)
- Manually trigger the workflow with a specific tag name

And when using the CI workflow, remember to set `disable_release: true` to prevent duplicate releases

