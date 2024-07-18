# Expressions, Contexts, Functions, Environment Variables & Secrets.

[Main Documentation] (https://docs.github.com/en/actions/learn-github-actions/expressions).

Expressions can be used inside the run part of every step. They follow this format:

```
${{ Expresison }}
```
Here's an example

```yaml
name: External Events
on:
    repository_dispatch:
        types: [build]

jobs:
    echo-a-string:
        runs-on: ubuntu-latest
        steps:
            - run: echo ${{ github.event.client_payload.env }}
```

## Contexts

It's the _first word_ of an expression:

The context here is **github**:

```js
${{ github.event.client_payload.env }}
```

The context here is **inputs**:

```js
${{ inputs.texto }}
```

The context here is the **steps**:
```js
${{ steps.greet.outputs.time }}
```

## An example

```yaml
name: Expressions & Contexts
on: [push, pull_request, issues, workflow_dispatch]

jobs:
    using-expressions-and-contexts:
        runs-on: ubuntu-latest
        steps:
            - name: Expressions
              id: expressions_id
              # We can use numbers, strings, null, boolean values, comparison and logical operators
              run: |
                echo ${{ 1 }}                           
                echo ${{ 'This is a string' }}          
                echo ${{ null }}
                echo ${{ true }}
                echo ${{ 1 > 2 }}
                echo ${{ 'string' == 'String' }}
                echo ${{ true && false }}
                echo ${{ true || (false && true) }}
            - name: Dump Contexts
              # Here we use a function (toJson) to explore content of some contexts
              run: |
                echo '${{ toJson(github) }}'
                echo '${{ toJson(job) }}'
                echo '${{ toJson(secrets) }}'
                echo '${{ toJson(steps) }}'
                echo '${{ toJson(runner) }}'
```

## The IF key

Can be used in a job level or in a step level.

In a job level it's used like this:

```yaml
name: Expressions & Contexts
on: [push, pull_request, issues, workflow_dispatch]
run-name: 'Expressions & Contexts by @${{ github.actor }}, event ${{ github.event_name }}'

jobs:
    using-expressions-and-contexts:
        runs-on: ubuntu-latest
        if: ${{ github.event_name == 'push' }}  # This job will be executed only with push event
```

But we don't need to use the **${{ }}** syntax because Github actions will understand that everything in the IF is an expression.

```yaml
name: Expressions & Contexts
on: [push, pull_request, issues, workflow_dispatch]
run-name: 'Expressions & Contexts by @${{ github.actor }}, event ${{ github.event_name }}'

jobs:
    using-expressions-and-contexts:
        runs-on: ubuntu-latest
        if: github.event_name == 'push' # This job will be executed only with push event
```