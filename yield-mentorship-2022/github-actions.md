# Github Actions

## Events

#### An event is a trigger for workflow

A common event that occurs on a repo, is when someone pushes code.

![test.yml](<../.gitbook/assets/image (9) (1).png>)

* on: push
  * We specify the workflow to trigger when someone pushes new code.
  * This will run all the jobs within the workflow.
  * There are other triggers: [https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

## Jobs&#x20;

![](<../.gitbook/assets/image (170).png>)

* In this workflow we have a single named, **super-lint**
* It specifies multiple **steps** and **actions**

**Under steps, there are 2 actions being run.**&#x20;

1. **First it is going to checkout our code with checkout@v2**
2. **Then its going to run the super-linter@v3 against it**

{% hint style="info" %}
runs-on: specifies a container environment for the code to run within. By default Github runs this container for you - but you can host your own if you wish.

The default container options are: ubuntu, windows, macOS
{% endhint %}

Runners

Steps

actions

