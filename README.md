# CLR
This repository includes modified Keras optimizers that allow implementation of three cyclical learning rate policies, as detailed in this paper https://arxiv.org/abs/1506.01186.

clr_optimizers.py is a modified version of Keras' default optimizer module (https://github.com/fchollet/keras/blob/master/keras/optimizers.py) that includes 3 CLR policies, triangular, triangular2, and exp_range.

Each optimizer can now implement cyclical learning rate with one of the following three approaches:

## `triangular`

![Alt text](triangular.png?raw=true "Title")


This method is a simple triangular cycle.

To use `triangular` clr, simply pass into any optimizer the following:
```python
clr = {
    "mode":"triangular",
    "step_size":step_size,
    "max_lr":max_lr
    }
```   
Where `step_size` is half the period of the cycle in iterations,
and `max_lr` is the peak of the cycle (default lr is the base lr).

## `triangular2`

![Alt text](triangular2.png?raw=true "Title")

This method is a triangular cycle that decreases the cycle amplitude by half after each period, while keeping the base lr constant.

To use `triangular2` clr, simply pass into any optimizer the following:
```python
clr = {
    "mode":"triangular2",
    "step_size":step_size,
    "max_lr":max_lr
    }
```    
Where `step_size` is half the period of the cycle in iterations,
and `max_lr` is the peak of the cycle (default lr is the base lr).

## `exp_range`

![Alt text](exp_range.png?raw=true "Title")

This method is a triangular cycle that scales the cycle amplitude by a factor `gamma**(iterations)` at each iteration, while keeping the base lr constant.

To use `exp_range` clr, simply pass into any optimizer the following:
```python
clr = {
    "mode":"exp_range",
    "step_size":step_size,
    "max_lr":max_lr,
    "clr_gamma": gamma
    }
```    
Where `step_size` is half the period of the cycle in iterations,
`max_lr` is the peak of the cycle (default lr is the base lr),
and `clr_gamma` is the scaling factor.

## Changing/resetting Cycle
If you want to change your cycle to a different range:
```python
model.optimizer.lr.set_value(new_base_lr)
model.optimizer.max_lr.set_value(new_max_lr)
```
When changing the range, you should also reset the cycle iterations if you are using `triangular2` or `exp_range` to ensure your cycle starts at full range. Otherwise, the range will immediately be scaled by the previous number of cycle iterations.
```python
model.optimizer.clr_iterations.set_value(0)
```
Note: `clr_iterations` is distinct from the global `iterations` attibute that is used; this is because many of the adaptive optimizers require a global iteration count that you may not want to reset when changing to a new cycle (although you can reset that separately if you'd like). 

## Order of learning rate augmentation
Note that clr happens prior to any further learning rate adjustments as called for in a given optimizer, with the exception of learning rate decay. If implemented, decay happens prior to clr; this alters the cycle by lowering the minimum lr; the max_lr is kept constant.

clr_optimizer_tests.ipynb contains tests demonstrating desired behavior of optimizers in comparison to toy functions.
