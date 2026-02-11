# Examiner Clickable Actions

Actions for running verification tests to check whether a workshop user has completed required steps. The test examiner must be enabled in the workshop configuration.

## examiner:execute-test

Runs a test script to verify user progress. The script must be an executable in the `workshop/examiner/tests` directory.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `name` | string | (required) | Name of the test executable in `workshop/examiner/tests/`. Can include subdirectory prefix |
| `title` | string | (required) | Title displayed on the clickable action |
| `args` | array | — | Arguments passed to the test program |
| `timeout` | integer | `15` | Seconds before the test is killed and marked failed. `0` uses default |
| `retries` | integer | `0` | Number of times to retry on failure. `.INF` or `-1` for unlimited retries |
| `delay` | integer | — | Seconds between retries |
| `cooldown` | number | `3` | Seconds before the action can be clicked again. `.INF` or `-1` to prevent re-clicking |
| `url` | string | — | URL of a backend service to perform the test instead of a local script. Hostname must share the ingress domain |

The test script must exit with status `0` for success and non-zero for failure. The working directory is the workshop user's home directory.

**Example — basic test:**

````markdown
```examiner:execute-test
name: test-pod-exists
title: Verify the pod is running
args:
- my-pod
```
````

**Example — test with retries (continuous polling):**

````markdown
```examiner:execute-test
name: test-deployment-ready
title: Verify deployment is ready
args:
- my-deployment
timeout: 5
retries: .INF
delay: 1
```
````

**Example — test in a subdirectory:**

````markdown
```examiner:execute-test
name: kubernetes/test-service-exists
title: Verify the service was created
args:
- my-service
```
````

### Tests with user input

The `inputs` property allows a test to collect user input via an HTML form before running. Input data is provided to the script as JSON on standard input.

**Properties for inputs:**

| Property | Type | Description |
|----------|------|-------------|
| `inputs.schema` | mapping | Field definitions using [jsonform](https://github.com/jsonform/jsonform/wiki) schema format |
| `inputs.form` | array | Form layout. Use `["*"]` to render all fields, plus a submit button |

**Example — test with form inputs:**

````markdown
```examiner:execute-test
name: deploy-application
prefix: Task
title: Deploy application
inputs:
  schema:
    name:
      type: string
      title: "Name:"
      default: "my-app"
      required: true
    replicas:
      type: integer
      title: "Replicas:"
      default: "1"
      required: true
  form:
  - "*"
  - type: submit
    title: Deploy
```
````

The test script can read the JSON input from stdin:

```bash
#!/bin/bash
CONFIG=$HOME/exercises/config.json
cat - > $CONFIG
NAME=$(jq -r -e ".name" $CONFIG)
REPLICAS=$(jq -r -e ".replicas" $CONFIG)
```

Note: Do not use `autostart` with tests that require user input, as it would prevent users from filling in the form.
