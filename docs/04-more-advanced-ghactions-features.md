# Diving Deeper with More Advanced Github Actions Features
## Contents
- Timeout Minutes & Continue on Error
- Running a Job Multiple Times Using a Mtrix
- Including & Excluding Matrix Configurations
- Hanling Failing Jobs in a Matrix
- Step and Job Outputs & Dynamic Matrices
- Running a Single Job or Workflow at a Time Using Concurrency
- Reusable Workflows
- Reusable Workflows Outputs
- Nesting Reusable Workflows
- Caching Files in Github Actions
- Updating Cache Keys Dinamically && Adding Restore Keys
- Cache Limits & Restrictions
- Uploading & Downloading Job Artifacts


## Timeout Minutes & Continue on Error

### continue-on-error
At step level. If true then the execution of the following steps continues even if this step fails.

In this example Step 2 fails. But as we have configured the step with **continue-on-error: true** then all the following steps are executed as if there wasn't one fail. For example, **success()** function return true, just as is there was not a failure.
```yaml
    steps:
      - name: Step 1
        run: sleep 80
        timeout-minutes: 1
      - name: Step 2
        id: step-2
        continue-on-error: true
        run: exit 1
      - name: Runs on setp 2 Failure
        if: failure() && steps.step-2.conclusion == 'failure'
        run: echo 'Step 2 has failed.'
      - name: Runs on Failure
        if: failure() 
        run: echo 'Failure'        
      - name: Runs on success
        # this is not needed beacuse this is the default behaviour.
        if: success()
        run: echo 'Runs on Success'
```

### timeout-minutes:

Stablish a number of minutes for stopping the execution of our workflow. For example this step will be cancelled after 1 minute:

```yaml
    steps:
      - name: Step 1
        run: sleep 90
        timeout-minutes: 1
      - name: Step 2
        id: step-2
        continue-on-error: true
        run: exit 1
```

This can be set also at job level (so the time corresponds to the full job)

## Running a Job Multiple Times Using a Mtrix
If you need to run the job multiple times you can define a matrix and the job will be executed multiple times.

At job level you can create a **strategy** tag, where we can configure the matrix and the behaviour of the job.

```yaml
jobs:
  node-matrix:
    continue-on-error: false
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [14, 15, 16]
      fail-fast: false
      max-parallel: 3
```

Here we can notice some different options. First we have a two dimension matrix. The first fimmension name is **os**, and the second dimmension is called **node-version**. The name of every dimension is free and the user can choose whatever is the most clear word. As you can see we have 2 values in the **os** dimension, and 3 values on **node-version**. This means that the job will be executed 6 times, one with every combination of values of the matrix. That is:

- One execution in ubuntu-latest, for version 14.
- One execution in ubuntu-latest, for version 15.
- One execution in ubuntu-latest, for version 16.
- One execution in windows-latest, for version 14.
- One execution in windows-latest, for version 15.
- One execution in windows-latest, for version 16.

We need to use the context to use the values of the matrix inside every execution. For example:

```yaml
    runs-on: ${{ matrix.os }}
    steps:
      - run: node -v
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - run: node -v
```

Here we use the value of **os** for the runs-on option, and the **node-version** inside one step.

Every execution of the matrix will be launched in parallel. If we want to limit the number of parallel executions we can use the option **max-parallel** to set the maximum number of parallel executions.


## Including & Excluding Matrix Configurations

We can configure and addapt the matrix values in a fine grain way with the **includes** keyword. Let's start with this example:

```yaml
jobs:
  node-matrix:
    continue-on-error: false
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [14, 15]
```

In this example we have 4 executions for the job:

1. **os: ubuntu-latest** and **node-version: 14**.
2. **os: ubuntu-latest** and **node-version: 15**.
3. **os: windows-latest** and **node-version: 14**.
4. **os: windows-latest** and **node-version: 15**.

Now we can create a **include** tag. Inside include tag you can define different objects to fine tune the matrix. Let's see some examples:

```yaml
jobs:
  node-matrix:
    continue-on-error: false
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [14, 15]
        include: 
          - os: ubuntu-latest
            is-ubuntu: true
          - os: macos-latest
            node-version: 15
          - experimental: false
          - os: ubuntu-latest
            node-version: 16
            experimental: true
          - os: ubuntu-latest
            node-version: 17            
```

The first thing we need to understand is that **include** will try to find existing combinations, and change this combinations, before adding one more option to the matrix. Let's see the first **include** object:

