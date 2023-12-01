.. _cqm_sdk:

============================
Constrained Quadratic Models
============================

The constrained quadratic model (CQM) are problems of the form:

.. math::

    \begin{align}
        \text{Minimize an objective:} & \\
        & \sum_{i} a_i x_i + \sum_{i \le j} b_{ij} x_i x_j + c, \\
        \text{Subject to constraints:} & \\
        & \sum_i a_i^{(m)} x_i + \sum_{i \le j} b_{ij}^{(m)} x_i x_j+ c^{(m)} \circ 0,
        \quad m=1, \dots, M,
    \end{align}

where :math:`\{ x_i\}_{i=1, \dots, N}` can be binary\ [#]_, integer, or 
continuous\ [#]_ variables, :math:`a_{i}, b_{ij}, c` are real values,
:math:`\circ \in \{ \ge, \le, = \}` and  :math:`M` is the total number of constraints.

.. [#]
    For binary variables, the range of the quadratic-term summation is
    :math:`i < j` because :math:`x^2 = x` for binary values :math:`\{0, 1\}`
    and :math:`s^2 = 1` for spin values :math:`\{-1, 1\}`.

.. [#] 
    Real-valued variables are currently not supported in quadratic interactions. 

CQM Basics 
==========

The :class:`dimod.ConstrainedQuadraticModel` class can contain this 
model and its methods provide convenient utilities for working with 
representations of a problem.

As a simple example to demonstrate commonly used function, consider 
a problem of finding the rectangle with the greatest area (the objective) 
for a given perimeter (a constraint). You can formulate the problem 
mathematically as,

.. math::

  \textrm{Objective: } \quad &\max_{i,j} \quad ij

  \textrm{Constraint:} \quad &2i+2j \le 8

where the components are,

*   **Variables**: :math:`i` and :math:`j` are the lengths of two sides of the
    rectangle.
*   **Objective**: maximize the area, which is given by the standard geometric
    formula :math:`ij`.
*   **Constraint**: subject any acceptable ("feasible") solutions to 
    the restriction of not exceeding the given perimeter length, 
    arbitrarily selected here to be 8; that is, the summation of the 
    rectangle's four sides, :math:`2i+2j`, is constrained to a maximum 
    value of 8.

Representing the Problem as a CQM 
---------------------------------

For small and experimental models, it's convenient to start coding 
with :ref:`symbolic math <intro_symbolic_math>`; later, for production
models, you typically :ref:`scale up <intro_scaling>` using higher 
performing code. 

>>> import dimod 
>>> i, j = dimod.Integers(["i", "j"])
>>> objective = -i * j
>>> constraint = 2 * i + 2 * j <= 8
>>> print(objective.to_polystring())
-i*j
>>> print(constraint.to_polystring())
2*i + 2*j <= 8

Here variables :code:`i,j` are of type integer, perhaps representing the number
of tiles laid horizontally and vertically in creating a rectangular floor, and
the coded ``objective`` is set to negative because D-Wave samplers minimize
rather than maximize.

Create a CQM, set the objective and add the constraint.

>>> cqm = dimod.ConstrainedQuadraticModel()
>>> cqm.set_objective(objective)
>>> c1 = cqm.add_constraint(constraint, label="max perimeter length")

Viewing and Updating the CQM 
----------------------------

The simplest way to view your CQM is to print it.

>>> print(cqm)
Constrained quadratic model: 2 variables, 1 constraints, 5 biases
<BLANKLINE>
Objective
  -Integer('i')*Integer('j')
<BLANKLINE>
Constraints
  max perimeter length: 2*Integer('i') + 2*Integer('j') <= 8.0
<BLANKLINE>
Bounds
  0.0 <= Integer('i') <= ...
  0.0 <= Integer('j') <= ...

For bigger models, it can be convenient to view just parts of the 
model. 

>>> print(cqm.objective.to_polystring())
-i*j
>>> print(cqm.constraints[c1].to_polystring())
2*i + 2*j <= 8.0

You can see size-related parameters, such as the number of biases.

>>> cqm.num_biases()
5

When possible, always set boundaries on non-binary variables to reduce
the search space. In this example, where the perimeter cannot exceed 8,
the sides are upper bound to 8 too.

>>> cqm.set_upper_bound('i', 8)
>>> cqm.set_upper_bound('j', 8)

Solutions  
---------

For *very* small problems, you can brute force a solution. The 
:class:`~dimod.reference.samplers.ExactCQMSolver` finds the CQM energy
for all possible assignments of the variables.  

>>> sampleset = dimod.ExactCQMSolver().sample_cqm(cqm)

For constrained problems, good solutions must be both feasible (meet 
all constraints) and minimize the objective (lowest energy).

>>> feasible_sampleset = sampleset.filter(lambda d: d.is_feasible)
>>> print(feasible_sampleset.first.sample)
{'i': 2, 'j': 2}

A solution such as ``{'i': 1, 'j': 2}`` is not optimal while a solution 
such as ``{'i': 8, 'j': 8}`` is infeasible.

>>> cqm.objective.energy({'i': 2, 'j': 2})
-4.0
>>> cqm.objective.energy({'i': 1, 'j': 2})
-2.0
>>> cqm.violations({'i': 8, "j": 8})
{'max perimeter length': 24.0}



Hard and Soft Constraints 
=========================

Constraints can be categorized as either "hard" or "soft". Any hard constraint
must be satisfied for a solution of the model to qualify as feasible. Soft
constraints may be violated to achieve an overall good solution. By setting
appropriate weights to soft constraints in comparison to the objective
and to other soft constraints, you can express the relative importance of such
constraints.


