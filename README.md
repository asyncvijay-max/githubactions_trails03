On GitHub, “events” are notifications that something happened in a repository, organization, or account, such as a push, an opened pull request, or a new issue comment. The same events show up in three places.

1. Webhook events. GitHub sends an HTTP POST to a URL you configure whenever the event happens. Common ones:

push: commits pushed to a branch or tag
pull_request: opened, closed, merged, synchronized, labeled, and so on
pull_request_review / pull_request_review_comment: reviews and review comments
issues / issue_comment: issue activity and comments
create / delete: a branch or tag created or deleted
release: a release published, edited, or deleted
fork, watch (starring), star
workflow_run / workflow_job / check_run / check_suite: CI activity
deployment / deployment_status
member, team, organization, repository: access and settings changes
security_advisory, dependabot_alert, code_scanning_alert, secret_scanning_alert

Each event usually carries an action field (for example opened, closed, edited) that narrows down what happened.

2. GitHub Actions trigger events. These are the events you put under on: in a workflow file. Most mirror the webhook events above (push, pull_request, issues, release, ...). A few exist only in Actions:

workflow_dispatch: a manual run from the UI or API
schedule: cron-based runs
repository_dispatch: a custom event sent through the API
workflow_call: a reusable workflow called by another workflow
pull_request_target: like pull_request, but it runs in the context of the base repo and has write access, so it needs care with forks
merge_group: for merge queues
yaml
on:
  push:
    branches: [main]
  pull_request:
    types: [opened, synchronize]
  workflow_dispatch:

3. Events API (activity feed). This is a read-only REST endpoint, such as GET /repos/{owner}/{repo}/events or GET /users/{user}/events, that returns recent public activity. Its event names are slightly different and end in “Event”: PushEvent, PullRequestEvent, IssuesEvent, WatchEvent, ForkEvent, CreateEvent, ReleaseEvent, and so on. It isn’t real-time and only covers about the last 90 days.

If you tell me which of these you’re working with, I can go into the payloads or the specific triggers you need.

can you go bit slower for a beginner

Sure, let’s start from the beginning.

What’s an “event”?

An event is just GitHub noticing that something happened. Every time someone does something on GitHub, it gets recorded as an event:

You upload new code → that’s a push event
Someone opens a bug report → that’s an issues event
Someone asks to merge their changes into your project → that’s a pull_request event
Someone stars your project → that’s a watch event

Think of it like the notifications on your phone. Something happens, and a little message says “hey, this just happened.”

Why do events matter?

The useful part is that you can tell GitHub: “when this event happens, do something automatically.”

For example: “Whenever someone pushes new code, run my tests.” Then you don’t have to remember to run the tests yourself. GitHub sees the push event and kicks them off for you.

The most common way beginners use this: GitHub Actions

GitHub Actions is GitHub’s built-in automation tool. You write a small file that says which event to listen for and what to do when it happens. Here’s a tiny example:

yaml
on: push

jobs:
  say-hello:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Someone pushed code!"

Line by line:

on: push means “start this whenever someone pushes code”
jobs: is the list of things to do
runs-on: ubuntu-latest means “use a computer GitHub provides”
run: echo ... means “print this message”

You’d save this file in your project at .github/workflows/hello.yml, push it, and GitHub would run it automatically.

The three events to learn first

push: new code was uploaded
pull_request: someone proposed changes
workflow_dispatch: you clicked a “Run” button yourself (handy for testing)

Once those make sense, the rest are variations on the same idea.
============

on: [push, workflow_dispatch]

jobs:
  say-hello:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello! Something happened!"
      - run: date

      ====================
xplaining how shell commands run on the CI workflow runner.
Explaining how shell commands run on the CI workflow runner.
Whatever you put after run: is a normal command-line command. GitHub runs it in the terminal of the computer named in runs-on.

