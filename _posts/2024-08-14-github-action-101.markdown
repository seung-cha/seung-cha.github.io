---
layout: post
title: "Github Action 101"
date: 2024-08-14 22:19:00 +1100
categories: Coding
---
This post goes through the basic syntax needed to write the most primitive form of Github Action. 

Introduction
---
To use Github Action, a workflow file is required. Workflow files are located in `.github/workflows` and have extension of `.yaml` or `.yml`.

The File
---
Workflow files consist of two parts: metadata and jobs.
Metadata assigns a name to the workflow (displayed on Github) and a set of triggers that activates it.

A typical file starts with the `name` `run-name` and `on`.
`name` and `run-name` are optional. `on` is a set of triggers that activate the workflow. A typical workflow file that is triggered when a commit is pushed looks as follows:
{% highlight yaml %}
name: Example Workflow
run-name: Run this example workflow
on: push

jobs:
    runs-on:
        ...
{% endhighlight %}

Workflow can be triggered by schedule using `schedule` with `cron`. `cron` defines the interval at which the workflow is run.
{% highlight yaml %}
on:
    schedule:
        - cron: '0 6 * * * ' # minute, hour, day, month, day-of-the-week
{% endhighlight %}
Here, workflow is defined to run at 6am everyday (UTC time).
Execution may be delayed if the server is overloaded by high traffic.


Jobs
---
Jobs are the essence of Action. Jobs define a set of tasks to carry out when the workflow is triggered. Jobs are defined as follows:
{% highlight yaml %}

on: ...

jobs:
    Name: 
        runs-on: ubuntu-latest
        steps:
            -uses: actions/checkout@v4
            -run: ...
            -run: ...
{% endhighlight %}
Here, a job named `Name` runs on `ubuntu-latest` to carry out a set of tasks defined by `run`s. `uses` is a syntax to perform a pre-made action. In this case, `actions/checkout@v4` is used to checkout the latest commit of the repository.

`Name` uniquely identifies a job and is required. By default, Github runs each job in parallel. If a job must run after another job, you can use `needs` to define this dependency.
{% highlight yaml %}

on: ...

jobs:
    Name: 
        needs: Previous_Job
        runs-on: ubuntu-latest
        steps:
            -uses: actions/checkout@v4
            -run: ...
            -run: ...
{% endhighlight %}
This job requires job named `Previous_Job` to successfully finish.

Each `run` is executed in sequence and is independent to each other.
{% highlight yaml %}

on: ...

jobs:
    Name: 
        needs: Previous_Job
        runs-on: ubuntu-latest
        steps:
            -run: cd dir    # at root/dir
            -run: run_task  # called run_task at root
{% endhighlight %}

Multiple commands can be executed in one `run` by following:
{% highlight yaml %}
run:|
    line1
    line2
    ...
{% endhighlight %}

What To Do?
---
This post will expand as I continuously explore Github Action.
One thing I want to understand is how it detects if a command fails.