# crane

A GitLab CI ready image to upgrade services in Rancher.

## Usage

1. Deploy your application on Rancher manually,
   with a commit SHA tagged image.
2. Get a Rancher Environment API key
   and add the API keypair as secret variables in the project,
   named `RANCHER_ACCESS_KEY` and `RANCHER_SECRET_KEY`.
3. Also add `RANCHER_URL` and `RANCHER_ENV_ID`,
   preferably in secret variables, or in `.gitlab-ci.yml`.
   (In the example URL `https://rancher.example.com/env/1a81/apps/stacks/1e551/services/1s1456/containers`
   the environment ID is `1a81`. This ID always starts with `1a`.)
4. Add something like this to your `.gitlab-ci.yml`:

   ```yaml
   stages:
     # [...]
     - deploy

   deploy-production:
     stage: deploy
     image: kiwicom/crane
     script:
       - crane --stack my-app --service api --service worker --new-image $CI_REGISTRY_IMAGE:$CI_BUILD_REF
     environment:
       name: production
       url: https://my-app.example.com/
     when: manual
   ```

## Settings

| CLI flag                | Environment variable        | Required | Default |
| ----------------------- | --------------------------- | -------- | ------- |
| `--url`                 | `RANCHER_URL`               | Yes      |         |
| `--access-key`          | `RANCHER_ACCESS_KEY`        | Yes      |         |
| `--secret-key`          | `RANCHER_SECRET_KEY`        | Yes      |         |
| `--env`                 | `RANCHER_ENV_ID`            | Yes      |         |
| `--stack`               | `RANCHER_STACK_NAME`        | Yes      |         |
| `--service`             | `RANCHER_SERVICE_NAME`      | No       | app     |
| `--sidekick`            | `RANCHER_SIDEKICK_NAME`     | No       | None    |
| `--batch-size`          | `RANCHER_BATCH_SIZE`        | No       | 1       |
| `--batch-interval`      | `RANCHER_BATCH_INTERVAL`    | No       | 2       |
| `--start-first`         | `RANCHER_START_FIRST`       | No       | False   |
| `--new-image`           | `CRANE_NEW_IMAGE`           | No       | None    |
| `--sleep-after-upgrade` | `CRANE_SLEEP_AFTER_UPGRADE` | No       | 0       |
| `--manual-finish`       | `CRANE_MANUAL_FINISH`       | No       | False   |

## Integrations & Extensions

### Slack

When `--slack-token` is set,
crane can post an announcement to `--slack-channel`
with details about the ongoing deployment.
You can use `--slack-link` to add useful URLs to this announcements
such as Datadog dashboards, Sentry issues, or the project repository.

| CLI flag          | Environment variable  | Details                      |
| ----------------- | --------------------- | ---------------------------- |
| `--slack-token`   | `CRANE_SLACK_TOKEN`   | Slack API token              |
| `--slack-channel` | `CRANE_SLACK_CHANNEL` | Slack channel to announce in |
| `--slack-link`    | `CRANE_SLACK_LINK`    | links to mention in Slack    |

### Sentry

With `--sentry-webhook`, crane can post release details to Sentry.
[Release tracking](https://docs.sentry.io/learn/releases/#what-is-a-release) is useful
to provide additional context to errors tracked in Sentry.

| CLI flag           | Environment variable   | Details                    |
| ------------------ | ---------------------- | -------------------------- |
| `--sentry-webhook` | `CRANE_SENTRY_WEBHOOK` | Sentry release webhook URL |

### Generic webhooks

With the `--webhook-url` option,
you can specify URLs that crane will send release info to,
in its own format.
One use for this is for analytics;
if somebody sets up a listener for these events,
they'll have the data needed to identify correlations
between releases and changes in user behavior or sales numbers.

| CLI flag          | Environment variable  | Details                      |
| ----------------- | --------------------- | ---------------------------- |
| `--webhook-url`   | `CRANE_WEBHOOK_URL`   | URLs to post release info to |
| `--webhook-token` | `CRANE_WEBHOOK_TOKEN` | Auth token for webhooks      |
