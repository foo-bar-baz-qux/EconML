Problem Setup and API Design
============================


.. rubric::
    Potential Outcomes Formulation

All of the applications described in the previous section fall into the following general problem: 
given two vectors of treatments :math:`\vec{t}_0, \vec{t}_1 \in \T`, a vector of co-variates :math:`\vec{z}` 
and a random vector of potential outcomes :math:`Y(\vec{t})`, we want to estimate the quantity: 

.. math ::
    \tau(\vec{t}_0, \vec{t}_1, \vec{x}) = \E[Y(\vec{t}_1) - Y(\vec{t}_0) | X=\vec{x}]

We will refer to the latter quantity as the *heterogeneous treatment effect* of going from treatment 
:math:`\vec{t}_0` to treatment :math:`\vec{t}_1` conditional on observables :math:`\vec{x}`.  
If treatments are continuous, then one might also be interested in a local effect around a treatment point. 
The latter translates to estimating a local gradient around a treatment vector conditional on observables:

.. math ::
    \partial\tau(\vec{t}, \vec{x}) = \E\left[\nabla_{\vec{t}} Y(\vec{t}) | X=\vec{x}\right] \tag{marginal CATE}

We will refer to the latter as the *heterogeneous marginal effect*. [1]_ 
Finally, we might not only be interested in the effect but also in the actual *counterfactual prediction*, i.e. estimating the quatity: 

.. math ::
    \mu(\vec{t}, \vec{x}) = \E\left[Y(\vec{t}) | X=\vec{x}\right] \tag{counterfactual prediction}

We assume we have data that are generated from some collection policy. In particular, we assume that we have data of the form: 
:math:`\{Y_i(T_i), T_i, X_i, W_i, Z_i\}`, where :math:`Y_i(T_i)` is the observed outcome for the chosen treatment, 
:math:`T_i` is the treatment, :math:`X_i` are the co-variates used for heterogeneity, 
:math:`W_i` are other observable co-variates that we believe are affecting the potential outcome :math:`Y_i(T_i)` 
and potentially also the treatment :math:`T_i`; and :math:`Z_i` are variables that affect 
the treatment :math:`T_i` but do not directly affect the potential outcome. 
We will refer to variables :math:`W_i` as *controls* and variables :math:`Z_i` as *instruments*. 
The variables :math:`X_i` can also be thought of as *control* variables, but they are special in the sense that 
they are a subset of the controls with respect to which we want to measure treatment effect heterogeneity. 
We will refer to them as *features*.

.. rubric:: 
    Structural Equation Formulation

We can equivalently describe the data and the quantities of interest via the means of structural equations. In particular, 
suppose that we observe i.i.d. samples :math:`\{Y_i, T_i, X_i, W_i, Z_i\}` from some joint distribution and 
we assume the following structural equation model of the world:

.. math ::
    Y =~& g(T, X, W, \epsilon)

    T =~& f(X, W, Z, \eta)

where :math:`\epsilon` and :math:`\eta` are extra *noise* random variables, that could be potentially correlated with each other. 
The target quantity that we want to estimate can then be expressed as:

.. math ::
    :nowrap:

    \begin{align}
        \tau(\vec{t}_0, \vec{t}_1, \vec{x}) =~& \E[g(\vec{t}_1, X, W, \epsilon) - g(\vec{t}_0, X, W, \epsilon) | X=\vec{x}] \tag{CATE} \\
        \partial\tau(\vec{t}, \vec{x}) =~& \E[\nabla_{\vec{t}} g(\vec{t}, X, W, \epsilon) | X=\vec{x}] \tag{marginal CATE} \\
    \end{align}

where in these expectations, the random variables :math:`W, \epsilon` are taken from the same distribution as the one that generated the data. 
In other words, there is a one-to-one correspondence between the potential outcomes formulation and the structural equations formulation 
in that the random variable :math:`Y(t)` is equal to the random variable :math:`g(t, X, W, \epsilon)`, where :math:`X, W, \epsilon` 
is drawn from the distribution that generated each sample in the data set.

