---
title: Use the Python SDK for Quantum Inspired Optimization
description: This documents provides a basic usage overview of the Python SDK for Quantum Inspired Optimization.
author: KittyYeungQ
ms.author: kitty
ms.date: 06/29/2020
ms.topic: article
uid: microsoft.azure.quantum.qio.python-sdk
---

# Use the Python SDK for Quantum-Inspired Optimization

This guide provides a basic overview of how to use the Python SDK for Quantum-Inspired Optimization. It assumes you have already completed the [Creating an
Azure Quantum Workspace guide](xref:microsoft.azure.quantum.workspaces-portal).

## Installation

The Python SDK is distributed as the `azure-quantum` [PyPI](https://pypi.org)
package. During this private preview, you'll need to follow these instructions to
install it from the private Azure Quantum feed:

1. Install [Python](https://www.python.org/downloads/) 3.6 or later.
1. Install [PIP](https://pip.pypa.io/en/stable/), the Python Package Installer,
   and ensure you have **version 19.2 or higher**
1. Install the `azure-quantum` python package.

   ```bash
   pip install --upgrade azure-quantum --pre
   ```

## Connecting to a Quantum Workspace

A `Workspace` represents the Quantum Workspace you [previously created](xref:microsoft.azure.quantum.workspaces-portal) and is the main interface for
interacting with the service.

```python
from azure.quantum import Workspace

workspace = Workspace(
    subscription_id = "", # The Subscription ID for your Azure Subscription
    resource_group  = "", # The resource group your Quantum Workspace is deployed in
    name            = ""  # The name of the Quantum Workspace
)
workspace.login()
```

The values for `subscription_id`, `resource_group`, and `name` are from the [Quantum Workspace you created](xref:microsoft.azure.quantum.workspaces-portal).
You may get these values for an existing workspace by running `az quantum workspace show`.


When you call the command `login`, the SDK will see the following displayed in your console:

```output
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code [RANDOM-CODE] to authenticate.
```

Once you complete signing into your Azure account, your credentials will be
cached so that you do not have to repeat this process for future runs.

## Expressing and solving a simple problem

To express a simple problem to be solved, create an instance of a `Problem` and
set the `problem_type` to either `ProblemType.ising` or
`ProblemType.pubo`. For more information, see [`ProblemType`](xref:microsoft.azure.quantum.reference.python-sdk.azure.quantum.optimization).

```py
from azure.quantum.optimization import Problem, ProblemType, Term, ParallelTempering

problem = Problem(name="My First Problem", problem_type=ProblemType.ising)
```

Next, create an array of terms and add them to the `problem`:

```py
terms = [
    Term(w=-9, indices=[0]),
    Term(w=-3, indices=[1,0]),
    Term(w=5, indices=[2,0]),
    Term(w=9, indices=[2,1]),
    Term(w=2, indices=[3,0]),
    Term(w=-4, indices=[3,1]),
    Term(w=4, indices=[3,2])
]

problem.add_terms(terms=terms)
```
![NOTE]
> There are [multiple ways](xref:microsoft.azure.quantum.qio.python-sdk.advanced#Methods-for-supplying-problem-terms)
> to supply terms to the problem, and not all terms must be added at once.

Next, we're ready to solve by applying a **solver**. In this example we'll use a
parameter-free version of parallel tempering. You can find documentation on this
solver and the other available solvers in the [Azure Quantum Provider
Reference](xref:microsoft.azure.quantum.providers.azure-quantum).

```py
solver = ParallelTempering(workspace, timeout=100)
```

For arguments, the solver takes the `workspace` created previously, plus a single
parameter which is the maximum amount of time (in seconds) to run the solver.
Detailed documentation on parameters is available in the [Azure Quantum Provider
Reference](xref:microsoft.azure.quantum.providers.azure-quantum).

Solvers provide an `optimize` method that expects a `Problem` which uploads the
problem definition, submits a job, and polls the status until the job has completed
running. This is a blocking method that waits for the job to complete
running. It returns a `JobOutput` object with the job ID and results.

```py
result = solver.optimize(problem)
print(result)
```

> Note: If you receive an `Unauthorized` error when calling `solver.optimize()`, please see [this guide](Common-issues#error-operation-returned-an-invalid-status-code-unauthorized).

This method will submit the problem to Azure Quantum for optimization and
synchronously wait for it to be solved. You'll see output like the following in
your terminal:

```
> {'energy': -32.0, 'solution': [1, 1, -1, 1]}
```

## Next Steps

Ready to learn more about creating and solving optimization problems? Check out our samples:
- [Solving a simple shipping optimization problem](https://github.com/MicrosoftDocs/quantum-docs-private/blob/feature/onboarding-azure-quantum/azure-quantum/samples/shipping-sample.ipynb) (Jupyter Notebook)
- `[Coming Soon]` Job Shop Scheduling

This guide provides an overview of a simple use case. `Problem` also provides
individual methods for `upload` and `submit`; `submit` returns an instance of
the `Job` which exposes the job metadata and `fetch`, `get_output` and `cancel`
methods. These provide more fine-grain control for the run, and are
covered in [Advanced Usage of the Python SDK](xref:microsoft.azure.quantum.qio.python-sdk.advanced).

You may also want to review the [solvers reference for Microsoft Optimization Solvers](xref:microsoft.azure.quantum.providers.azure-quantum).

## Common issues

Refer to this document for common issues you may run into during the limited preview: [Common
issues](xref:microsoft.azure.quantum.common-issues)