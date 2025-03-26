.. _opt_workflow:

===================================
Basic Workflow: Models and Sampling
===================================

This section provides a high-level description of how you solve problems using
**hybrid** solvers. (For solving problems directly on a
:term:`QPU`, see :ref:`this <qpu_workflow>` section instead.)

.. include:: ../shared/workflow.rst
    :start-after: start_workflow_intro
    :end-before: end_workflow_intro

.. _opt_workflow_objective_functions:

Objective Functions
===================

.. include:: ../shared/workflow.rst
    :start-after: start_objective
    :end-before: end_objective

.. _opt_workflow_simple_obj_example:

Simple Objective Example
------------------------

.. include:: ../shared/workflow.rst
    :start-after: start_simple_objective_example
    :end-before: end_simple_objective_example

The :ref:`opt_simple_sampling_example` example below shows an equally simple
solution by sampling.

.. _opt_workflow_models:

Supported Models
----------------

To express your problem as an objective function and submit to a |dwave_short|
:term:`hybrid` sampler for solution, you typically use one of the
:ref:`Ocean software <index_ocean_sdk>`
:term:`quadratic models <quadratic model>` or :term:`nonlinear model` supported
by the `Leap <https://cloud.dwavesys.com/leap/>`_ service's hybrid solvers:

*   .. include:: ../shared/models.rst
        :start-after: start_models_nonlinear
        :end-before: end_models_nonlinear

*   .. include:: ../shared/models.rst
        :start-after: start_models_cqm
        :end-before: end_models_cqm

*   .. include:: ../shared/models.rst
        :start-after: start_models_bqm
        :end-before: end_models_bqm

*   .. include:: ../shared/models.rst
        :start-after: start_models_dqm
        :end-before: end_models_dqm

.. _opt_workflow_samplers:

Samplers
========

.. include:: ../shared/workflow.rst
    :start-after: start_samplers
    :end-before: end_samplers

.. _opt_simple_sampling_example:

Simple Sampling Example
-----------------------

.. include:: ../shared/workflow.rst
    :start-after: start_simple_sampler_example
    :end-before: end_simple_sampler_example

.. _opt_workflow_simple_example:

Simple Workflow Example
=======================

This example uses :ref:`Ocean software <index_ocean_sdk>` tools to
demonstrate the solution workflow described in this section on a simple problem
of finding the rectangle with the greatest area when the perimeter is limited.

In this example, the perimeter of the rectangle is set to 8 (meaning the
largest area is for the :math:`2X2` square).

A CQM is created that will have two integer variables, :math:`i, j`, each
limited to half the maximum perimeter length of 8, to represent the lengths of
the rectangle's sides:

>>> from dimod import ConstrainedQuadraticModel, Integer
...
>>> i = Integer('i', upper_bound=4)
>>> j = Integer('j', upper_bound=4)
>>> cqm = ConstrainedQuadraticModel()

The area of the rectangle is given by the multiplication of side :math:`i` by
side :math:`j`. The goal is to maximize the area, :math:`i*j`. Because
|dwave_short| samplers minimize, the objective should have its lowest value when
this goal is met. Objective :math:`-i*j` has its minimum value when :math:`i*j`,
the area, is greatest:

>>> cqm.set_objective(-i*j)

Finally, the requirement that the sum of both sides must not exceed the
perimeter is represented as constraint :math:`2i + 2j <= 8`:

>>> cqm.add_constraint(2*i+2*j <= 8, "Max perimeter")
'Max perimeter'

Instantiate a hybrid CQM sampler and submit the problem for solution by a remote
solver provided by the Leap quantum cloud service:

>>> from dwave.system import LeapHybridCQMSampler
...
>>> sampler = LeapHybridCQMSampler()                # doctest: +SKIP
>>> sampleset = sampler.sample_cqm(cqm)             # doctest: +SKIP
>>> print(sampleset.first)                          # doctest: +SKIP
Sample(sample={'i': 2.0, 'j': 2.0}, energy=-4.0, num_occurrences=1,
...            is_feasible=True, is_satisfied=array([ True]))

The best (lowest-energy) solution found has :math:`i=j=2` as expected, a
solution that is feasible because all the constraints (one in this example)
are satisfied.