API of Conditional Average Treatment Effect (CATE) Package
----------------------------------------------------------

.. code-block:: python3
    :caption: Base CATE Estimator Class

    class BaseCateEstimator
        
        def fit(self, Y, T, X=None, W=None, Z=None):
            ''' Estimates the counterfactual model from data, i.e. estimates functions 
            τ(·, ·, ·)}, ∂τ(·, ·) and μ(·, ·)
        
            Parameters:
            Y: (n × d_y) matrix of outcomes for each sample
            T: (n × d_t) matrix of treatments for each sample
            X: optional (n × d_x) matrix of features for each sample
            W: optional (n × d_w) matrix of controls for each sample
            Z: optional (n × d_z) matrix of instruments for each sample
            '''
        
        def effect(self, T0, T1, X=None):
            ''' Calculates the heterogeneous treatment effect τ(·, ·, ·) between two treatment
            points conditional on a vector of features on a set of m test samples {T0_i, T1_i, X_i}
        
            Parameters:
            T0: (m × d_t) matrix of base treatments for each sample
            T1: (m × d_t) matrix of target treatments for each sample
            X: optional (m × d_x) matrix of features for each sample
        
            Returns:
            tau: (m × d_y) matrix of heterogeneous treatment effects on each outcome
                for each sample
            '''
        
        def marginal_effect(self, T, X=None):
            ''' Calculates the heterogeneous marginal effect ∂τ(·, ·) around a base treatment
            point conditional on a vector of features on a set of m test samples {T_i, X_i}
        
            Parameters:
            T: (m × d_t) matrix of base treatments for each sample
            X: optional (m × d_x) matrix of features for each sample
        
            Returns:
            grad_tau: (m × d_y × d_t) matrix of heterogeneous marginal effects on each outcome
                for each sample
            '''


Linear in Treatment CATE Estimators
-----------------------------------

In many settings, we might want to make further structural assumptions on the form of the data generating process. One particular prevalent assumption is that the outcome $y$ is linear in the treatment vector and therefore that the marginal effect is constant across treatments, i.e.:

.. math ::
    Y =~& H(X, W) \cdot T + g(X, W, \epsilon)

    T =~& f(X, W, Z, \eta)

where :math:`\epsilon, \eta` are exogenous noise terms. Under such a linear response assumption we observe that the CATE and marginal CATE takes a special form of:

.. math ::

    \tau(\vec{t}_0, \vec{t}_1, \vec{x}) =~& \E[H(X, W) | X=\vec{x}] \cdot (\vec{t}_1 - \vec{t}_0) 

    \partial \tau(\vec{t}, \vec{x}) =~&  \E[H(X, W) | X=\vec{x}]

Hence, the marginal CATE is independent of :math:`\vec{t}`. In these settings, we will denote with :math:`\theta(\vec{x})` the constant marginal CATE, i.e. 

.. math ::
    \theta(\vec{x}) = \E[H(X, W) | X=\vec{x}] \tag{constant marginal CATE}

Given the prevalence of linear treatment effect assumptions, we will create a generic LinearCateEstimator, which will support a method that returns the constant marginal CATE at any target feature vector :math:`\vec{x}`.

.. code-block:: python3
    :caption: Linear CATE Estimator Class

    class LinearCateEstimator(BaseCateEstimator):
        
        def const_marginal_effect(self, X=None):
            ''' Calculates the constant marginal CATE θ(·) conditional on a vector of
            features on a set of m test samples {X_i}
        
            Parameters:
            X: optional (m × d_x) matrix of features for each sample
        
            Returns:
            theta: (m × d_y × d_t) matrix of constant marginal CATE of each treatment
            on each outcome	for each sample
            '''
        
        def effect(self, T0, T1, X=None):
            return const_marginal_effect(X) * (T1 - T0)
        
        def marginal_effect(self, T, X=None)
            return const_marginal_effect(X)

Example Use of API
------------------