Since you used runs-on: ubuntu-latest, your commands run on Linux (Ubuntu). So any Linux terminal command will work, not just commands made for GitHub. echo and date are both standard Linux commands.

Good beginner commands to try

yaml
      - run: pwd          # show which folder you're in
      - run: ls           # list files in the current folder
      - run: whoami       # show the username
      - run: uname -a     # show details about the computer
      - run: python3 --version   # Python comes pre-installed
Running several commands in one step

Put a | after run: and write the commands on separate lines below it, indented:

yaml
      - run: |
          echo "Starting..."
          date
          ls
Where to learn more commands

Linux terminal basics: search for “Linux commands for beginners” (Ubuntu’s own command-line tutorial is a good start). Anything you learn there works in run:.
What’s already installed on GitHub’s computers: GitHub publishes a full list of preinstalled software (Python, Node.js, Git, Java, Docker and more) in the actions/runner-images repository on GitHub. Anything on that list can be used straight away.
GitHub’s own docs: search for “GitHub Actions workflow syntax” to see everything you can write in a workflow file, run: included.
=====

The next step is to get your repository’s files onto GitHub’s computer, so your workflow can actually work with your code. You do this with a ready-made action called actions/checkout.

What’s an “action”?
So far you’ve written your own commands with run:. An action is a ready-made step someone else has written that you can reuse. You use one with uses: instead of run:. actions/checkout is the most common one. Its job is to copy your repository’s files onto the runner.

Try it

First, add a file to your repository so there’s something to see. On the Code tab, click Add file → Create new file, name it notes.txt, type a sentence in it, and commit.

Then edit hello.yml to look like this:

yaml
on: [push, workflow_dispatch]

jobs:
  say-hello:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - run: ls
      - run: cat notes.txt

Here’s what each step does:

uses: actions/checkout@v6 copies your repository’s files onto the runner. The @v6 picks the version.
run: ls lists the files. This time you’ll see notes.txt, README.md and the others.
run: cat notes.txt prints what’s inside notes.txt.

Commit it, then open the new run in the Actions tab and expand each step. If you compare the ls output with before, you’ll see that checkout made your files available.

Why this matters
Once your code is on the runner, you can do real work with it, like running tests or checking for errors. That’s how most real projects use GitHub Actions: someone pushes code, the workflow checks it out, runs the tests, and shows a ✓ or ✗.

Note: the version number in @v6 goes up over time. GitHub’s own actions/checkout page always shows the current one at the top of its instructions, so use that if it shows a newer number.

Drafting a continuous integration workflow configuration.

The idea is the same as with Playwright, just with Java tools. The job needs to:

Check out the test code
Install Java
Run Maven, which downloads dependencies (Selenium, TestNG) and runs the tests
Upload the test reports

GitHub’s Ubuntu VMs already have Maven and Google Chrome installed, and Selenium 4.6 or newer downloads the matching ChromeDriver automatically. So you only need to set up Java.

The job

yaml
name: Selenium TestNG Tests

on: [push, workflow_dispatch]

jobs:
  selenium-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6

      - name: Set up Java
        uses: actions/setup-java@v5
        with:
          distribution: temurin
          java-version: '17'
          cache: maven

      - name: Run Selenium tests
        run: mvn -B test

      - name: Upload test reports
        uses: actions/upload-artifact@v4
        if: ${{ !cancelled() }}
        with:
          name: testng-reports
          path: target/surefire-reports/

What each part does

actions/setup-java installs Java:
distribution: temurin is a free, widely used Java build.
java-version: '17' should match the version your project uses (17 and 21 are common).
cache: maven saves downloaded Maven libraries between runs, so later runs are much faster.
mvn -B test runs your tests. -B means “batch mode”, which gives cleaner logs in CI.
target/surefire-reports/ is where Maven saves the TestNG results, including the HTML report. Uploading it lets you download the report from the run page, even when tests fail.

Two things your project must have