```yaml
          - os: ubuntu-latest
            is-ubuntu: true
```

This seems that is adding another tag (__is-ubuntu__) to the matrix, so now we have more combinations. But don't work that way. What this **include** object is doing is add the property **is-ubuntu**, with the value of **true** to every combination of the matrix where **os** tag has the **ubuntu-latest** value. Then our matrix will have this values:

1. **os: ubuntu-latest**, **node-version: 14** and **is-ubuntu: true**.
2. **os: ubuntu-latest**, **node-version: 15** and **is-ubuntu: true**.
3. **os: windows-latest** and **node-version: 14**.
4. **os: windows-latest** and **node-version: 15**.

Combinations 3 and 4 don't have the new tag with a value because they don't match the os property with the value "ubuntu-latest". This works as pattern matching in elixir or rust.

If we add an option with a tag that exists but a value that don't matches any existing combination, then a new execution is added to the matrix. This is what the next object is doing:

```yaml
          - os: macos-latest
            node-version: 15
```

After the addition of this object the execution matrix has this combinations:

1. **os: ubuntu-latest**, **node-version: 14** and **is-ubuntu: true**.
2. **os: ubuntu-latest**, **node-version: 15** and **is-ubuntu: true**.
3. **os: windows-latest** and **node-version: 14**.
4. **os: windows-latest** and **node-version: 15**.
5. **os: macos-latest** and **node-version: 15**.

In the case we add a new tag, that don't exists previously in the matrix, this is added as a new property and value to the existing combinations, not a new ones. For example this line:

```yaml
          - experimental: false
```

After this line inside **includes** the matrix execution combination has this values:

1. **os: ubuntu-latest**, **node-version: 14**, **is-ubuntu: true** and **experimental: false**.
2. **os: ubuntu-latest**, **node-version: 15**, **is-ubuntu: true** and **experimental: false**.
3. **os: windows-latest**, **node-version: 14** and **experimental: false**.
4. **os: windows-latest**, **node-version: 15** and **experimental: false**.
5. **os: macos-latest**, **node-version: 15**.

As you can see in the original matrix there wasn't a tag called experimental, so this tag is added to all the combinations, but no new execution is created. You muyst notice that this only changes the original values for the matrix, so the **macos-latest** combination remains unchanged. This is very important.

Now, If we want to change a value for the **experimental** tag, we can use patter matching as before:

```yaml
          - os: ubuntu-latest
            node-version: 15
            experimental: true
```

These are the new values:

1. **os: ubuntu-latest**, **node-version: 14**, **is-ubuntu: true** and **experimental: false**.
2. **os: ubuntu-latest**, **node-version: 15**, **is-ubuntu: true** and **experimental: true**.
3. **os: windows-latest**, **node-version: 14** and **experimental: false**.
4. **os: windows-latest**, **node-version: 15** and **experimental: false**.
5. **os: macos-latest**, **node-version: 15** and **experimental: false**.

With the pattern matching technique we changed the value **experimental** to **true** for a combination of operating system and node version.

Finally, if you add new combinations, they don't get the new values and tag created, only what you configure:

```yaml
          - os: ubuntu-latest
            node-version: 17            
```

1. **os: ubuntu-latest**, **node-version: 14**, **is-ubuntu: true** and **experimental: false**.
2. **os: ubuntu-latest**, **node-version: 15**, **is-ubuntu: true** and **experimental: true**.
3. **os: windows-latest**, **node-version: 14** and **experimental: false**.
4. **os: windows-latest**, **node-version: 15** and **experimental: false**.
5. **os: macos-latest**, **node-version: 15** and **experimental: false**.
6. **os: ubuntu-latest**, **node-version: 17**.

As you can see this new combination don't have a tag or value for **experimental**.

There's also an **exclude** tag that you can use to exclude executions that matches the pattern. For example we can add this object to exclude the execution of a node version inside one operating systema:


```yaml
        excludes:
          - os: windows-version
            node-version: 15            
```

If we make this exclusion we will have this executions:

1. **os: ubuntu-latest**, **node-version: 14**, **is-ubuntu: true** and **experimental: false**.
2. **os: ubuntu-latest**, **node-version: 15**, **is-ubuntu: true** and **experimental: true**.
3. **os: windows-latest**, **node-version: 14** and **experimental: false**.
4. **os: macos-latest**, **node-version: 15** and **experimental: false**.
5. **os: ubuntu-latest**, **node-version: 17**.