.. code-block:: python3
    :caption: Example Data Generated from Structural Equations

    import numpy as np

    # Instance parameters
    n_controls = 100
    n_instruments = 1
    n_features = 1
    n_treatments = 1
    alpha = np.random.normal(size=(n_controls, 1))
    beta = np.random.normal(size=(n_instruments, 1))
    gamma = np.random.normal(size=(n_treatments, 1))
    delta = np.random.normal(size=(n_treatments, 1))
    zeta = np.random.normal(size=(n_controls, 1))

    ''' Generate data from structural equations model:
            y = γ t^2 + δ x t + ⟨ζ,w⟩ + κ + ϵ
            t = ⟨α,w⟩ + ⟨β,z⟩ + η
    '''
    n_samples = 1000
    W = np.random.normal(size=(n_samples, n_controls))
    Z = np.random.normal(size=(n_samples, n_instruments))
    X = np.random.normal(size=(n_samples, n_features))
    eta = np.random.normal(size=(n_samples, n_treatments))
    epsilon = np.random.normal(size=(n_samples, 1))
    T = np.dot(W, alpha) + np.dot(Z, beta) + eta
    y = np.dot(T**2, gamma) + np.dot(np.multiply(T, X), delta) + np.dot(W, zeta) + epsilon

.. code-block:: python3
    :caption: Example Use of Package

    # Fit counterfactual model 
    cfest = BaseCateEstimator()
    cfest.fit(y, T, X, W, Z)
    X_test = X
    # Estimate heterogeneous treatment effects from going from treatment 0 to treatment 1
    T0_test = np.zeros((X_test.shape[0], n_treatments))
    T1_test = np.ones((X_test.shape[0], n_treatments))
    hetero_te = cfest.effect(T0_test, T1_test, X_test) # returns estimates of γ + δ X_test

    # Estimate heterogeneous marginal effects around treatment 0
    T_test = np.zeros((X_test.shape[0], n_treatments))
    hetero_marginal_te = cfest.marginal_effect(T_test, X_test) # returns estimates of δ X_test

    # Estimate average treatment effects over a population of z's
    T0_test = np.zeros((X_test.shape[0], n_treatments))
    T1_test = np.ones((X_test.shape[0], n_treatments))

    # average treatment effect
    ate = np.mean(cfest.effect(T0_test, T1_test, X_test)) # returns estimate of γ + δ 𝔼[x]

    # average treatment effect of population with x>1/2
    # returns estimate of γ + δ 𝔼[x | x>1/2]
    cate = np.mean(cfest.effect(T0_test[X_test>1/2], T1_test[X_test>1/2], X_test[X_test>1/2])) 

    # Estimate expected lift of treatment policy: π(z) = 𝟙{x > 0} over existing policy
    Pi0_test = T
    Pi1_test = (X_test > 0) * 1.
    # returns estimate of γ/2 + δ/√(2π)
    policy_effect = np.mean(cfest.effect(Pi0_test, Pi1_test, X_test)) 

    # Estimate expected lift of treatment policy: π(x) = 𝟙{x > 0} over baseline of no treatment
    Pi0_test = np.zeros((X_test.shape[0], n_treatments))
    Pi1_test = (X_test > 0) * 1.
    # returns estimate of γ/2 + δ/√(2π)
    policy_effect = np.mean(cfest.effect(Pi0_test, Pi1_test, X_test)) 

.. rubric:: Footnotes

.. [1] One can always approximate the latter with the former and vice versa, 
    i.e. :math:`\partial_i \tau(\vec{t},\vec{x}) \approx \tau(\vec{t}, \vec{t} + \delta \vec{e}_i, \vec{x})/\delta` 
    for some small enough :math:`\delta`, and similarly, 
    :math:`\tau(\vec{t_0}, \vec{t_1}, \vec{x}) = \int_{0}^{1} \partial\tau(\vec{t}_0 + q (\vec{t}_1 - \vec{t}_0), \vec{x}) (\vec{t}_1 - \vec{t_0})dq`. 
    However, in many settings more direct methods that make use of the structure might simplify these generic transformations.

