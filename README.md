# DR2 GitHub actions

This is a repository of [reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows) and [custom actions](https://docs.github.com/en/actions/creating-actions/about-custom-actions) to use with our repository builds.

See the [GitHub actions](https://docs.github.com/en/actions) page for more information.

## Reusable workflows

### DR2 test

This workflow runs git secrets against the repository provided in the input. It then runs a test command provided as an input. 

#### Inputs

#### `repo-name`
**Required** The name of the repository to run the tests against.

#### `test-command`
**Required** The command to run tests for this repository.

#### Secrets
#### `MANAGEMENT_ACCOUNT`
**Required** The management account number. Used for assuming an AWS role to download our dependencies from S3.

#### `SLACK_WEBHOOK`
**Required** The slack webhook to post messages to the release channel.

#### Example usage

```yaml
jobs:
  test:
    uses: nationalarchives/dr2-github-actions/.github/workflows/dr2_test.yml@main
    with:
      repo-name: dr2-preservica-client
      test-command: |
        sbt test
    secrets:
      MANAGEMENT_ACCOUNT: ${{ secrets.MANAGEMENT_ACCOUNT }}
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

### Test
This workflow tests the custom actions and runs on push and pull request creation. It carries out the following steps:
* Adds the string 111111111111 to a tmp file and adds it.
* Runs the git secrets custom action.
* Checks that the git secrets call fails.
* Runs the slack send custom action with an input url of example.com.
* Checks the status returned is 200.

### Scala Steward
This workflow runs once per day at 08:15. It runs Scala Steward against all of the repositories in [repos.md](./repos.md).

The pull requests are created using the tna-da-bot user

There is a config file in .github/scala-steward.conf. This is ignoring updates from AWS for now because they release a new patch version every day and this is causing a lot of updates. Once AWS release a new minor version, Scala Steward will create a pull request for that.

## Custom Actions

### Slack send
This is a [composite action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)

It sends a message to the webhook provided in the secret passed to the action.

#### Inputs

#### `message`
**Required** The message to send.

#### `slack-url`
**Required** The webhook url to send the message to. You cannot explicitly set secrets in a custom action but GitHub knows this is sensitive and won't print it.

#### Outputs
#### `status`
The http status code of the response from the slack URL post.

#### `Example usage`
```yaml
- name: Send success message
    uses: nationalarchives/dr2-github-actions/.github/actions/slack-send@main
    with:
      message: ":white_check_mark: Service updated successfully"
      slack-url: ${{ secrets.SLACK_WEBHOOK }}
``` 

### Run git secrets.
This is a [Docker container action](https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action)

It runs git secrets against the currently checked out repository.

When the image builds, it installs git secrets to an alpine container. When the container runs, git secrets is configured with the patterns we need and run against the files.

#### Example usage
```yaml
- uses: nationalarchives/dr2-github-actions/.github/actions/run-git-secrets@main
```

## Deploying the workflows and actions.

As this is an internal library, we can have the workflows in the other repositories pointed at the `main` branch of this repository. 
So once changes here are merged to `main` they are available immediately to other repositories.

If there is an error which means we need to use an earlier version, the calling repository can reference a specific commit. 
```yaml
uses: nationalarchives/github-actions/.github/workflows/ecs_deploy.yml@ecb24cbe882bdf4568f8558aec72b7053824920f
```
