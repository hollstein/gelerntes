---
title: "Optimizing black box functions purely from A / B comparisons"
date: 2023-06-14T10:27:55+02:00
draft: false
tags: ["python", "llm", "optimization"]
---

Promting or prompt engineering is becoming the new programming paradigm for many NLP tasks. Among other things, it means specifying the prompt for a given task as well as generation parameters such as temperature, top-k or penalty-alpha. This is, in fact, an optimization task over the space of possible prompts, suitable LLMs as well as generation. Some o parameters here are discrete (e.g. top-k) and some are continuous (e.g. temperature). There are ways to directly learn prompts or to fine-tune an LLM on a given task, but this might be more costly and time-consuming compared to simply selecting a set of good enough prompt, LLM and generation parameters (80-20 rule). 

Back to this optimization task: For many instances, there will not be a good metric that one can automatically compute and that is very well aligned with the task at hand. For these cases, manual judgement or labeling will be needed. However, rating outputs consistently on a scale is very difficult and unpractical for many reasons. What should be easy for most humans is to express preference over one outcome versus another one.

So, we need an optimization approach that doesn't act on direct function values but on comparisons given by a human, which is efficient in exploring the parameter space to make best use of human time. 

After some research on the current state of the art, I'm settling with [botorch](https://botorch.org/) which implements methods for this approach. They way to use it is to imports the used libraries:

```python
from botorch.acquisition.preference import AnalyticExpectedUtilityOfBestOption
from botorch.models.pairwise_gp import (
    PairwiseGP, 
    PairwiseLaplaceMarginalLogLikelihood
)
from botorch.models.transforms.input import Normalize
from botorch.optim import optimize_acqf
from botorch.fit import fit_gpytorch_mll

import numpy as np
import torch
from itertools import combinations
from scipy.stats import multivariate_normal
import warnings
warnings.filterwarnings('ignore')
````

and to give the to be optimized utility function. Here I'm using a toy-example which is a multinomial Gaussian, but in a real application a human would express preferences over generated text:


```python
# data generating helper functions, not to be called directly :-)
def __utility(X):
    """Given X, output corresponding utility (i.e., the latent function)"""
    global mn
    return mn.pdf(X)

def compare(xx,ii,jj):
    """Costly eval function here"""
    
    return [
        x[0] for x in sorted(
            [(ii,__utility(xx[ii])),(jj,__utility(xx[jj]))],
            key=lambda x:x[1],reverse=True
        )
    ]
```


Specify some settings:

```python
NUM_BATCHES = 15
dim = 2
NUM_RESTARTS = 3
RAW_SAMPLES = 512
q = 2  # number of points per query
q_comp = 1 # number of comparisons per query
noise = 0.01
```

And run the optimization loop:

```python
bounds = torch.stack([torch.zeros(dim), torch.ones(dim)])

# user either one
#x_max_unknown = np.random.uniform(low=0,high=1,size=dim)  # random center point
x_max_unknown = np.array([0.492, 0.53])  # put in center so graph looks nice

mn = multivariate_normal(mean=x_max_unknown,cov=.05)

print("x max:",x_max_unknown)
print()

xx = np.random.uniform(low=0,high=1,size=(q,dim))  # initial random dataset
comparisons = np.array(
    [
        compare(xx,ii,jj)
        for ii,jj in list(combinations(range(len(xx)), 2))
    ]
)
model = PairwiseGP(
    torch.tensor(xx),
    torch.tensor(comparisons),
    input_transform=Normalize(d=xx.shape[-1]),
)

x_max_trace=[]

with warnings.catch_warnings():
    for j in range(1, NUM_BATCHES + 1):
        
        next_X, acq_val = optimize_acqf(
            acq_function=AnalyticExpectedUtilityOfBestOption(pref_model=model),
            bounds=bounds,
            q=q,
            num_restarts=NUM_RESTARTS,
            raw_samples=RAW_SAMPLES,
        )

        xx = np.concatenate([xx,next_X.numpy()])
        comparisons = np.concatenate([
            comparisons, 
            np.array([compare(xx,len(xx)-2,len(xx)-1)])]
        )
        
        
        model = PairwiseGP(
            torch.tensor(xx),
            torch.tensor(comparisons),
            input_transform=Normalize(d=xx.shape[-1]),
        )        
        
        
        i_x_max = int(model.utility.argmax())
        x_max = model.datapoints[i_x_max].tolist()
        x_max_trace.append(x_max)
        
        if j%2==0:
            print(
                f"j={j:02}: {','.join([f'{j:.3}' for j in x_max])} "
                f"->  {float(model.utility[i_x_max]):.4} "
                f"L2: {((x_max - x_max_unknown)**2).sum():.4f}"
            )
```

This is the expected output:

```verbose
x max: [0.492 0.53 ]

j=02: 0.385,0.323 ->  0.5722 L2: 0.0543
j=04: 0.359,0.518 ->  0.7756 L2: 0.0179
j=06: 0.538,0.466 ->  0.8494 L2: 0.0063
j=08: 0.514,0.486 ->  0.9426 L2: 0.0025
j=10: 0.514,0.486 ->  1.032 L2: 0.0025
j=12: 0.514,0.486 ->  1.048 L2: 0.0025
j=14: 0.514,0.486 ->  1.101 L2: 0.0025
```

Visualize what is going on:

```python
import plotly.express as px
import pandas as pd
import plotly.graph_objects as go


df = pd.DataFrame(
    [
        {
            "x":pp[0],
            "y":pp[1],
            "u":__utility(pp)
        }
        for pp in np.random.uniform(low=0,high=1,size=(1500,dim))
    ]
)
fig = px.scatter(df, x="x", y="y",color="u")
fig.update_layout(xaxis=dict(domain=[0, 1.0]), yaxis=dict(domain=[0.0, 1.0]))


x,y = x_max
ax,ay = x_max
fig.add_annotation(
    xref="x",
    yref="y",
    x=x,
    y=y,
    font={"size":20},
    text=f"X",
    axref="x",
    ayref="y",
    ax=ax,
    ay=ay,
    arrowhead=2,
)
fig.add_trace(
    go.Scatter(x=[x[0] for x in x_max_trace],y=[x[1] for x in x_max_trace],mode="lines",name="opt path",line={"width":5})
)

for ii,(ci,cj) in enumerate(comparisons.tolist()):
    x,y = xx[ci].tolist()
    ax,ay = xx[cj].tolist()
    fig.add_annotation(
        xref="x",
        yref="y",
        x=x,
        y=y,
        arrowcolor="#BCBBB5",
        text=f"{ii}",
        axref="x",
        ayref="y",
        ax=ax,
        ay=ay,
        arrowhead=2,
    )
    
fig.show()
```

{{< load-plotly >}}
{{< plotly json="https://hollstein.github.io/gelerntes/optimization.json" height="400px" >}}

We see that the algorithm picks comparison points to update the expected optimum point in parameter space in a quite efficient way.