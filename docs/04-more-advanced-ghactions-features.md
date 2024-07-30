# Diving Deeper with More Advanced Github Actions Features
## Contents
- Timeout Minutes & Continue on Error
- Running a Job Multiple Times Using a Mtrix
- INcluding & Excluding Matrix Configurations
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


## Running a Job Multiple Times Using a Mtrix
## INcluding & Excluding Matrix Configurations
## Hanling Failing Jobs in a Matrix
## Step and Job Outputs & Dynamic Matrices
## Running a Single Job or Workflow at a Time Using Concurrency
## Reusable Workflows
## Reusable Workflows Outputs
## Nesting Reusable Workflows
## Caching Files in Github Actions
## Updating Cache Keys Dinamically && Adding Restore Keys
## Cache Limits & Restrictions
## Uploading & Downloading Job Artifacts

