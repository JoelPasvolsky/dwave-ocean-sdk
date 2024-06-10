.. _example_nl_cvrp:

===============
Vehicle Routing
===============

This example demonstrates the use of a `Leap <https://cloud.dwavesys.com/leap/>`_
hybrid solver on a :ref:`nonlinear-model <nl_model_sdk>`. It explains how to 
estimate the minimum time the solver requires for the model, check that solutions 
meet the model's constraints, set initial states, and more. For a simpler example
usage of the solver, start with the :ref:`example_nl_tsp` example; for information 
on formulating problems as nonlinear models, see the 
:ref:`dwave-optimization <intro_optimization>` package..

The goal of the well-known capacitated vehicle routing problem, 
`CVRP <https://en.wikipedia.org/wiki/Vehicle_routing_problem>`_, is to find 
the shortest possible routes for a fleet of vehicles delivering to multiple 
customer locations from a central depot. Vehicles have a specified delivery 
capacity, and on the routes to locations and then back to the depot, no vehicle 
is allowed to exceed its carrying capacity. 

Example Requirements
====================

.. include:: hybrid_solver_service.rst
	  :start-after: example-requirements-start-marker
	  :end-before: example-requirements-end-marker

Solution Steps
==============

.. include:: hybrid_solver_service.rst
  :start-after: example-steps-start-marker
  :end-before: example-steps-end-marker

This example formulates this problem as a :ref:`nonlinear model <nl_model_sdk>`
and uses the :class:`~dwave.system.samplers.LeapHybridNLSampler` to find good
solutions.

Formulate the Problem
=====================

First, define customer demand and locations. A standard format
for signaling the location of the depot, used by libraries such 
as `CVRPLIB <http://vrp.atd-lab.inf.puc-rio.br/index.php/en/>`_, 
is to set the demand of the first location as zero.

>>> demand = [0, 34, 12, 65, 10, 43, 27, 55, 61, 22]
>>> sites = [(15, 38), (23, -19), (44, 62), (3, 12), (-56, -21), (-53, 2), 
...          (33, 63), (14, -33), (42, 41), (13, -62)]

Here there are ten locations, with the depot being located at coordinates
:math:`(15, 38)`.

This example uses one of Ocean software's model generators to instantiate a 
:class:`~dwave.optimization.model.Model` class for a CVRP. 
The :class:`~dwave.optimization.model.Model` class encodes all the information 
(:term:`objective function`, constraints, constants, and decision variables) 
relevant to your models. 

>>> from dwave.optimization.generators import capacitated_vehicle_routing
>>> model = capacitated_vehicle_routing(
...     demand=demand,
...     number_of_vehicles=2,
...     vehicle_capacity=200,
...     locations_x=[x for x,y in sites],
...     locations_y=[y for x,y in sites])

For detailed information on how the CVRP is modelled, 
see the documentation for the 
:class:`~dwave.optimization.generators.capacitated_vehicle_routing` 
generator. 

Solve the Problem by Sampling
=============================

D-Wave's quantum cloud service provides cloud-based
:std:doc:`hybrid solvers <sysdocs_gettingstarted:doc_leap_hybrid>` you can
submit quadratic and nonlinear models to. These solvers, which implement 
state-of-the-art classical algorithms together with intelligent allocation 
of the quantum processing unit (QPU) to parts of the problem where it benefits 
most, are designed to accommodate even very large problems. Leap's solvers can
relieve you of the burden of any current and future development and optimization
of hybrid algorithms that best solve your problem.

Ocean software's :doc:`dwave-system </docs_system/sdk_index>`
:class:`~dwave.system.samplers.LeapHybridNLSampler` class enables you to
easily incorporate Leap's hybrid nonlinear-model solvers into your application:

>>> from dwave.system import LeapHybridNLSampler
>>> sampler = LeapHybridNLSampler()                  # doctest: +SKIP

Check the minimum required solution time estimated for the model. You can choose
to set your own :ref:`time_limit <sysdocs_gettingstarted:param_time_limit>`, which 
can be higher or lower:

Check the minimum required solution time estimated for the model. You can choose
to set your own :ref:`time_limit <sysdocs_gettingstarted:param_time_limit>`, which 
can be higher or lower:

*   Higher than the estimated time: allows the solver time to possibly find better
    solutions.
*   Lower than the estimated time: tries finding solutions quickly. Leap's hybrid 
    solvers are not guaranteed to complete processing in under the estimated 
    time limit, and you may be charged up to the estimated minimum required time.   

>>> print(sampler.estimated_min_time_limit(model))	# doctest: +SKIP
5

Submit the model to the selected solver. 

>>> results = sampler.sample(
...     model, 
...     time_limit=10)  	# doctest: +SKIP

You can check information such as timing in the returned results:

>>> print(results.result().timing['charge_time'])       # doctest: +SKIP
10000000

You can iterate through the returned samples. The code below shows up
to three solutions, printing the value of the objective function, the
itinerary for the fleet's two vehicles, and whether the solution meets 
the model's constraints on maximum capacity.

>>> num_samples = model.states.size()
>>> route, = model.iter_decisions()                     # doctest: +SKIP
>>> route1, route2 = route.iter_successors()            # doctest: +SKIP
>>> for i in range(min(3, num_samples)):
...     print(f"Objective value {int(model.objective.state(i))} for \n" \
...     f"\t Route 1: {route1.state(i)} \t Route 2: {route2.state(i)} \n" \
...     f"\t Feasible: {all(model.iter_constraints())}")   # doctest: +SKIP
Objective value 484 for 
	 Route 1: [4. 3. 7. 1. 5.] 	 Route 2: [4. 3. 7. 1. 5.]
	 Feasible: True
Objective value 423 for 
	 Route 1: [0. 6. 8. 3. 4.] 	 Route 2: [0. 6. 8. 3. 4.]
	 Feasible: True
Objective value 423 for 
	 Route 1: [2. 7. 1. 5.] 	 Route 2: [2. 7. 1. 5.]
	 Feasible: True

Providing an Initial State
--------------------------

For some problems you might have estimates or guesses of solutions, and 
by providing to the solver, as part of your problem submission, such 
assignments of decision variables as an initial state of the model, you may 
accelerate the solution.

Leap's hybrid nonlinear-model solver supports accepting an initial state
as part of the submitted model.