1. TestNG connected to Maven in pom.xml. If you run tests through a testng.xml suite file, the Surefire plugin needs to point at it:

xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-surefire-plugin</artifactId>
      <version>3.2.5</version>
      <configuration>
        <suiteXmlFiles>
          <suiteXmlFile>testng.xml</suiteXmlFile>
        </suiteXmlFiles>
      </configuration>
    </plugin>
  </plugins>
</build>

If mvn test works on your own computer, this part is already fine.

2. Chrome must run in headless mode. GitHub’s VM has no screen, so Chrome has to run without a visible window. Where you create the driver:

java
ChromeOptions options = new ChromeOptions();
options.addArguments("--headless=new");
options.addArguments("--no-sandbox");
options.addArguments("--disable-dev-shm-usage");
options.addArguments("--window-size=1920,1080");
WebDriver driver = new ChromeDriver(options);

Forgetting headless mode is the most common reason Selenium tests pass locally but fail in GitHub Actions.

Using it with the two-repo setup

If the Selenium tests live in a separate repository and should test the deployed app, the job looks like this:

yaml
  qa-automation:
    runs-on: ubuntu-latest
    needs: deploy
    steps:
      - uses: actions/checkout@v6
        with:
          repository: your-username/selenium-tests
          # token: ${{ secrets.REPO2_TOKEN }}   ← only if the repo is private

      - uses: actions/setup-java@v5
        with:
          distribution: temurin
          java-version: '17'
          cache: maven

      - name: Run Selenium tests
        run: mvn -B test -DbaseUrl=${{ needs.deploy.outputs.app_url }}

      - uses: actions/upload-artifact@v4
        if: ${{ !cancelled() }}
        with:
          name: testng-reports
          path: target/surefire-reports/

-DbaseUrl=... passes the website address into Java. In your test code you read it like this:

java
String baseUrl = System.getProperty("baseUrl", "https://example.com");
driver.get(baseUrl);

The second value, "https://example.com", is a default used when you run the tests locally without -DbaseUrl.

Running only part of your tests

yaml
        run: mvn -B test -Dgroups=smoke                      # only TestNG tests in the "smoke" group
        run: mvn -B test -Dsurefire.suiteXmlFiles=smoke.xml  # a different suite file

If you share your project’s structure (or your pom.xml), I can check that it will work as is.

whats the difference btw using code this way on: ['push', 'workflow_dispatch'] vs name: Playwright Tests on: push: branches: [main, master] pull_request: branches: [main, master]

There are three differences: which events, which branches, and the writing style. (The name: line is unrelated to triggers. It’s just the workflow’s display name in the Actions tab.)

1. Which events trigger it

	on: ['push', 'workflow_dispatch']	Second version
Push	✓	✓
Manual “Run workflow” button	✓	✗
Pull requests	✗	✓

The first version listens for pushes and the manual button. The second listens for pushes and pull requests, but has no Run workflow button.

2. Which branches

First version: a push to any branch triggers it.
Second version: only pushes to the main or master branch, and only pull requests that target main or master. A push to a branch like feature-login does not trigger it.

3. Writing style

Both are valid YAML. They’re two ways of writing the same kind of setting:

Short list form: on: [push, workflow_dispatch]. Quick and simple, but you can’t add any extra options.
Expanded form: each event on its own line, with options like branches: underneath.

The quotes in ['push', 'workflow_dispatch'] don’t matter. With or without them, it means the same thing.

Combining both

You can use the expanded form to get everything: branch filters plus the manual button.

yaml
name: Playwright Tests

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]
  workflow_dispatch:

workflow_dispatch: is written with nothing after the colon, which simply means “turn this on with no extra options.”

Which to use

For learning and experiments: on: [push, workflow_dispatch] is simple and runs whenever you change anything.
For real projects: the expanded form with branch filters is the usual choice. Tests run when code reaches the main branch or when someone proposes changes to it, and you avoid wasting runs on every small push to work-in-progress branches.

