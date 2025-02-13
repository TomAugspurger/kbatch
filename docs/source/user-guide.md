# User Guide

After you've [configured](./index.md#configure-with-jupyterhub-deployment) `kbatch`, you can submit jobs, list previously submitted jobs, and query the details of individual jobs.

## Submit a job

The simplest possible job includes a few key pieces of information:

1. A name, to later identify your job
2. A container image, that defines your software environment
3. A command to run

Let's make a simple job that just lists the files in a directory.

```{code-block} console
$ kbatch job submit \
    --name=list-files \
    --image=alpine \
    --command='["ls", "-lh"]'
{
  "api_version": "batch/v1",
  "kind": "Job",
  ...
}
```

The output is the full Job specification that was submitted to Kubernetes.

You can list running jobs:

```{code-block} console
$ kbatch job list -o table
                          Jobs
┏━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━┓
┃ name             ┃ submitted                 ┃ status ┃
┡━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━┩
│ list-files-jfprp │ 2021-11-08T15:07:49+00:00 │ done   │
└──────────────────┴───────────────────────────┴────────┘
```

And get details on any individual job:

```{code-block} console
$ kbatch job show list-files-jfprp
{
  "api_version": "batch/v1",
  "kind": "Job",
  ...
}
```

With `kbatch job logs` you can get the logs for a job. Make sure to pass the container id.

## Submitting code files

Your job likely relies on some local code files to function. Perhaps this is a notebook, shell script, or utility library that wouldn't be present in your container image. You can use the `-c` or `--code` option to provide a single file or list of files that will be made available before your job starts running.


```{code-block} console
$ kbatch job submit \
    --name=list-files \
    --image=alpine \
    --command='["sh", "my-script.sh"]' \
    --code="my-script.sh"
```

This will be available at the path `/code/my-script.sh` when the job is executing. The default working
directory is `/code/`, so you can refer to the script at `my-script.sh`.

## Configuration file

As an alternative to specifying arguments on the command-line, you can provide them through a YAML configuration file.

Rather than providing all those arguments on the command-line, you can create a [YAML] configuration file.

```yaml
# file: config.yaml
name: "my-job"
command:
  - sh
  - script.sh
image: "mcr.microsoft.com/planetary-computer/python:latest"
code: "script.sh"
```

And use it like

```{code-block} console
$ kbatch job submit -f config.yaml
```

You'll often use a mixture of the configuration file and arguments from the command-line. For example,
you might provide most arguments through the configuration file, but pass a secret token as `--env=$MY_ENV_VAR`.