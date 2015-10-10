.. Automatically generated Sphinx-extended reStructuredText file from DocOnce source
   (https://github.com/hplgit/doconce/)

.. Document title:

Analysis of exponential decay models
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

:Authors: Hans Petter Langtangen
:Date: Oct 10, 2015

.. !split

We address the ODE for exponential decay,

.. math::
   :label: _auto1
        
        u'(t) = -au(t),\quad u(0)=I,
        
        

where :math:`a` and :math:`I` are given constants. This problem is solved
by the :math:`\theta`-rule finite difference scheme, resulting in
the recursive equations

.. math::
   :label: decay:analysis:scheme
        
        u^{n+1} = \frac{1 - (1-\theta) a\Delta t}{1 + \theta a\Delta t}u^n
        
        

for the numerical solution :math:`u^{n+1}`, which approximates the exact
solution :math:`{u_{\small\mbox{e}}}` at time point :math:`t_{n+1}`. For constant mesh spacing,
which we assume here, :math:`t_{n+1}=(n+1)\Delta t`.

The example programs associated with this chapter are found in
the directory `src/analysis <http://tinyurl.com/ofkw6kc/analysis>`__.

Experimental investigations
===========================

We first perform a series of numerical explorations to see how the
methods behave as we change the parameters :math:`I`, :math:`a`, and :math:`\Delta t`
in the problem.

Discouraging numerical solutions
--------------------------------

Choosing :math:`I=1`, :math:`a=2`, and running experiments with :math:`\theta =1,0.5, 0`
for :math:`\Delta t=1.25, 0.75, 0.5, 0.1`, gives the results in
Figures :ref:`decay:analysis:BE4c`, :ref:`decay:analysis:CN4c`, and
:ref:`decay:analysis:FE4c`.

.. _decay:analysis:BE4c:

.. figure:: fig-analysis/BE4c.png
   :width: 600

   *Backward Euler*

.. _decay:analysis:CN4c:

.. figure:: fig-analysis/CN4c.png
   :width: 600

   *Crank-Nicolson*

.. _decay:analysis:FE4c:

.. figure:: fig-analysis/FE4c.png
   :width: 600

   *Forward Euler*

The characteristics of the displayed curves can be summarized as follows:

  * The Backward Euler scheme gives a monotone solution in all cases,
    lying above the exact curve.

  * The Crank-Nicolson scheme gives the most accurate results, but for
    :math:`\Delta t=1.25` the solution oscillates.

  * The Forward Euler scheme gives a growing, oscillating solution for
    :math:`\Delta t=1.25`; a decaying, oscillating solution for :math:`\Delta t=0.75`;
    a strange solution :math:`u^n=0` for :math:`n\geq 1` when :math:`\Delta t=0.5`; and
    a solution seemingly as accurate as the one by the Backward Euler
    scheme for :math:`\Delta t = 0.1`, but the curve lies below the exact
    solution.

Since the exact solution of our model problem is a monotone function,
:math:`u(t)=Ie^{-at}`, some of these qualitatively wrong results indeed seem alarming!


.. admonition:: Key questions

   
    * Under what circumstances, i.e., values of
      the input data :math:`I`, :math:`a`, and :math:`\Delta t` will the Forward Euler and
      Crank-Nicolson schemes result in undesired oscillatory solutions?
   
    * How does :math:`\Delta t` impact the error in the numerical solution?
   
   The first question will be investigated both by numerical experiments and
   by precise mathematical theory. The theory will help establish
   general criteria on :math:`\Delta t` for avoiding non-physical oscillatory
   or growing solutions.
   
   For our simple model problem we can answer the second
   question very precisely, but
   we will also look at simplified formulas for small :math:`\Delta t`
   and touch upon important concepts such as *convergence rate* and
   *the order of a scheme*. Other fundamental concepts mentioned are
   stability, consistency, and convergence.




Detailed experiments
--------------------

To address the first question above,
we may set up an experiment where we loop over values of :math:`I`, :math:`a`,
and :math:`\Delta t` in our chosen model problem.
For each experiment, we flag the solution as
oscillatory if

.. math::
         u^{n} > u^{n-1},

for some value of :math:`n`. This seems like a reasonable choice,
since we expect :math:`u^n` to decay with :math:`n`, but oscillations will make
:math:`u` increase over a time step. Doing some initial experimentation
with varying :math:`I`, :math:`a`, and :math:`\Delta t`, quickly reveals that
oscillations are independent of :math:`I`, but they do depend on :math:`a` and
:math:`\Delta t`. We can therefore limit the investigation to
vary :math:`a` and :math:`\Delta t`. Based on this observation,
we introduce a two-dimensional
function :math:`B(a,\Delta t)` which is 1 if oscillations occur
and 0 otherwise. We can visualize :math:`B` as a contour plot
(lines for which :math:`B=\hbox{const}`). The contour :math:`B=0.5`
corresponds to the borderline between oscillatory regions with :math:`B=1`
and monotone regions with :math:`B=0` in the :math:`a,\Delta t` plane.

The :math:`B` function is defined at discrete :math:`a` and :math:`\Delta t` values.
Say we have given :math:`P` values for :math:`a`, :math:`a_0,\ldots,a_{P-1}`, and
:math:`Q` values for :math:`\Delta t`, :math:`\Delta t_0,\ldots,\Delta t_{Q-1}`.
These :math:`a_i` and :math:`\Delta t_j` values, :math:`i=0,\ldots,P-1`,
:math:`j=0,\ldots,Q-1`, form a rectangular mesh of :math:`P\times Q` points
in the plane spanned by :math:`a` and :math:`\Delta t`.
At each point :math:`(a_i, \Delta t_j)`, we associate
the corresponding value :math:`B(a_i,\Delta t_j)`, denoted :math:`B_{ij}`.
The :math:`B_{ij}` values are naturally stored in a two-dimensional
array. We can thereafter create a plot of the
contour line :math:`B_{ij}=0.5` dividing the oscillatory and monotone
regions. The file `decay_osc_regions.py <http://tinyurl.com/ofkw6kc/analysis/decay_osc_regions.py>`__  given below (``osc_regions`` stands for "oscillatory regions") contains all nuts and
bolts to produce the :math:`B=0.5` line in Figures :ref:`decay:analysis:B:FE`
and :ref:`decay:analysis:B:CN`. The oscillatory region is above this line.

.. code-block:: python

        from decay_mod import solver
        import numpy as np
        import scitools.std as st
        
        def non_physical_behavior(I, a, T, dt, theta):
            """
            Given lists/arrays a and dt, and numbers I, dt, and theta,
            make a two-dimensional contour line B=0.5, where B=1>0.5
            means oscillatory (unstable) solution, and B=0<0.5 means
            monotone solution of u'=-au.
            """
            a = np.asarray(a); dt = np.asarray(dt)  # must be arrays
            B = np.zeros((len(a), len(dt)))         # results
            for i in range(len(a)):
                for j in range(len(dt)):
                    u, t = solver(I, a[i], T, dt[j], theta)
                    # Does u have the right monotone decay properties?
                    correct_qualitative_behavior = True
                    for n in range(1, len(u)):
                        if u[n] > u[n-1]:  # Not decaying?
                            correct_qualitative_behavior = False
                            break  # Jump out of loop
                    B[i,j] = float(correct_qualitative_behavior)
            a_, dt_ = st.ndgrid(a, dt)  # make mesh of a and dt values
            st.contour(a_, dt_, B, 1)
            st.grid('on')
            st.title('theta=%g' % theta)
            st.xlabel('a'); st.ylabel('dt')
            st.savefig('osc_region_theta_%s.png' % theta)
            st.savefig('osc_region_theta_%s.pdf' % theta)
        
        non_physical_behavior(
            I=1,
            a=np.linspace(0.01, 4, 22),
            dt=np.linspace(0.01, 4, 22),
            T=6,
            theta=0.5)

.. _decay:analysis:B:FE:

.. figure:: fig-analysis/osc_region_FE.png
   :width: 500

   *Forward Euler scheme: oscillatory solutions occur for points above the curve*

.. _decay:analysis:B:CN:

.. figure:: fig-analysis/osc_region_CN.png
   :width: 500

   *Crank-Nicolson scheme: oscillatory solutions occur for points above the curve*

By looking at the curves in the figures one may guess that :math:`a\Delta t`
must be less than a critical limit to avoid the undesired
oscillations.  This limit seems to be about 2 for Crank-Nicolson and 1
for Forward Euler.  We shall now establish a precise mathematical
analysis of the discrete model that can explain the observations in
our numerical experiments.

Stability
=========

The goal now is to understand the results in the previous section.
To this end, we shall investigate the properties of the mathematical
formula for the solution of the equations arising from the finite
difference methods.

Exact numerical solution
------------------------

Starting with :math:`u^0=I`, the simple recursion :eq:`decay:analysis:scheme`
can be applied repeatedly :math:`n` times, with the result that

.. math::
   :label: decay:analysis:unex
        
        u^{n} = IA^n,\quad A = \frac{1 - (1-\theta) a\Delta t}{1 + \theta a\Delta t}{\thinspace .}
        
        


.. admonition:: Solving difference equations

   Difference equations where all terms are linear in
   :math:`u^{n+1}`, :math:`u^n`, and maybe :math:`u^{n-1}`, :math:`u^{n-2}`, etc., are
   called *homogeneous, linear* difference equations, and their solutions
   are generally of the form :math:`u^n=A^n`, where :math:`A` is a constant to be
   determined. Inserting this expression in the difference equation
   and dividing by :math:`A^{n+1}` gives
   a polynomial equation in :math:`A`. In the present case we get
   
   .. math::
            A = \frac{1 - (1-\theta) a\Delta t}{1 + \theta a\Delta t}{\thinspace .} 
   
   This is a solution technique of wider applicability than repeated use of
   the recursion :eq:`decay:analysis:scheme`.




Regardless of the solution approach, we have obtained a formula for
:math:`u^n`.  This formula can explain everything we see in the figures
above, but it also gives us a more general insight into accuracy and
stability properties of the three schemes.

.. index:: stability

Since :math:`u^n` is a factor :math:`A`
raised to an integer power :math:`n`, we realize that :math:`A < 0`
will imply :math:`u^n < 0` for odd :math:`n` and :math:`u^n > 0` for even :math:`n`.
That is, the solution oscillates between the mesh points.
We have oscillations due to :math:`A < 0` when

.. math::
   :label: decay:th:stability
        
        (1-\theta)a\Delta t > 1 {\thinspace .}
        
        

Since :math:`A>0` is a requirement for having a numerical solution with the
same basic property (monotonicity) as the exact solution, we may say
that :math:`A>0` is a *stability criterion*. Expressed in terms of :math:`\Delta t`
the stability criterion reads

.. math::
   :label: _auto2
        
        \Delta t < \frac{1}{(1-\theta)a}{\thinspace .}
        
        

The Backward
Euler scheme is always stable since :math:`A < 0` is impossible for :math:`\theta=1`, while
non-oscillating solutions for Forward Euler and Crank-Nicolson
demand :math:`\Delta t\leq 1/a` and :math:`\Delta t\leq 2/a`, respectively.
The relation between :math:`\Delta t` and :math:`a` look reasonable: a larger
:math:`a` means faster decay and hence a need for smaller time steps.

Looking at the upper left plot in Figure :ref:`decay:analysis:FE4c`,
we see that :math:`\Delta t=1.25`, and remembering that :math:`a=2` in these
experiments, :math:`A` can be calculated to be
:math:`-1.5`, so the Forward Euler solution becomes :math:`u^n=(-1.5)^n` (:math:`I=1`).
This solution oscillates *and* grows. The upper right plot has
:math:`a\Delta t = 2\cdot 0.75=1.5`, so :math:`A=-0.5`,
and :math:`u^n=(-0.5)^n` decays but oscillates. The lower left plot
is a peculiar case where the Forward Euler scheme produces a solution
that is stuck on the :math:`t` axis. Now we can understand why this is so,
because :math:`a\Delta t= 2\cdot 0.5=1`, which gives :math:`A=0`,
and therefore :math:`u^n=0` for :math:`n\geq 1`.  The decaying oscillations in the Crank-Nicolson scheme in the upper left plot in Figure :ref:`decay:analysis:CN4c`
for :math:`\Delta t=1.25` are easily explained by the fact that :math:`A\approx -0.11 < 0`.

Stability properties derived from the amplification factor
----------------------------------------------------------

.. index:: amplification factor

The factor :math:`A` is called the *amplification factor* since the solution
at a new time level is the solution at the previous time
level amplified by a factor :math:`A`.
For a decay process, we must obviously have :math:`|A|\leq 1`, which
is fulfilled for all :math:`\Delta t` if :math:`\theta \geq 1/2`. Arbitrarily
large values of :math:`u` can be generated when :math:`|A|>1` and :math:`n` is large
enough. The numerical solution is in such cases totally irrelevant to
an ODE modeling decay processes! To avoid this situation, we must
demand :math:`|A|\leq 1` also for :math:`\theta < 1/2`, which implies

.. math::
   :label: _auto3
        
        \Delta t \leq \frac{2}{(1-2\theta)a},
        
        

For example, :math:`\Delta t` must not exceed  :math:`2/a` when computing with
the Forward Euler scheme.

.. index:: A-stable methods

.. index:: L-stable methods


.. admonition:: Stability properties

   We may summarize the stability investigations as follows:
   
   1. The Forward Euler method is a *conditionally stable* scheme because
      it requires :math:`\Delta t < 2/a` for avoiding growing solutions
      and :math:`\Delta t < 1/a` for avoiding oscillatory solutions.
   
   2. The Crank-Nicolson is *unconditionally stable* with respect to
      growing solutions, while it is conditionally stable with
      the criterion :math:`\Delta t < 2/a` for avoiding oscillatory solutions.
   
   3. The Backward Euler method is unconditionally stable with respect
      to growing and oscillatory solutions - any :math:`\Delta t` will work.
   
   Much literature on ODEs speaks about L-stable and A-stable methods.
   In our case A-stable methods ensures non-growing solutions, while
   L-stable methods also avoids oscillatory solutions.




Accuracy
========

While stability concerns the qualitative properties of the numerical
solution, it remains to investigate the quantitative properties to
see exactly how large the numerical errors are.

Visual comparison of amplification factors
------------------------------------------

After establishing how :math:`A` impacts the qualitative features of the
solution, we shall now look more into how well the numerical amplification
factor approximates the exact one. The exact solution reads
:math:`u(t)=Ie^{-at}`, which can be rewritten as

.. math::
   :label: _auto4
        
        {{u_{\small\mbox{e}}}}(t_n) = Ie^{-a n\Delta t} = I(e^{-a\Delta t})^n {\thinspace .}
        
        

From this formula we see that the exact amplification factor is

.. math::
   :label: _auto5
        
        {A_{\small\mbox{e}}} = e^{-a\Delta t} {\thinspace .}
        
        

We see from all of our analysis
that the exact and numerical amplification factors depend
on :math:`a` and :math:`\Delta t` through the dimensionless
product :math:`a\Delta t`: whenever there is a
:math:`\Delta t` in the analysis, there is always an associated :math:`a`
parameter. Therefore, it
is convenient to introduce a symbol for this product, :math:`p=a\Delta t`,
and view :math:`A` and :math:`{A_{\small\mbox{e}}}` as functions of :math:`p`. Figure
:ref:`decay:analysis:fig:A` shows these functions. The two amplification
factors are clearly closest for the
Crank-Nicolson method, but that method has
the unfortunate oscillatory behavior when :math:`p>2`.

.. _decay:analysis:fig:A:

.. figure:: fig-analysis/A_factors.png
   :width: 500

   *Comparison of amplification factors*


.. admonition:: Significance of the :math:`p=a\Delta t` parameter

   The key parameter for numerical performance of a scheme is in this model
   problem :math:`p=a\Delta t`. This is a *dimensionless number* (:math:`a` has dimension
   1/s and :math:`\Delta t` has dimension s) reflecting how the discretization
   parameter plays together with a physical parameter in the problem.
   
   One can bring the present model problem on dimensionless form
   through a process called scaling. The scaled modeled has a modified
   time :math:`\bar t = at` and modified response :math:`\bar u =u/I` such that
   the model reads :math:`d\bar u/d\bar t = -\bar u`, :math:`\bar u(0)=1`.
   Analyzing this model, where there are no physical parameters,
   we find that :math:`\Delta \bar t` is the key parameter
   for numerical performance. In the unscaled model,
   this corresponds to :math:`\Delta \bar t = a\Delta t`.
   
   It is common that the numerical performance of methods for solving ordinary and
   partial differential equations is governed by dimensionless parameters
   that combine mesh sizes with physical parameters.




Series expansion of amplification factors
-----------------------------------------

As an alternative to the visual understanding inherent in Figure
:ref:`decay:analysis:fig:A`, there is a strong tradition in numerical
analysis to establish formulas for approximation errors when the
discretization parameter, here :math:`\Delta t`, becomes small. In the
present case, we let :math:`p` be our small discretization parameter, and it
makes sense to simplify the expressions for :math:`A` and :math:`{A_{\small\mbox{e}}}` by using
Taylor polynomials around :math:`p=0`.  The Taylor polynomials are accurate
for small :math:`p` and greatly simplify the comparison of the analytical
expressions since we then can compare polynomials, term by term.

Calculating the Taylor series for :math:`{A_{\small\mbox{e}}}` is easily done by hand, but
the three versions of :math:`A` for :math:`\theta=0,1,{\frac{1}{2}}` lead to more
cumbersome calculations.
Nowadays, analytical computations can benefit greatly by
symbolic computer algebra software. The Python package ``sympy``
represents a powerful computer algebra system, not yet as sophisticated as
the famous Maple and Mathematica systems, but it is free and
very easy to integrate with our numerical computations in Python.

.. index:: interactive Python

.. index:: isympy

.. index:: sympy

When using ``sympy``, it is convenient to enter an interactive Python
shell where the results of expressions and statements can be shown
immediately.
Here is a simple example. We strongly recommend to use
``isympy`` (or ``ipython``) for such interactive sessions.

Let us illustrate ``sympy`` with a standard Python shell syntax
(``>>>`` prompt) to compute a Taylor polynomial approximation to :math:`e^{-p}`:

.. code-block:: python

        >>> from sympy import *
        >>> # Create p as a mathematical symbol with name 'p'
        >>> p = Symbol('p')
        >>> # Create a mathematical expression with p
        >>> A_e = exp(-p)
        >>>
        >>> # Find the first 6 terms of the Taylor series of A_e
        >>> A_e.series(p, 0, 6)
        1 + (1/2)*p**2 - p - 1/6*p**3 - 1/120*p**5 + (1/24)*p**4 + O(p**6)

Lines with ``>>>`` represent input lines, whereas without
this prompt represent the result of the previous command (note that
``isympy`` and ``ipython`` apply other prompts, but in this text
we always apply ``>>>`` for interactive Python computing).
Apart from the order of the powers, the computed formula is easily
recognized as the beginning of the Taylor series for :math:`e^{-p}`.

Let us define the numerical amplification factor where :math:`p` and :math:`\theta`
enter the formula as symbols:

.. code-block:: python

        >>> theta = Symbol('theta')
        >>> A = (1-(1-theta)*p)/(1+theta*p)

To work with the factor for the Backward Euler scheme we
can substitute the value 1 for ``theta``:

.. code-block:: python

        >>> A.subs(theta, 1)
        1/(1 + p)

Similarly, we can substitute ``theta`` by 1/2 for Crank-Nicolson,
preferably using an exact rational representation of 1/2 in ``sympy``:

.. code-block:: python

        >>> half = Rational(1,2)
        >>> A.subs(theta, half)
        1/(1 + (1/2)*p)*(1 - 1/2*p)

The Taylor series of the amplification factor for the Crank-Nicolson
scheme can be computed as

.. code-block:: python

        >>> A.subs(theta, half).series(p, 0, 4)
        1 + (1/2)*p**2 - p - 1/4*p**3 + O(p**4)

We are now in a position to compare Taylor series:

.. code-block:: python

        >>> FE = A_e.series(p, 0, 4) - A.subs(theta, 0).series(p, 0, 4)
        >>> BE = A_e.series(p, 0, 4) - A.subs(theta, 1).series(p, 0, 4)
        >>> CN = A_e.series(p, 0, 4) - A.subs(theta, half).series(p, 0, 4 )
        >>> FE
        (1/2)*p**2 - 1/6*p**3 + O(p**4)
        >>> BE
        -1/2*p**2 + (5/6)*p**3 + O(p**4)
        >>> CN
        (1/12)*p**3 + O(p**4)

From these expressions we see that the error :math:`A-{A_{\small\mbox{e}}}\sim {\mathcal{O}(p^2)}`
for the Forward and Backward Euler schemes, while
:math:`A-{A_{\small\mbox{e}}}\sim {\mathcal{O}(p^3)}` for the Crank-Nicolson scheme.
The notation :math:`{\mathcal{O}(p^m)}` here means a polynomial in :math:`p` where
:math:`p^m` is the term of lowest-degree, and consequently the term that
dominates the expression for :math:`p < 0`. We call this the
*leading order term*. As :math:`p\rightarrow 0`, the leading order term
clearly dominates over the higher-order terms (think of :math:`p=0.01`:
:math:`p` is a hundred times larger than :math:`p^2`).

Now, :math:`a` is a given parameter in the problem, while :math:`\Delta t` is
what we can vary. Not surprisingly, the error expressions are usually
written in terms :math:`\Delta t`. We then have

.. math::
   :label: _auto6
        
        A-{A_{\small\mbox{e}}} = \left\lbrace\begin{array}{ll}
        {\mathcal{O}(\Delta t^2)}, & \hbox{Forward and Backward Euler},\\ 
        {\mathcal{O}(\Delta t^3)}, & \hbox{Crank-Nicolson}
        \end{array}\right.
        
        

We say that the Crank-Nicolson scheme has an error in the amplification
factor of order :math:`\Delta t^3`, while the two other schemes are
of order :math:`\Delta t^2` in the same quantity.

What is the significance of the order expression? If we halve :math:`\Delta t`,
the error in amplification factor at a time level will be reduced
by a factor of 4 in the Forward and Backward Euler schemes, and by
a factor of 8 in the Crank-Nicolson scheme. That is, as we
reduce :math:`\Delta t` to obtain more accurate results, the Crank-Nicolson
scheme reduces the error more efficiently than the other schemes.

The ratio of numerical and exact amplification factors
------------------------------------------------------

.. index::
   single: error; amplification factor

An alternative comparison of the schemes is provided by looking at the
ratio :math:`A/{A_{\small\mbox{e}}}`, or the error :math:`1-A/{A_{\small\mbox{e}}}` in this ratio:

.. code-block:: python

        >>> FE = 1 - (A.subs(theta, 0)/A_e).series(p, 0, 4)
        >>> BE = 1 - (A.subs(theta, 1)/A_e).series(p, 0, 4)
        >>> CN = 1 - (A.subs(theta, half)/A_e).series(p, 0, 4)
        >>> FE
        (1/2)*p**2 + (1/3)*p**3 + O(p**4)
        >>> BE
        -1/2*p**2 + (1/3)*p**3 + O(p**4)
        >>> CN
        (1/12)*p**3 + O(p**4)

The leading-order terms have the same powers as
in the analysis of :math:`A-{A_{\small\mbox{e}}}`.

.. _decay:analysis:gobal:error:

The global error at a point
---------------------------

.. index::
   single: error; global

The error in the amplification factor reflects the error when
progressing from time level :math:`t_n` to :math:`t_{n-1}` only. That is,
we disregard the error already present in the solution at :math:`t_{n-1}`.
The real error at a point, however, depends on the error development
over all previous time steps. This error,
:math:`e^n = u^n-{u_{\small\mbox{e}}}(t_n)`, is known as the *global error*. We
may look at :math:`u^n` for some :math:`n` and Taylor expand the
mathematical expressions as functions of :math:`p=a\Delta t` to get a simple
expression for the global error (for small :math:`p`):

.. code-block:: python

        >>> n = Symbol('n')
        >>> u_e = exp(-p*n)
        >>> u_n = A**n
        >>> FE = u_e.series(p, 0, 4) - u_n.subs(theta, 0).series(p, 0, 4)
        >>> BE = u_e.series(p, 0, 4) - u_n.subs(theta, 1).series(p, 0, 4)
        >>> CN = u_e.series(p, 0, 4) - u_n.subs(theta, half).series(p, 0, 4)
        >>> FE
        (1/2)*n*p**2 - 1/2*n**2*p**3 + (1/3)*n*p**3 + O(p**4)
        >>> BE
        (1/2)*n**2*p**3 - 1/2*n*p**2 + (1/3)*n*p**3 + O(p**4)
        >>> CN
        (1/12)*n*p**3 + O(p**4)

Note that ``sympy`` does not sort the polynomial terms in the output,
so :math:`p^3` appears before :math:`p^2` in the output of ``BE``.

For a fixed time :math:`t`, the parameter :math:`n` in these expressions increases
as :math:`p\rightarrow 0` since :math:`t=n\Delta t =\mbox{const}` and hence
:math:`n` must increase like :math:`\Delta t^{-1}`. With :math:`n` substituted by
:math:`t/\Delta t` in
the leading-order error terms, these become

.. math::
   :label: decay:analysis:gobal:error:FE
        
        e^n = \frac{1}{2} n p^2 = {\frac{1}{2}}ta^2\Delta t, \hbox{Forward Euler}
        
        

.. math::
   :label: decay:analysis:gobal:error:BE
          
        e^n = -\frac{1}{2} n p^2 = -{\frac{1}{2}}ta^2\Delta t, \hbox{Backward Euler}
        
        

.. math::
   :label: decay:analysis:gobal:error:CN
          
        e^n = \frac{1}{12}np^3 = \frac{1}{12}ta^3\Delta t^2, \hbox{Crank-Nicolson}
        
        

The global error is therefore of
second order (in :math:`\Delta t`) for the Crank-Nicolson scheme and of
first order for the other two schemes.


.. admonition:: Convergence

   When the global error :math:`e^n\rightarrow 0` as :math:`\Delta t\rightarrow 0`,
   we say that the scheme is *convergent*. It means that the numerical
   solution approaches the exact solution as the mesh is refined, and
   this is a much desired property of a numerical method.




.. _decay:analysis:gobal:error_int:

Integrated error
----------------

The :math:`L^2` norm of the error can be computed by treating :math:`e^n` as a function
of :math:`t` in ``sympy`` and performing symbolic integration. For
the Forward Euler scheme we have

.. code-block:: python

        p, n, a, dt, t, T, theta = symbols('p n a dt t T 'theta')
        A = (1-(1-theta)*p)/(1+theta*p)
        u_e = exp(-p*n)
        u_n = A**n
        error = u_e.series(p, 0, 4) - u_n.subs(theta, 0).series(p, 0, 4)
        # Introduce t and dt instead of n and p
        error = error.subs('n', 't/dt').subs(p, 'a*dt')
        error = error.as_leading_term(dt) # study only the first term
        print error
        error_L2 = sqrt(integrate(error**2, (t, 0, T)))
        print error_L2

The output reads

.. code-block:: text

        sqrt(30)*sqrt(T**3*a**4*dt**2*(6*T**2*a**2 - 15*T*a + 10))/60

which means that the :math:`L^2` error behaves like :math:`a^2\Delta t`.

Strictly speaking, the numerical error is only defined at the
mesh points so it makes most sense to compute the
:math:`\ell^2` error

.. math::
         ||e^n||_{\ell^2} = \sqrt{\Delta t\sum_{n=0}^{N_t} ({{u_{\small\mbox{e}}}}(t_n) - u^n)^2}
        {\thinspace .} 

We have obtained an exact analytical expression for the error at
:math:`t=t_n`, but here we use the leading-order error term only since we
are mostly interested in how the error behaves as a polynomial in
:math:`\Delta t` or :math:`p`, and then the leading order term will dominate.  For
the Forward Euler scheme, :math:`{u_{\small\mbox{e}}}(t_n) - u^n \approx {\frac{1}{2}}np^2`, and
we have

.. math::
        
        ||e^n||_{\ell^2}^2 = \Delta t\sum_{n=0}^{N_t} \frac{1}{4}n^2p^4
        =\Delta t\frac{1}{4}p^4 \sum_{n=0}^{N_t} n^2{\thinspace .}
        

Now, :math:`\sum_{n=0}^{N_t} n^2\approx \frac{1}{3}N_t^3`. Using this approximation,
setting :math:`N_t =T/\Delta t`, and taking the square root gives the expression

.. math::
   :label: decay:analysis:gobal:error_int:FE
        
        ||e^n||_{\ell^2} = \frac{1}{2}\sqrt{\frac{T^3}{3}} a^2\Delta t{\thinspace .}
        
        

Calculations for the Backward Euler scheme are very similar and provide
the same result, while the Crank-Nicolson scheme leads to

.. math::
   :label: decay:analysis:gobal:error_int:CN
        
        ||e^n||_{\ell^2} = \frac{1}{12}\sqrt{\frac{T^3}{3}}a^3\Delta t^2{\thinspace .}
        
        


.. admonition:: Summary of errors

   Both the global point-wise errors :eq:`decay:analysis:gobal:error:FE`-:eq:`decay:analysis:gobal:error:CN`
   and their time-integrated versions :eq:`decay:analysis:gobal:error_int:FE` and :eq:`decay:analysis:gobal:error_int:CN` show that
   
    * the Crank-Nicolson scheme is of second order in :math:`\Delta t`, and
   
    * the Forward Euler and Backward Euler schemes are of first order in :math:`\Delta t`.




.. _decay:analysis:trunc:

Truncation error
----------------

The truncation error is a very frequently used error measure for
finite difference methods. It is defined as *the error
in the difference equation that arises when inserting the exact
solution*. Contrary to many other error measures, e.g., the
true error :math:`e^n={u_{\small\mbox{e}}}(t_n)-u^n`, the truncation error is a quantity that
is easily computable.

Let us illustrate the calculation of the truncation error
for the Forward Euler scheme.
We start with the difference equation on operator form,

.. math::
         \lbrack D_t^+ u = -au\rbrack^n,

which is the short form for

.. math::
         \frac{u^{n+1}-u^n}{\Delta t} = -au^n{\thinspace .}

The idea is to see how well the exact solution :math:`{u_{\small\mbox{e}}}(t)` fulfills
this equation. Since :math:`{u_{\small\mbox{e}}}(t)` in general will not obey the
discrete equation, we get an error in the discrete equation. This
error is called
a *residual*, denoted here by :math:`R^n`:

.. math::
   :label: decay:analysis:trunc:Req
        
        R^n = \frac{{u_{\small\mbox{e}}}(t_{n+1})-{u_{\small\mbox{e}}}(t_n)}{\Delta t} + a{u_{\small\mbox{e}}}(t_n)
        {\thinspace .}
        
        

The residual is defined at each mesh point and is therefore a mesh
function with a superscript :math:`n`.

The interesting feature of :math:`R^n` is to see how it
depends on the discretization parameter :math:`\Delta t`.
The tool for reaching
this goal is to Taylor expand :math:`{u_{\small\mbox{e}}}` around the point where the
difference equation is supposed to hold, here :math:`t=t_n`.
We have that

.. math::
         {u_{\small\mbox{e}}}(t_{n+1}) = {u_{\small\mbox{e}}}(t_n) + {u_{\small\mbox{e}}}'(t_n)\Delta t + \frac{1}{2}{u_{\small\mbox{e}}}''(t_n)
        \Delta t^2 + \cdots, 

which may be used to reformulate the fraction in
:eq:`decay:analysis:trunc:Req` so that

.. math::
         R^n = {u_{\small\mbox{e}}}'(t_n) + \frac{1}{2}{u_{\small\mbox{e}}}''(t_n)\Delta t + \ldots + a{u_{\small\mbox{e}}}(t_n){\thinspace .}

Now, :math:`{u_{\small\mbox{e}}}` fulfills the ODE :math:`{u_{\small\mbox{e}}}'=-a{u_{\small\mbox{e}}}`, which means that the first and last
term cancel and we have

.. math::
         R^n = \frac{1}{2}{u_{\small\mbox{e}}}''(t_n)\Delta t + {\mathcal{O}(\Delta t^2)}{\thinspace .} 

This :math:`R^n` is the *truncation error*, which for the Forward Euler is seen
to be of first order in :math:`\Delta t` as :math:`\Delta \rightarrow 0`.

The above procedure can be repeated for the Backward Euler and the
Crank-Nicolson schemes. We start with the scheme in operator notation,
write it out in detail, Taylor expand :math:`{u_{\small\mbox{e}}}` around the point :math:`\tilde t`
at which the difference equation is defined, collect terms that
correspond to the ODE (here :math:`{u_{\small\mbox{e}}}' + a{u_{\small\mbox{e}}}`), and identify the remaining
terms as the residual :math:`R`, which is the truncation error.
The Backward Euler scheme leads to

.. math::
         R^n \approx -\frac{1}{2}{u_{\small\mbox{e}}}''(t_n)\Delta t, 

while the Crank-Nicolson scheme gives

.. math::
         R^{n+\frac{1}{2}} \approx \frac{1}{24}{u_{\small\mbox{e}}}'''(t_{n+\frac{1}{2}})\Delta t^2,

when :math:`\Delta t\rightarrow 0`.

The *order* :math:`r` of a finite difference scheme is often defined through
the leading term :math:`\Delta t^r` in the truncation error. The above
expressions point out that the Forward and Backward Euler schemes are
of first order, while Crank-Nicolson is of second order.  We have
looked at other error measures in other sections, like the error in
amplification factor and the error :math:`e^n={u_{\small\mbox{e}}}(t_n)-u^n`, and expressed
these error measures in terms of :math:`\Delta t` to see the order of the
method. Normally, calculating the truncation error is more
straightforward than deriving the expressions for other error measures
and therefore the easiest way to establish the order of a scheme.

Consistency, stability, and convergence
---------------------------------------

.. index:: consistency

.. index:: stability

.. index:: convergence

Three fundamental concepts when solving differential equations by
numerical methods are consistency, stability, and convergence.  We
shall briefly touch upon these concepts below in the context of the present
model problem.

Consistency means that the error in the difference equation, measured
through the truncation error, goes to zero as :math:`\Delta t\rightarrow
0`. Since the truncation error tells how well the exact solution
fulfills the difference equation, and the exact solution fulfills the
differential equation, consistency ensures that the difference
equation approaches the differential equation in the limit. The
expressions for the truncation errors in the previous section are all
proportional to :math:`\Delta t` or :math:`\Delta t^2`, hence they vanish as
:math:`\Delta t\rightarrow 0`, and all the schemes are consistent.  Lack of
consistency implies that we actually solve some other differential
equation in the limit :math:`\Delta t\rightarrow 0` than we aim at.

Stability means that the numerical solution exhibits the same
qualitative properties as the exact solution. This is obviously a
feature we want the numerical solution to have. In the present
exponential decay model, the exact solution is monotone and
decaying. An increasing numerical solution is not in accordance with
the decaying nature of the exact solution and hence unstable. We can
also say that an oscillating numerical solution lacks the property of
monotonicity of the exact solution and is also unstable. We have seen
that the Backward Euler scheme always leads to monotone and decaying
solutions, regardless of :math:`\Delta t`, and is hence stable. The Forward
Euler scheme can lead to increasing solutions and oscillating
solutions if :math:`\Delta t` is too large and is therefore unstable unless
:math:`\Delta t` is sufficiently small.  The Crank-Nicolson can never lead
to increasing solutions and has no problem to fulfill that stability
property, but it can produce oscillating solutions and is unstable in
that sense, unless :math:`\Delta t` is sufficiently small.

Convergence implies that the global (true) error mesh function :math:`e^n =
{u_{\small\mbox{e}}}(t_n)-u^n\rightarrow 0` as :math:`\Delta t\rightarrow 0`. This is really
what we want: the numerical solution gets as close to the exact
solution as we request by having a sufficiently fine mesh.

Convergence is hard to establish theoretically, except in quite simple
problems like the present one. Stability and consistency are much
easier to calculate. A major breakthrough in the understanding of
numerical methods for differential equations came in 1956 when Lax and
Richtmeyer established equivalence between convergence on one hand and
consistency and stability on the other (the `Lax equivalence theorem <http://en.wikipedia.org/wiki/Lax_equivalence_theorem>`__).  In practice
it meant that one can first establish that a method is stable and
consistent, and then it is automatically convergent (which is much
harder to establish).  The result holds for linear problems only, and
in the world of nonlinear differential equations the relations between
consistency, stability, and convergence are much more complicated.

We have seen in the previous analysis that the Forward Euler,
Backward Euler, and Crank-Nicolson schemes are convergent (:math:`e^n\rightarrow 0`),
that they are consistent (:math:`R^n\rightarrow 0`), and that they are
stable under certain conditions on the size of :math:`\Delta t`.
We have also derived explicit mathematical expressions for :math:`e^n`,
the truncation error, and the stability criteria.

.. Look in Asher and Petzold, p 40

Exercises
=========

.. --- begin exercise ---

.. _decay:analysis:exer:fd:exp:plot:

Problem 1: Visualize the accuracy of finite differences
-------------------------------------------------------

The purpose of this exercise is to visualize the accuracy of finite difference
approximations of the derivative of a given function.
For any finite difference approximation, take the Forward Euler difference
as an example, and any specific function, take  :math:`u=e^{-at}`,
we may introduce an error fraction

.. math::
        
        E = \frac{[D_t^+ u]^n}{u'(t_n)} &= \frac{\exp{(-a(t_n+\Delta t))} - \exp{(-at_n)}}{-a\exp{(-at_n)\Delta t}}\\ 
        &= \frac{1}{a\Delta t}\left(1 -\exp{(-a\Delta t)}\right),
        

and view :math:`E` as a function of :math:`\Delta t`. We expect that
:math:`\lim_{\Delta t\rightarrow 0}E=1`, while :math:`E` may deviate significantly from
unity for large :math:`\Delta t`. How the error depends on :math:`\Delta t` is best
visualized in a graph where we use a logarithmic scale for :math:`\Delta t`,
so we can cover many orders of magnitude of that quantity. Here is
a code segment creating an array of 100 intervals, on the logarithmic
scale, ranging from :math:`10^{-6}` to :math:`10^{-0.5}` and then plotting :math:`E` versus
:math:`p=a\Delta t` with logarithmic scale on the :math:`p` axis:

.. code-block:: python

        from numpy import logspace, exp
        from matplotlib.pyplot import semilogx
        p = logspace(-6, -0.5, 101)
        y = (1-exp(-p))/p
        semilogx(p, y)

Illustrate such errors for the finite difference operators :math:`[D_t^+u]^n`
(forward), :math:`[D_t^-u]^n` (backward), and :math:`[D_t u]^n` (centered) in
the same plot.

Perform a Taylor series expansions of the error fractions and find
the leading order :math:`r` in the expressions of type
:math:`1 + Cp^r + {\mathcal{O}(p^{r+1)}}`, where :math:`C` is some constant.

.. --- begin hint in exercise ---

**Hint.**
To save manual calculations and learn more about symbolic computing,
make functions for the three difference operators and use ``sympy``
to perform the symbolic differences, differentiation, and Taylor series
expansion. To plot a symbolic expression ``E`` against ``p``, convert the
expression to a Python function first: ``E = sympy.lamdify([p], E)``.

.. --- end hint in exercise ---

.. removed !bsol ... !esol environment (because of the command-line option --without_solutions)

Filename: ``decay_plot_fd_error``.

.. --- end exercise ---

.. --- begin exercise ---

.. _decay:analysis:exer:growth:

Problem 2: Explore the :math:`\theta`-rule for exponential growth
-----------------------------------------------------------------

This exercise asks you to solve the ODE :math:`u'=-au` with :math:`a < 0` such that
the ODE models exponential growth instead of exponential decay.  A
central theme is to investigate numerical artifacts and non-physical
solution behavior.

**a)**
Set :math:`a=-1` and run experiments with :math:`\theta=0, 0.5, 1` for
various values of :math:`\Delta t` to uncover numerical artifacts.
Recall that the exact solution is a
monotone, growing function when :math:`a < 0`. Oscillations or significantly
wrong growth are signs of wrong qualitative behavior.

From the experiments, select four values of :math:`\Delta t` that
demonstrate the kind of numerical solutions that are characteristic
for this model.

.. removed !bsol ... !esol environment (because of the command-line option --without_solutions)

**b)**
Write up the amplification factor and plot it for :math:`\theta=0,0.5,1`
together with the exact one for :math:`a\Delta t < 0`. Use the plot to
explain the observations made in the experiments.

.. --- begin hint in exercise ---

**Hint.**
Modify the `decay_ampf_plot.py <http://tinyurl.com/ofkw6kc/analysis/decay_ampf_plot.py>`__ code
(in the ``src/analysis`` directory).

.. --- end hint in exercise ---

.. removed !bsol ... !esol environment (because of the command-line option --without_solutions)

Filename: ``exponential_growth``.

.. --- end exercise ---

Various types of errors in a differential equation model
========================================================

So far we have been concerned with one type of error, namely the
discretization error committed by replacing the differential equation
problem by a recursive set of difference equations. There are,
however, other types of errors that must be considered too. We can
classify errors into four groups:

1. model errors

2. data errors

3. discretization errors

4. round-off errors

Below, we shall briefly describe and illustrate these four types
of errors.

Model errors
------------

Any mathematical model like :math:`u^{\prime}=-au`, :math:`u(0)=I`, is just an
approximate description of a real-world phenomenon. Suppose a more
accurate model has :math:`a` as a function of time rather than a
constant. Here we take :math:`a(t)` as a simple linear function: :math:`a +
pt`. Obviously, :math:`u` with :math:`p>0` will go faster to zero with time than a
constant :math:`a`.

The solution of

.. math::
         u^{\prime} = (a + pt)u,\quad u(0)=I,

can be shown (see below) to be

.. math::
         u(t) = I e^{t \left(- a - \frac{p t}{2}\right)}{\thinspace .}

With the above :math:`u` available in a Python function ``u_true(t, I, a, p)``
and the solution from our model :math:`u^{\prime}=-au` available in ``u(t, I, a)``,
we can make some plots of the two models and the error for some values
of :math:`p`. Figure :ref:`decay:analysis:model_errors:fig:model_u` displays
the two curves for :math:`p=0.01, 0.1, 1`, while Figure
:ref:`decay:analysis:model_errors:fig:model_e` shows the difference
between the two models as a function of :math:`t` for the same :math:`p` values.

.. _decay:analysis:model_errors:fig:model_u:

.. figure:: fig-analysis/model_errors_u.png
   :width: 800

   Comparison of two models for three values of :math:`p`

.. _decay:analysis:model_errors:fig:model_e:

.. figure:: fig-analysis/model_errors_e.png
   :width: 500

   Discrepancy of Comparison of two models for three values of :math:`p`

The code that was used to produce the plots looks like

.. code-block:: python

        from numpy import linspace, exp
        from matplotlib.pyplot import \ 
             plot, show, xlabel, ylabel, legend, savefig, figure, title
        
        def model_errors():
            p_values = [0.01, 0.1, 1]
            a = 1
            I = 1
            t = linspace(0, 4, 101)
            legends = []
            # Work with figure(1) for the discrepancy and figure(2+i)
            # for plotting the model and the true model for p value no i
            for i, p in enumerate(p_values):
                u = model(t, I, a)
                u_true = true_model(t, I, a, p)
                discrepancy = u_true - u
                figure(1)
                plot(t, discrepancy)
                figure(2+i)
                plot(t, u, 'r-', t, u_true, 'b--')
                legends.append('p=%g' % p)
            figure(1)
            legend(legends, loc='lower right')
            savefig('tmp1.png'); savefig('tmp1.pdf')
            for i, p in enumerate(p_values):
                figure(2+i)
                legend(['model', 'true model'])
                title('p=%g' % p)
                savefig('tmp%d.png' % (2+i)); savefig('tmp%d.pdf' % (2+i))

To derive the analytical solution of the model :math:`u^{\prime}=-(a+pt)u`, :math:`u(0)=I`, we can use SymPy and the code below. This is somewhat advanced
SymPy use for a newbie, but serves to illustrate the possibilities to
solve differential equations by symbolic software.

.. code-block:: python

        def derive_true_solution():
            import sympy as sym
            u = sym.symbols('u', cls=sym.Function)  # function u(t)
            t, a, p, I = sym.symbols('t a p I', real=True)
        
            def ode(u, t, a, p):
                """Define ODE: u' = (a + p*t)*u. Return residual."""
                return sym.diff(u, t) + (a + p*t)*u
        
            eq = ode(u(t), t, a, p)
            s = sym.dsolve(eq)
            # s is sym.Eq object u(t) == expression, we want u = expression,
            # so grab the right-hand side of the equality (Eq obj.)
            u = s.rhs
            print u
            # u contains C1, replace it with a symbol we can fit to
            # the initial condition
            C1 = sym.symbols('C1', real=True)
            u = u.subs('C1', C1)
            print u
            # Initial condition equation
            eq = u.subs(t, 0) - I
            s = sym.solve(eq, C1)  # solve eq wrt C1
            print s
            # s is a list s[0] = ...
            # Replace C1 in u by the solution
            u = u.subs(C1, s[0])
            print 'u:', u
            print sym.latex(u)  # latex formula for reports
        
            # Consistency check: u must fulfill ODE and initial condition
            print 'ODE is fulfilled:', sym.simplify(ode(u, t, a, p))
            print 'u(0)-I:', sym.simplify(u.subs(t, 0) - I)
        
            # Convert u expression to Python numerical function
            # (modules='numpy' allows numpy arrays as arguments,
            # we want this for t)
            u_func = sym.lambdify([t, I, a, p], u, modules='numpy')
            return u_func
        
        true_model = derive_true_solution()

Data errors
-----------

.. index:: Monte Carlo simulation

By "data" we mean all the input parameters to a model, in our case
:math:`I` and :math:`a`. The values of these may contain errors, or at least
uncertainty. Suppose :math:`I` and :math:`a` are measured from some physical
experiments. Ideally, we have many samples of :math:`I` and :math:`a` and
from these we can fit probability distributions. Assume that :math:`I`
turns out to be normally distributed with mean 1 and standard deviation 0.2,
while :math:`a` is uniformly distributed in the interval :math:`[0.5, 1.5]`.

How will the uncertainty in :math:`I` and :math:`a` propagate through the model
:math:`u=Ie^{-at}`? That is, what is the uncertainty in :math:`u` at a particular
time :math:`t`? This answer can easily be answered using *Monte Carlo
simulation*. It means that we draw a lot of samples from the
distributions for :math:`I` and :math:`a`. For each combination of :math:`I` and :math:`a`
sample we compute the corresponding :math:`u` value for selected values of
:math:`t`.  Afterwards, we can for each selected :math:`t` values make a histogram
of all the computed :math:`u` values to see what the distribution of :math:`u`
values look like. Figure :ref:`decay:analysis:data_errors:fig` shows the
histograms corresponding to :math:`t=0,1,3`. We see that the distribution of
:math:`u` values is much like a symmetric normal distribution at :math:`t=0`,
centered around :math:`u=1`. At later times, the distribution gets more
asymmetric and narrower. It means that the uncertainty decreases with
time. From the computed :math:`u` values we can easily calculate the mean
and standard deviation. The table below shows the mean and standard
deviation values along with the value if we just use the formula
:math:`u=Ie^{-at}` with the mean values of :math:`I` and :math:`a`: :math:`I=1` and :math:`a=1`. As
we see, there is some discrepancy between this latter (naive)
computation and the mean value produced by Monte Carlo simulation.

====  ====  =======  ====================  
time  mean  st.dev.  :math:`u(t;I=1,a=1)`  
====  ====  =======  ====================  
0     1.00  0.200    1.00                  
1     0.38  0.135    0.37                  
3     0.07  0.060    0.14                  
====  ====  =======  ====================  

Actually, :math:`u(t;I,a)` becomes a stochastic variable for each :math:`t` when
:math:`I` and :math:`a` are stochastic variables, as they are in the above
Monte Carlo simulation. The mean of the stochastic :math:`u(t;I,a)` is
not equal to :math:`u(t;I=1,a=1)` unless :math:`u` is linear in :math:`I` and :math:`a`
(here :math:`u` is nonlinear in :math:`a`). The accuracy of the Monte Carlo
results increases with increasing number of samples.

.. _decay:analysis:data_errors:fig:

.. figure:: fig-analysis/data_errors.png
   :width: 800

   *Histogram of solution uncertainty at three time points, due to data errors*

The computer code required to do the Monte Carlo simulation and
produce the plots in Figure :ref:`decay:analysis:data_errors:fig`
is shown below.

.. code-block:: python

        def data_errors():
            from numpy import random, mean, std
            from matplotlib.pyplot import hist
            N = 10000
            # Draw random numbers for I and a
            I_values = random.normal(1, 0.2, N)
            a_values = random.uniform(0.5, 1.5, N)
            # Compute corresponding u values for some t values
            t = [0, 1, 3]
            u_values = {}  # samples for various t values
            u_mean = {}
            u_std = {}
            for t_ in t:
                # Compute u samples corresponding to I and a samples
                u_values[t_] = [model(t_, I, a)
                                for I, a in zip(I_values, a_values)]
                u_mean[t_] = mean(u_values[t_])
                u_std[t_] = std(u_values[t_])
        
                figure()
                dummy1, bins, dummy2 = hist(
                    u_values[t_], bins=30, range=(0, I_values.max()),
                    normed=True, facecolor='green')
                #plot(bins)
                title('t=%g' % t_)
                savefig('tmp_%g.png' % t_); savefig('tmp_%g.pdf' % t_)
            # Table of mean and standard deviation values
            print 'time   mean   st.dev.'
            for t_ in t:
                print '%3g    %.2f    %.3f' % (t_, u_mean[t_], u_std[t_])

Discretization errors
---------------------

The errors implied by solving the differential equation problem by
the :math:`\theta`-rule has been thoroughly analyzed in the previous
sections. Below are some plots of the error versus time for the
Forward Euler (FE), Backward Euler (BN), and Crank-Nicolson (CN)
schemes for decreasing values of :math:`\Delta t`. Since the difference
in magnitude between the errors in the CN scheme versus the FE ad
BN schemes grows significantly as :math:`\Delta t` is reduced (the error
goes like :math:`\Delta t^2` for CN versus :math:`\Delta t` for FE/BE), we have
plotted the logarithm of the absolute value of the numerical error.

.. _decay:analysis:numerical_errors:fig:

.. figure:: fig-analysis/numerical_errors.png
   :width: 700

   *Discretization errors in various schemes for 4 time step values*

The computer code used to generate the plots appear next. It makes use
of a ``solver`` function
for computing the numerical solution of :math:`u^{\prime}=-au` with
the :math:`\theta`-rule.

.. code-block:: python

        def discretization_errors():
            from numpy import log, abs
            I = 1
            a = 1
            T = 4
            t = linspace(0, T, 101)
            schemes = {'FE': 0, 'BE': 1, 'CN': 0.5}  # theta to scheme name
            dt_values = [0.8, 0.4, 0.1, 0.01]
            for dt in dt_values:
                figure()
                legends = []
                for scheme in schemes:
                    theta = schemes[scheme]
                    u, t = solver(I, a, T, dt, theta)
                    u_e = model(t, I, a)
                    error = u_e - u
                    # Plot log(error), but exclude error[0] since it is 0
                    plot(t[1:], log(abs(error[1:])))
                    legends.append(scheme)
                xlabel('t');  ylabel('log(abs(numerical error))')
                legend(legends, loc='upper right')
                title(r'$\Delta t=%g$' % dt)
                savefig('tmp_dt%g.png' % dt); savefig('tmp_dt%g.pdf' % dt)

Rounding errors
---------------

Real numbers on a computer are represented by `floating-point numbers <https://en.wikipedia.org/wiki/Floating_point>`__, which means that just a finite
number of digits are stored and used. Therefore, the floating-point number
is an approximation to the underlying real number. When doing
arithmetics with floating-point numbers, there will be small
approximation errors, called round-off errors or rounding errors,
that may or may not accumulate in comprehensive computations.
A typical example is shown below.

.. code-block:: python

        >>> 1.0/51*51
        1.0
        >>> 1.0/49*49
        0.9999999999999999

We see there is not an exact result in the latter case, but a rounding
error of :math:`10^{-16}`. This is the typical level of a rounding error
from an arithmetic operation with 64 bit floating-point numbers
(``float`` object in Python, often called ``double`` or double precision
in other languages).

What is the effect of using ``float`` objects and not exact arithmetics?
Python has a ``Decimal`` object in the `decimal <https://docs.python.org/2/library/decimal.html>`__ module that allows us
to use as many digits in floating-point numbers as we like. We take
1000 digits as the true answer where rounding errors are negligible,
and then we run our numerical algorithm (the Crank-Nicolson scheme to
be precise) with ``Decimal`` objects for all real numbers and test the
error arising from using 4, 16, 64, and 128 digits.

When computing with numbers around unity in size, we typically
get a rounding error of :math:`10^{-d}`, where :math:`d` is the number
of digits used. However, if we compute with numbers that
are larger, e.g., the :math:`u` values implied by :math:`I=1000` and :math:`a=100`,
the rounding errors increase to about :math:`10^{-d+3}`. Below is
a table of the the computed maximum rounding error for various number of digits
and two different magnitudes of :math:`I` and :math:`a`.

======  ===========================  =============================  
digits    :math:`I=1`, :math:`a=1`   :math:`I=1000`, :math:`a=100`  
======  ===========================  =============================  
     4  :math:`3.05\cdot 10^{-4}`    :math:`3.05\cdot 10^{-1}`      
    16  :math:`1.71\cdot 10^{-16}`   :math:`1.58\cdot 10^{-13}`     
    64  :math:`2.99\cdot 10^{-64}`   :math:`2.06\cdot 10^{-61}`     
   128  :math:`1.60\cdot 10^{-128}`  :math:`2.41\cdot 10^{-125}`    
======  ===========================  =============================  

The computer code for doing these experiments need a new version
of the ``solver`` function where we do arithmetics with ``Decimal``
objects:

.. code-block:: python

        def solver_decimal(I, a, T, dt, theta):
            """Solve u'=-a*u, u(0)=I, for t in (0,T] with steps of dt."""
            from numpy import zeros, linspace
            from decimal import Decimal as D
            dt = D(dt)
            a = D(a)
            theta = D(theta)
            Nt = int(round(D(T)/dt))
            T = Nt*dt
            u = zeros(Nt+1, dtype=object)  # array of Decimal objects
            t = linspace(0, float(T), Nt+1)
        
            u[0] = D(I)               # assign initial condition
            for n in range(0, Nt):    # n=0,1,...,Nt-1
                u[n+1] = (1 - (1-theta)*a*dt)/(1 + theta*dt*a)*u[n]
            return u, t

The function below carries out the experiments. Note that we can
set the number of digits as we want through the ``decimal.getcontext().prec``
variable.

.. code-block:: python

        def rounding_errors(I=1, a=1, T=4, dt=0.1):
            import decimal
            from numpy import log, array, abs
            digits_values = [4, 16, 64, 128]
            # "Exact" arithmetics is taken as 1000 decimals here
            decimal.getcontext().prec = 1000
            u_e, t = solver_decimal(I=I, a=a, T=T, dt=dt, theta=0.5)
            for digits in digits_values:
                decimal.getcontext().prec = digits  # set no of digits
                u, t = solver_decimal(I=I, a=a, T=T, dt=dt, theta=0.5)
                error = u_e - u
                error = array(error[1:], dtype=float)
                print '%d digits, max abs(error): %.2E' % \ 
                      (digits, abs(error).max())

Discussion of the size of various errors
----------------------------------------

.. !split
