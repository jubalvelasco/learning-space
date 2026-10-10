# Github Actions Event Triggers

[main docs](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#schedule)

## Multiple Events - Run on Push and Pull Request

```yml
on: [push, pull_request]
```

## Push

#### Run a workflow when the push event occurs.

```yml
on:
  push
```

#### run when someone pushes a change to a JavaScript file (.js)

```yml
on:
  push:
    paths:
      - '**.js'
```

#### Run when someone pushes to `main` or to a branch that starts with `releases/`

```yml
on:
  push:
    branches:
      - 'main'
      - 'releases/**'
```

#### Run when someone pushes to any branch that is neither `main` nor a branch that starts with `releases/`

```yml
on:
  push:
    branches-ignore:
      - 'main'
      - 'releases/**'
```

You cannot use both the branches and branches-ignore filters for the same event in a workflow.

#### run when a push that includes a change to a JavaScript (`.js`) file is made to a branch whose name starts with `releases/`

```yml
on:
  push:
    branches:
      - 'releases/**'
    paths:
      - '**.js'
```

#### Run when someone pushes a tag that starts with `v`

```yml
on:
  push:
    tags:
      - 'v*'
```

#### Run only on branches not on tags

```yml
on:
  push:
    branches:
      - '**'
  pull_request:
```

## Schedules

#### This example triggers the workflow every day at 5:30 and 17:30 UTC

(scheduled workflows always use UTC, and may be delayed during periods of high load)

```yml
on:
  schedule:
    # * is a special character in YAML so you have to quote this string
    - cron:  '30 5,17 * * *'
```

#### This example triggers the workflow to run at 5:30 UTC every Monday-Thursday, but skips the Not on Monday or Wednesday step on Monday and Wednesday.

```yml
on:
  schedule:
    - cron: '30 5 * * 1,3'
    - cron: '30 5 * * 2,4'

jobs:
  test_schedule:
    runs-on: ubuntu-latest
    steps:
      - name: Not on Monday or Wednesday
        if: github.event.schedule != '30 5 * * 1,3'
        run: echo "Workflow will run but this specific step will be skipped on Monday and Wednesday"
      - name: Every time
        run: echo "This step will always run"
```

#### Cron syntax

```yml
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of the month (1 - 31)
│ │ │ ┌───────────── month (1 - 12 or JAN-DEC)
│ │ │ │ ┌───────────── day of the week (0 - 6 or SUN-SAT)
│ │ │ │ │
* * * * *
```
