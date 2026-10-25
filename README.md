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
