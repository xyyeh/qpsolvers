# QP Solvers for Python

Wrapper around Quadratic Programming (QP) solvers in Python, with a unified
interface.

## Installation

The simplest way to install this module is:
```sh
# Python 2
sudo apt install python-dev
pip install qpsolvers

# Python 3
sudo apt install python3-dev
pip3 install qpsolvers
```
You can add the ``--user`` parameter for a user-only installation. See also the
[documentation](https://scaron.info/doc/qpsolvers/installation.html) for
advanced installation instructions.

## Usage

The function ``solve_qp(P, q, G, h, A, b, lb, ub)`` is called with the ``solver``
keyword argument to select the backend solver. The quadratic program it solves
is, in standard form:

![Equation of Quadratic Program](doc/src/images/qp.gif)

Vector inequalities are taken coordinate by coordinate.

## Solvers

The list of supported solvers currently includes:

- Dense solvers:
    - [CVXOPT](http://cvxopt.org/)
    - [qpOASES](https://github.com/coin-or/qpOASES)
    - [quadprog](https://pypi.python.org/pypi/quadprog/)
- Sparse solvers:
    - [ECOS](https://web.stanford.edu/~boyd/papers/ecos.html)
    - [Gurobi](https://www.gurobi.com/)
    - [MOSEK](https://mosek.com/)
    - [OSQP](https://github.com/oxfordcontrol/osqp)

## Example

To solve a quadratic program, simply build the matrices that define it and call
the ``solve_qp`` function:

```python
from numpy import array, dot
from qpsolvers import solve_qp

M = array([[1., 2., 0.], [-8., 3., 2.], [0., 1., 1.]])
P = dot(M.T, M)  # quick way to build a symmetric matrix
q = dot(array([3., 2., 3.]), M).reshape((3,))
G = array([[1., 2., 1.], [2., 0., 1.], [-1., 2., -1.]])
h = array([3., 2., -2.]).reshape((3,))
A = array([1., 1., 1.])
b = array([1.])

x = solve_qp(P, q, G, h, A, b)
print("QP solution: x = {}".format(x))
```

This example outputs the solution ``[0.30769231, -0.69230769,  1.38461538]``.

## Performances

On a [dense problem](examples/dense_problem.py), the performance of all solvers
(as measured by IPython's ``%timeit`` on my machine) is:

| Solver   | Type   | Time (ms) |
| -------- | ------ | --------- |
| quadprog | Dense  | 0.02      |
| qpoases  | Dense  | 0.03      |
| osqp     | Sparse | 0.04      |
| ecos     | Sparse | 0.34      |
| cvxopt   | Dense  | 0.46      |
| gurobi   | Sparse | 0.84      |
| cvxpy    | Sparse | 3.40      |
| mosek    | Sparse | 7.17      |

On a [sparse problem](examples/sparse_problem.py), these performances become:

| Solver   | Type   | Time (ms) |
| -------- | ------ | --------- |
| osqp     | Sparse |    1      |
| mosek    | Sparse |   17      |
| ecos     | Sparse |   21      |
| cvxopt   | Dense  |  186      |
| gurobi   | Sparse |  221      |
| quadprog | Dense  |  550      |
| cvxpy    | Sparse |  654      |
| qpoases  | Dense  | 2250      |

Finally, here are the results on a benchmark of [random
problems](examples/random_problems.py) (each data point corresponds to an
average over 10 runs):

<img src="https://scaron.info/images/qp-benchmark.png">

Note that performances of QP solvers largely depend on the problem solved. For
instance, MOSEK performs an [automatic conversion to Second-Order Cone
Programming
(SOCP)](https://docs.mosek.com/8.1/pythonapi/prob-def-quadratic.html) which the
documentation advises bypassing for better performance. Similarly, ECOS
reformulates [from QP to SOCP](qpsolvers/ecos_.py) and [works best on small
problems](https://web.stanford.edu/%7Eboyd/papers/ecos.html).
