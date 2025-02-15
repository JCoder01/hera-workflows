# Hera (hera-workflows)

```text
The Argo was constructed by the shipwright Argus,
and its crew were specially protected by the goddess Hera.
```

(https://en.wikipedia.org/wiki/Argo)

[![Build](https://github.com/argoproj-labs/hera-workflows/actions/workflows/cicd.yaml/badge.svg)](https://github.com/argoproj-labs/hera-workflows/blob/main/.github/workflows/cicd.yaml)

[![codecov](https://codecov.io/gh/argoproj-labs/hera-workflows/branch/main/graph/badge.svg?token=x4tvsQRKXP)](https://codecov.io/gh/argoproj-labs/hera-workflows)

[![Pypi](https://img.shields.io/pypi/v/hera-workflows.svg)](https://pypi.python.org/pypi/hera-workflows)
[![CondaForge](https://anaconda.org/conda-forge/hera-workflows/badges/version.svg)](https://anaconda.org/conda-forge/hera-workflows)
[![Versions](https://img.shields.io/pypi/pyversions/hera-workflows.svg)](https://github.com/argoproj-labs/hera-workflows)

[![Downloads](https://pepy.tech/badge/hera-workflows)](https://pepy.tech/project/hera-workflows)
[![Downloads/month](https://pepy.tech/badge/hera-workflows/month)](https://pepy.tech/project/hera-workflows)
[![Downloads/week](https://pepy.tech/badge/hera-workflows/week)](https://pepy.tech/project/hera-workflows)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Hera is a Python framework for constructing and submitting Argo Workflows. The main goal of Hera is to make Argo
Workflows more accessible by abstracting away some setup that is typically necessary for constructing workflows.

Python functions are first class citizens in Hera - they are the atomic units (execution payload) that are submitted for
remote execution. The framework makes it easy to wrap execution payloads into Argo Workflow tasks, set dependencies,
resources, etc.

You can watch the introductory Hera presentation at the "Argo Workflows and Events Community Meeting 20 Oct
2021" [here](https://www.youtube.com/watch?v=QETfzfVV-GY&t=181s)!

# Table of content

- [Assumptions](#assumptions)
- [Installation](#installation)
- [Examples](#examples)
- [Contributing](#contributing)
- [Concepts](#concepts)
- [Comparison](#comparison)

# Assumptions

Hera is exclusively dedicated to remote workflow submission and execution. Therefore, it requires an Argo server to be
deployed to a Kubernetes cluster. Currently, Hera assumes that the Argo server sits behind an authentication layer that
can authenticate workflow submission requests by using the Bearer token on the request. To learn how to deploy Argo to
your own Kubernetes cluster you can follow the
[Argo Workflows](https://argoproj.github.io/argo-workflows/quick-start/) guide!

Another option for workflow submission without the authentication layer is using port forwarding to your Argo server
deployment and submitting workflows to `localhost:2746` (2746 is the default, but you are free to use yours). Please
refer to the documentation of [Argo Workflows](https://argoproj.github.io/argo-workflows/quick-start/) to see the
command for port forward!

In the future some of these assumptions may either increase or decrease depending on the direction of the project. Hera
is mostly designed for practical data science purposes, which assumes the presence of a DevOps team to set up an Argo
server for workflow submission.

# Installation

There are multiple ways to install Hera:

1. You can install from [PyPi](https://pypi.org/project/hera-workflows/):

   ```shell
   pip install hera-workflows
   ```

2. You can install from [conda](https://anaconda.org/conda-forge/hera-workflows):

   ```shell
   conda install -c conda-forge hera-workflows
   ```

3. Install it directly from this repository using:

   ```shell
   python -m pip install git+https://github.com/argoproj-labs/hera-workflows --ignore-installed
   ```

4. Alternatively, you can clone this repository and then run the following to install:

   ```shell
   pip install .
   ```

# Examples

A very primitive example of submitting a task within a workflow through Hera is:

```python
from hera import Task, Workflow


def say(m: str):
    print(m)


with Workflow('my-workflow') as w:
    Task('say', say, ['Hello, world!'])

w.create()
```

See the [examples](https://github.com/argoproj-labs/hera-workflows/tree/main/examples) directory for a collection of
Argo workflow construction and submission via Hera!

# Contributing

If you plan to submit contributions to Hera you can install Hera in a virtual environment managed by `poetry`:

```shell
poetry install
```

In you activated `poetry shell`, you can utilize the tasks found in `tox.ini`, e.g.:

To run tests on all supported python versions with coverage run [tox](https://tox.wiki/en/latest/):

```shell
tox
```

To list all available `tox` envs run:

```shell
tox -a
```

To run selected tox envs, e.g. for a specific python version with coverage run:

```shell
tox -e py37,coverage
```

As `coverage` *depends* on `py37`, it will run *after* `py37`

See project `tox.ini` for more details

Also, see the [contributing guide](https://github.com/argoproj-labs/hera-workflows/blob/main/CONTRIBUTING.md)!

# Concepts

Currently, Hera is centered around two core concepts. These concepts are also used by Argo, which Hera aims to stay
consistent with:

- `Task` - the object that holds the Python function for remote execution/the atomic unit of execution;
- `Workflow` - the higher level representation of a collection of tasks.

# Comparison

There are other libraries currently available for structuring and submitting Argo Workflows:

- [Couler](https://github.com/couler-proj/couler), which aims to provide a unified interface for constructing and
  managing workflows on different workflow engines;
- [Argo Python DSL](https://github.com/argoproj-labs/argo-python-dsl), which allows you to programmaticaly define Argo
  worfklows using Python.

While the aforementioned libraries provide amazing functionality for Argo workflow construction and submission, they
require an advanced understanding of Argo concepts. When [Dyno Therapeutics](https://dynotx.com) started using Argo
Workflows, it was challenging to construct and submit experimental machine learning workflows. Scientists and engineers
at [Dyno Therapeutics](https://dynotx.com) used a lot of time for workflow definition rather than the implementation of
the atomic unit of execution - the Python function - that performed, for instance, model training.

Hera presents a much simpler interface for task and workflow construction, empowering users to focus on their own
executable payloads rather than workflow setup. Here's a side by side comparison of Hera, Argo Python DSL, and Couler:

<table>
<tr><th>Hera</th><th>Couler</th><th>Argo Python DSL</th></tr>
<tr>

<td valign="top"><p>

```python
from hera import Task, Workflow


def say(message: str):
    print(message)


with Workflow("diamond") as w:
    a = Task('a', say, ['This is task A!'])
    b = Task('b', say, ['This is task B!'])
    c = Task('c', say, ['This is task C!'])
    d = Task('d', say, ['This is task D!'])

    a >> [b, c] >> d

w.create()
```

</p></td>

<td valign="top"><p>

```python
import couler.argo as couler
from couler.argo_submitter import ArgoSubmitter


def job(name):
    couler.run_container(
        image="docker/whalesay:latest",
        command=["cowsay"],
        args=[name],
        step_name=name,
    )


def diamond():
    couler.dag(
        [
            [lambda: job(name="A")],
            [lambda: job(name="A"), lambda: job(name="B")],  # A -> B
            [lambda: job(name="A"), lambda: job(name="C")],  # A -> C
            [lambda: job(name="B"), lambda: job(name="D")],  # B -> D
            [lambda: job(name="C"), lambda: job(name="D")],  # C -> D
        ]
    )


diamond()
submitter = ArgoSubmitter()
couler.run(submitter=submitter)
```

</p></td>

<td valign="top"><p>

```python
from argo.workflows.dsl import Workflow

from argo.workflows.dsl.tasks import *
from argo.workflows.dsl.templates import *


class DagDiamond(Workflow):

    @task
    @parameter(name="message", value="A")
    def A(self, message: V1alpha1Parameter) -> V1alpha1Template:
        return self.echo(message=message)

    @task
    @parameter(name="message", value="B")
    @dependencies(["A"])
    def B(self, message: V1alpha1Parameter) -> V1alpha1Template:
        return self.echo(message=message)

    @task
    @parameter(name="message", value="C")
    @dependencies(["A"])
    def C(self, message: V1alpha1Parameter) -> V1alpha1Template:
        return self.echo(message=message)

    @task
    @parameter(name="message", value="D")
    @dependencies(["B", "C"])
    def D(self, message: V1alpha1Parameter) -> V1alpha1Template:
        return self.echo(message=message)

    @template
    @inputs.parameter(name="message")
    def echo(self, message: V1alpha1Parameter) -> V1Container:
        container = V1Container(
            image="alpine:3.7",
            name="echo",
            command=["echo", "{{inputs.parameters.message}}"],
        )

        return container
```

</p></td>
</tr>
</table>