You can see we now have one less execution.


## Hanling Failing Jobs in a Matrix

In the first section we saw how **continue-on-error** tag works in conjungtion with **fail-fast** tag**:
- **continue-on-error**. Default option is false, and that means that if a job fails Github Actions will cancel all next jobs. In the case of a matrix all not executed combinations will not be executed. Changing this option to true will make that all the jobs will be executed even if some of them fails.
- **fail-fast**. If this is set to true Github Actions will cancel running jobs when one of the jobs fails.

We can configure this better using the matrix options and their tags. If, for example, we want that jobs with the option **experimental: true** have the **continue-on-error** to true, so all the other jobs will be executed, we can configure this using the matrix context. In this example we allow the jobs to fail if they are experimental or macos.

```yaml
jobs:
  node-matrix:
    continue-on-error: ${{ (matrix.experimental == true) || (matrix.os == 'macos-latest') }}
```

## Step and Job Outputs & Dynamic Matrices

### Step output

We can output values from a step to use in next steps of a job. We do this adding key=value strings to a special file contained in $GITHUB_OUTPUT. Here we have an example. In the first and second step we create outputs in every step. We can create more than one output in any step:


```yaml
jobs:
  output-values-example:
    runs-on: ubuntu-latest
    steps:
      - run: echo "step-output=value" >> $GITHUB_OUTPUT
        id: step-1
      - run: |
          echo "type-of-animal=butterfly" >> $GITHUB_OUTPUT
          echo "color=red" >> $GITHUB_OUTPUT
        id: step-2
      - run: |
          echo '${{ steps.step-1.outputs.step-output }}'
          echo '${{ steps.step-2.outputs.type-of-animal }}'
          echo '${{ steps.step-2.outputs.color }}'
```

In this example we have created one step that has an output with the key **step-output** and the value **value**. Then there is another step that created 2 outputs. The first one is **type-of-animal** which value is **butterfly**, and the second one is **color** with the value **red**.

Then in the last step we access this outputs with the context **steps.id of the step.outputs.key of the output** syntax.

### Job outputs

Step outputs are not available outside of their job. If we need this we must set the job output. Here we have an interesting example:

```yaml
  prepare-matrix:
    runs-on: ubuntu-latest
    outputs:
      matrix-arrays: ${{ steps.matrix-arrays.outputs.result }}
    steps:
      - name: Get all configured options
        id: matrix-arrays
        uses: actions/github-script@v6
        with: 
          script: "return {os: context.payload.inputs['os'].split(','), 'node-version': context.payload.inputs['node-version'].split(',') }"
          result-encoding: json
      - name: Check the values recovered
        run: echo '${{ steps.matrix-arrays.outputs.result }}'
```

In the **prepare-matrix** jon we find the **outputs** tag. Here we create and output which name is **matrix-arrays** and the value is attached to the output of a step inside the job (which name is **matrix-arrays**).

In this case we used a Github Action (actions/github-script) that allows us to use javascript and returns everything we put in the return sentence. In this special case we convert two comma separated strings in an json object containing two arrays of values.

### Dynamic Matrices

We can then configure a job with a matrix in a dynamic way. 

First thing we need to do is to configure the job for executing after the **prepare-matrix** job, that created de output with the matrix values.

```yaml
  node-matrix:
    needs: prepare-matrix
```
After that we set our strategy, creating a matrix for every tag:

```yaml
  node-matrix:
    needs: prepare-matrix
    strategy:
      matrix:
        os: ${{ fromJson(needs.prepare-matrix.outputs.matrix-arrays).os }}
        node-version: ${{ fromJson(needs.prepare-matrix.outputs.matrix-arrays).node-version }}
```

The complete code is this:

```yaml
  node-matrix:
    needs: prepare-matrix
    strategy:
      matrix:
        os: ${{ fromJson(needs.prepare-matrix.outputs.matrix-arrays).os }}
        node-version: ${{ fromJson(needs.prepare-matrix.outputs.matrix-arrays).node-version }}
    runs-on: ${{ matrix.os }}
    steps:
      - run: node -v
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - run: node -v
```

## Running a Single Job or Workflow at a Time Using Concurrency

## Reusable Workflows
## Reusable Workflows Outputs
## Nesting Reusable Workflows
## Caching Files in Github Actions
## Updating Cache Keys Dinamically && Adding Restore Keys
## Cache Limits & Restrictions
## Uploading & Downloading Job Artifacts

