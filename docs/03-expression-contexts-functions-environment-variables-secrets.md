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
You can use also if at a step level.

### Success

Github actions applies success condition to all if key by default. If we have this **if** key:

```yaml
if: steps.step-2.conclusion == 'failure'
```

Github Actions will _translate_ to something like this:

```yaml
if: success() && steps.step-2.conclusion == 'failure'
```

So if we need to apply this kind of condition you need to explicitly check for failure():

```yaml
if: failure() && steps.step-2.conclusion == 'failure'
```


## Functions


### contains
Functions that checks if a string is contained in another one. 

```js
contains(string, searched_string)
```

### Using an array with contains

```js
// First you need an array:

["first_value", "second_value"]

// You need no stringify this array with fromJson function

fromJson('["first_value", "second_value"]')     // See the use of simple quotation marks

// Te result is this
contains(fromJson('["first_value", "second_value"]'), "searched_string")        // false
contains(fromJson('["first_value", "second_value"]'), "second_value")           // true

```

### Access properties of an array of objects

If, for example, our workflow is launched by the creation or modification of an issue, this issue can have multiple labels. So inside the **github** context we can access the issue data through **github.events.issue**. Inside this theres one array of objets called **labels**. Every label has different properties (for example name). If we want to check if any label name of an issue is 'bug' (or any other string) an option is to get an array of all the names inside de **labels** array. We do this with this special syntax:

```js
    github.event.issue.labels.*.name            // This returns an array of names. ['bug', 'other', 'anything']
```

### Join

Joins in a string every member of an array seperated by a character

```js
    join(array, separator)

    join(github.event.issue.labels.*.name, ',')
    join(github.event.issue.labels.*.name, '||')
```

## Only execute steps depending on result (status) of previous steps