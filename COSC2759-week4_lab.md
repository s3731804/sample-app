# COSC2759 Week 4 - Lab 

## Goals

* Build on last weeks lab to explore more about GitHub actions
* Cover a more realistic example for a CI pipeline

## Create Repository and Get Sample App

1. Create a new repo for this week (see Week 3 for instructions if you need help)
2. Copy the contents of the `sample-app` folder into your repository. 

Your repo should look like this once completed:

```text
<your repo name>
├── __tests__
├── app
├── pages
├── ...
├── package.json
├── package-lock.json
└── ...
```

## Create a Feature Branch

1. Create a feature branch for our pipeline (once again see Week 3 for help if needed)
2. Push the new branch up to GitHub

## Explore the Sample App

The sample app in this case is a small Next.JS application. Next.JS can be used to build small, medium and large scale websites and applications (think Canvas / RMIT Home Site / Banking sites / etc.). While the size of the app will change, the way that we will run testing and linting does not, meaning this is a great way to get started with these concepts in our CI. 

## CI (Lint Step)

1. Firstly we are going to start with the lint step of the pipeline. It's always a good idea to ensure that we can run the command locally before putting it in our CI (e.g. so we can catch any early errors). Run the lint step by running `npm install` and then once completed (this may take a few mins) run `npm run lint`. These commands will need to be run in the same level as your `package.json`. If successful we will see no output (any errors will be returned if there are any).

2. Now that we know our linting step works lets create our CI pipeline. Create a new pipeline file at `.github/workflows/ci.yml` with the following content:

```yaml
name: Lab 4 (Lint and Unit Test)

on: 
  push: 
    branches: 
      main 
  pull_request: 
    branches: 
      main 
      
jobs: 
  lint: 
    runs-on: ubuntu-latest 
    steps: 
      - uses: actions/checkout@v7 

      - name: Use Node.js 24.x 
        uses: actions/setup-node@v7 
        with: 
          node-version: "24" 

      - name: Install Deps
        run: npm install

      - name: Run Lint
        run: npm run lint
```

This pipeline will run the lint step on our code every time we push to main, or make a PR targeting the main branch. As each job is run on a fresh VM we will need to install tools (e.g. Node) each time.

Commit and push the branch to GitHub and then open a PR to `main` to watch it run. Don't complete the PR yet as we can continue to push changes (e.g. our next test step) and have the CI run each time.

## CI (Unit Test Step)

1. Once again lets verify all our unit tests are passing before we integrate them into the CI. Run `npm run test:ci` to verify. If successful we should have 6 tests passing.

2. Extend our existing CI pipeline to add a new unit test job:

```yaml
# ...

  unit-test: 
    runs-on: ubuntu-latest 
    steps: 
      - uses: actions/checkout@v7 

      - name: Use Node.js 24.x 
        uses: actions/setup-node@v7 
        with: 
          node-version: "24" 

      - name: Install Deps
        run: npm install

      - name: Run Unit Tests
        run: npm run test:ci
        
      - if: success() || failure() 
        uses: actions/upload-artifact@v7 
        with: 
          name: unit-test-${{ github.sha }} 
          path: coverage/lcov-report
```

In the above job we run the test:ci step against our codebase. Sometimes we would generate output files that we wish to keep, in which case we could upload the report as an artifact so we could refer to it later. The code for that is below: 

```yaml
    steps: 
      - uses: actions/checkout@v7

    # ... Your Unit Test Step

      - if: success() || failure() 
        uses: actions/upload-artifact@v7 
        with: 
          name: unit-test-${{ github.sha }} 
          path: coverage/lcov-report
```

In most cases we won't care about reports for unit tests (we can just look at the output to see the failing one) but for longer running steps, or more complex ones (e.g. e2e tests) storing the output can be helpful. For e2e tests the report will often include screenshots or video of the failing test(s).

> Feel free to download and view the generated report from the CI, you can open the `index.html` file in your web browser to view it.

## Disabling Jobs

1. Lets test disabling a job in our CI, add an `if: false` just above the `runs-on: ubuntu-latest` line for the lint job to disable it. 
2. Push this to GitHub and see how GitHub will now skip the lint job.
3. Think about other conditions you might have, or when you would use this to control when jobs run.
4. Revert this change to enable the lint step again.

## Making Jobs Dependent on Each Other

1. Often when running CI pipelines we wish to exit the pipeline as soon as a failure occurs (called fail fast and fail first). Pipelines are often setup so that the quicker / easier jobs run first, then we increment up to more expensive and time consuming (read costly) operations last. There is no point running a 5 - 10 min test suite if our 10 second lint will fail on some invalid code. To make jobs rely on previous steps running first we can use the `needs` property. It can take in a single item or an array of steps. Update the unit-test step like so:

```yaml
  # ...

  unit-test: 
    needs: lint
    runs-on: ubuntu-latest 
    steps: 
      - uses: actions/checkout@v4
      
    # ...
```

2. Push this up to GitHub and see the pipeline running, notice how it looks different to before, and how the unit test step only runs once the lint step has completed (and passed).

### Extension (Optional)

1. Can you make the lint step fail? 
2. Can you make the unit test step fail?