:orphan:

# Release 0.24.0-dev (development release)

<h3>New features since last release</h3>

* The JAX JIT interface now supports evaluating vector-valued QNodes
  enabling new types of workflows to utilize the power of just-in-time
  compilation for significant performance boosts.
  [(#2034)](https://github.com/PennyLaneAI/pennylane/pull/2034)

  Vector-valued QNodes include those with:
  * `qml.probs`;
  * `qml.state`;
  * `qml.sample` or
  * multiple `qml.expval` / `qml.var` measurements.

  Consider a QNode that returns basis-state probabilities:
  ```python
  dev = qml.device('default.qubit', wires=2)
  x = jnp.array(0.543)
  y = jnp.array(-0.654)

  @jax.jit
  @qml.qnode(dev, diff_method="parameter-shift", interface="jax")
  def circuit(x, y):
      qml.RX(x, wires=[0])
      qml.RY(y, wires=[1])
      qml.CNOT(wires=[0, 1])
      return qml.probs(wires=[1])
  ```
  The QNode can now be evaluated:
  ```pycon
  >>> circuit(x, y)
  DeviceArray([0.8397495 , 0.16025047], dtype=float32)
  ```
  Computing the jacobian of vector-valued QNodes is not supported with the JAX
  JIT interface. The output of vector-valued QNodes can, however, be used in
  the definition of scalar-valued cost functions whose gradients can be
  computed.

  For example, one can define a cost function that outputs the first element of
  the probability vector:
  ```python
  def cost(x, y):
      return circuit(x, y)[0]
  ```
  ```pycon
  >>> jax.grad(cost, argnums=[0])(x, y)
  (DeviceArray(-0.2050439, dtype=float32),)
  ```

* A new quantum information module is added. It includes a function for computing the reduced density matrix functions
  for state vectors and density matrices.

  [(#2554)](https://github.com/PennyLaneAI/pennylane/pull/2554)
  [(#2569)](https://github.com/PennyLaneAI/pennylane/pull/2569)
  [(#2598)](https://github.com/PennyLaneAI/pennylane/pull/2598)
  [(#2617)](https://github.com/PennyLaneAI/pennylane/pull/2617)
  [(#2631)](https://github.com/PennyLaneAI/pennylane/pull/2631)
  [(#2640)](https://github.com/PennyLaneAI/pennylane/pull/2640)
  [(#2663)](https://github.com/PennyLaneAI/pennylane/pull/2663)
  [(#2684)](https://github.com/PennyLaneAI/pennylane/pull/2684)
  
  A `reduced_dm` function that can handle both state vectors and density matrix, to return a reduced density matrix:
  
  ```pycon
  >>> x = [1, 0, 1, 0] / np.sqrt(2)
  >>> reduced_dm(x, indices=[0])
  [[0.5+0.j 0.5+0.j]
   [0.5+0.j 0.5+0.j]]

  >>> reduced_dm(x, indices=[1])
  [[1.+0.j 0.+0.j]
   [0.+0.j 0.+0.j]]

  >>> y = [[0.5, 0, 0.0, 0.5], [0, 0, 0, 0], [0, 0, 0, 0], [0.5, 0, 0, 0.5]]
  >>> reduced_dm(y, indices=[0])
  [[0.5+0.j 0.0+0.j]
   [0.0+0.j 0.5+0.j]]

  >>> import tensorflow as tf
  >>> z = tf.Variable([[1, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0]], dtype=tf.complex128)
  >>> reduced_dm(z, indices=[1])
  tf.Tensor(
  [[1.+0.j 0.+0.j]
   [0.+0.j 0.+0.j]], shape=(2, 2), dtype=complex128)
  ```

  It also contains a `QNode` transform `qml.qinfo.reduced_dm`, that returns the density matrix from a `QNode`
  returning `qml.state`:
  ```python3
  dev = qml.device("default.qubit", wires=2)
  @qml.qnode(dev)
  def circuit(x):
      qml.IsingXX(x, wires=[0,1])
      return qml.state()
  ```
  ```pycon
  
  >>> qml.qinfo.reduced_dm(circuit, wires=[0])(np.pi/2)
  [[0.5+0.j 0.+0.j]
   [0.+0.j 0.5+0.j]]
  ```
  
  We add Von Neumann entropy capabilities, `qml.math.vn_entropy` that accepts both state vectors and density matrices
  for all interfaces (Numpy, Autograd, Torch, Tensorflow and Jax).

  ```pycon
  >>> x = [1, 0, 0, 1] / np.sqrt(2)
  >>> vn_entropy(x, indices=[0])
  0.6931472
  
  >>> y = [[1/2, 0, 0, 1/2], [0, 0, 0, 0], [0, 0, 0, 0], [1/2, 0, 0, 1/2]]
  >>> vn_entropy(x, indices=[0])
  0.6931472
  ```
  
  A Von Neumann measurement process `qml.vn_entropy` can be used as return in QNodes:

  ```python3
  dev = qml.device("default.qubit", wires=2)
  @qml.qnode(dev)
  def circuit_entropy(x):
      qml.IsingXX(x, wires=[0,1])
      return qml.vn_entropy(wires=[0], log_base=2)
  ```
  
  ```pycon
  >>> circuit_entropy(np.pi/2)
  1.0
  ```
  The quantum information module also now contains a QNode (returning states) transform for the Von Neumann entropy 
  `qml.qinfo.vn_entropy`:
  
  ```python3
  dev = qml.device("default.qubit", wires=2)
  @qml.qnode(dev)
  def circuit_entropy(x):
      qml.IsingXX(x, wires=[0,1])
      return qml.state()
  ```
  
  ```pycon
  >>> vn_entropy(circuit, indices=[0], base=2)(np.pi/2)
  1.0
  ```

  We add Von Neumann entropy capabilities, `qml.math.vn_entropy` that accepts both state vectors and density matrices
  for all interfaces (Numpy, Autograd, Torch, Tensorflow and Jax).

  ```pycon
  >>> x = [1, 0, 0, 1] / np.sqrt(2)
  >>> vn_entropy(x, indices=[0])
  0.6931472

  >>> y = [[1/2, 0, 0, 1/2], [0, 0, 0, 0], [0, 0, 0, 0], [1/2, 0, 0, 1/2]]
  >>> vn_entropy(x, indices=[0])
  0.6931472
  ```

  A Von Neumann measurement process `qml.vn_entropy` can be used as return in QNodes:

  ```python3
  dev = qml.device("default.qubit", wires=2)
  @qml.qnode(dev)
  def circuit_entropy(x):
      qml.IsingXX(x, wires=[0,1])
      return qml.vn_entropy(wires=[0], log_base=2)
  ```

  ```pycon
  >>> circuit_entropy(np.pi/2)
  1.0
  ```

  The quantum information module also now contains a QNode (returning states) transform for the Von Neumann entropy
  `qml.qinfo.vn_entropy`:

  ```python3
  dev = qml.device("default.qubit", wires=2)
  @qml.qnode(dev)
  def circuit_entropy(x):
      qml.IsingXX(x, wires=[0,1])
      return qml.state()
  ```

  ```pycon
  >>> vn_entropy(circuit, indices=[0], base=2)(np.pi/2)
  1.0
  ```

  Support for mutual information computation is also added. The `qml.math.mutual_info`
  function computes the mutual information from a state vector or a density matrix:
  ```pycon
  >>> x = np.array([1, 0, 0, 1]) / np.sqrt(2)
  >>> qml.math.mutual_info(x, indices0=[0], indices1=[1])
  1.3862943611198906
  >>>
  >>> y = np.array([[1/2, 0, 0, 1/2], [0, 0, 0, 0], [0, 0, 0, 0], [1/2, 0, 0, 1/2]])
  >>> qml.math.mutual_info(x, indices0=[0], indices1=[1])
  1.3862943611198906
  ```
  The `qml.mutual_info` measurement process can be returned from a QNode:
  ```python3
  dev = qml.device("default.qubit", wires=2)

  @qml.qnode(dev)
  def circuit(x):
      qml.IsingXX(x, wires=[0, 1])
      return qml.mutual_info(wires0=[0], wires1=[1])
  ```
  ```pycon
  >>> circuit(np.pi / 2)
  tensor(1.38629436, requires_grad=True)
  ```

  The `qml.qinfo.mutual_info` can be used to transform a QNode returning
  a state to a function that returns the mutual information:
  ```python3
  dev = qml.device("default.qubit", wires=2)

  @qml.qnode(dev)
  def circuit(x):
      qml.IsingXX(x, wires=[0, 1])
      return qml.state()
  ```

  ```pycon
  >>> mutual_info_circuit = qml.qinfo.mutual_info(circuit, wires0=[0], wires1=[1])
  >>> mutual_info_circuit(np.pi / 2)
  1.3862943611198906
  ```
  
  Support for the classical and quantum Fisher information matrices, `qml.qinfo.classical_fisher` and `qml.qinfo.quantum_fisher` is also added:

  These are typically employed in variational optimization schemes to tilt the gradient in a more favorable direction, see [2103.15191](https://arxiv.org/abs/2103.15191) and [1909.02108](https://arxiv.org/abs/1909.02108). Here is a very simple example of a Hamiltonian loss function:

  ```python3
  n_wires = 3

  dev = qml.device("default.qubit", wires=n_wires)

  @qml.qnode(dev)
  def circ(params):
      qml.RY(params[0], wires=1)
      qml.CNOT(wires=(1,0))
      qml.RY(params[1], wires=1)
      qml.RZ(params[2], wires=1)
      return qml.expval(1.*qml.PauliX(0) @ qml.PauliX(1) - 0.5 * qml.PauliZ(1))

  params = pnp.array([0.5, 1., 0.2], requires_grad=True)
  ```
  From this circuit we can directly obtain the gradient of the expectation value, as well as the classical fisher information matrix (cfim) and quantum fisher information matrix (qfim) of the variational state.
  ```pycon
  >>> grad = qml.grad(circ)(params)
  >>> cfim = qml.qinfo.classical_fisher(circ)(params)
  >>> qfim = qml.qinfo.quantum_fisher(circ)(params)
  ```
  From this we can compute the tilted (natural) gradients:
  ```pycon
  >>> c_grad = cfim @ grad
  >>> q_grad = qfim @ grad
  >>> print(f"Gradient: {grad} \n  c_grad: {c_grad} \n  q_grad: {q_grad}")
  Gradient: [ 0.59422561 -0.02615095 -0.05146226] 
    c_grad: [ 5.94225615e-01 -2.61509542e-02 -1.18674655e-18] 
    q_grad: [ 0.59422561 -0.02615095 -0.03989212]
  ```
  
  The support for calculating the fidelity between two arbitrary states is added as `qml.math.fidelity` for state 
  vectors and density matrices.
  
  ```pycon
  >>> state0 = [0, 1]
  >>> state1 = [[0, 0], [0, 1]]
  >>> qml.math.fidelity(state0, state1)
  1.0
  
  >>> state0 = [[0.5, 0.5], [0.5, 0.5]]
  >>> state1 = [[0.5, 0], [0, 0.5]]
  >>> qml.math.fidelity(state0, state1)
  0.4999999999999998
  ```
  The quantum information module now have a differentiable fidelity transform for QNodes.
  
  ```python
  dev = qml.device('default.qubit', wires=1)

  @qml.qnode(dev)
  def circuit_rx(x, y):
      qml.RX(x, wires=0)
      qml.RZ(y, wires=0)
      return qml.state()

  @qml.qnode(dev)
  def circuit_ry(y):
      qml.RY(y, wires=0)
      return qml.state()
  ```
  ```pycon
  >>> qml.qinfo.fidelity(circuit_rx, circuit_ry, wires0=[0], wires1=[0])((0.1, 0.3), (0.2))
  0.9905158135644924
  ```
  

* Operators have new attributes `ndim_params` and `batch_size`, and `QuantumTapes` have the new
  attribute `batch_size`.
  - `Operator.ndim_params` contains the expected number of dimensions per parameter of the operator,
  - `Operator.batch_size` contains the size of an additional parameter broadcasting axis, if present,
  - `QuantumTape.batch_size` contains the `batch_size` of its operations (see below).
  
* New `solarized_light` and `solarized_dark` styles available for drawing circuit diagram graphics. 
  [(#2662)](https://github.com/PennyLaneAI/pennylane/pull/2662)
  
* Support adding `Observable` objects to the integer `0`.
  [(#2603)](https://github.com/PennyLaneAI/pennylane/pull/2603)

  This allows us to directly sum a list of observables as follows:
  ```
  H = sum([qml.PauliX(i) for i in range(10)])
  ```

* Parameter broadcasting within operations and tapes was introduced.

  [(#2575)](https://github.com/PennyLaneAI/pennylane/pull/2575)
  [(#2590)](https://github.com/PennyLaneAI/pennylane/pull/2590)
  [(#2609)](https://github.com/PennyLaneAI/pennylane/pull/2609)

  Parameter broadcasting refers to passing parameters with a (single) leading additional
  dimension (compared to the expected parameter shape) to `Operator`'s.
  Introducing this concept involves multiple changes:

  1. New class attributes
    - `Operator.ndim_params` can be specified by developers to provide the expected number of dimensions for each parameter
      of an operator.
    - `Operator.batch_size` returns the size of an additional parameter-broadcasting axis,
      if present.
    - `QuantumTape.batch_size` returns the `batch_size` of its operations (see logic below).
    - `Device.capabilities()["supports_broadcasting"]` is a Boolean flag indicating whether a
      device natively is able to apply broadcasted operators.
  2. New functionalities
    - `Operator`s use their new `ndim_params` attribute to set their new attribute `batch_size`
      at instantiation. `batch_size=None` corresponds to unbroadcasted operators.
    - `QuantumTape`s automatically determine their new `batch_size` attribute from the
      `batch_size`s of their operations. For this, all `Operators` in the tape must have the same
      `batch_size` or `batch_size=None`. That is, mixing broadcasted and unbroadcasted `Operators`
      is allowed, but mixing broadcasted `Operators` with differing `batch_size` is not,
      similar to NumPy broadcasting.
    - A new tape `batch_transform` called `broadcast_expand` was added. It transforms a single
      tape with `batch_size!=None` (broadcasted) into multiple tapes with `batch_size=None`
      (unbroadcasted) each.
    - `Device`s natively can handle broadcasted `QuantumTape`s by using `broadcast_expand` if
      the new flag `capabilities()["supports_broadcasting"]` is set to `False` (the default).
  3. Feature support
    - Many parametrized operations now have the attribute `ndim_params` and
      allow arguments with a broadcasting dimension in their numerical representations.
      This includes all gates in `ops/qubit/parametric_ops` and `ops/qubit/matrix_ops`.
      The broadcasted dimension is the first dimension in representations.
      Note that the broadcasted parameter has to be passed as an `tensor` but not as a python
      `list` or `tuple` for most operations.

  **Example**

  Instantiating a rotation gate with a one-dimensional array leads to a broadcasted `Operation`:

  ```pycon
  >>> op = qml.RX(np.array([0.1, 0.2, 0.3], requires_grad=True), 0)
  >>> op.batch_size
  3
  ```

  It's matrix correspondingly is augmented by a leading dimension of size `batch_size`:

  ```pycon
  >>> np.round(op.matrix(), 4)
  tensor([[[0.9988+0.j    , 0.    -0.05j  ],
         [0.    -0.05j  , 0.9988+0.j    ]],
        [[0.995 +0.j    , 0.    -0.0998j],
         [0.    -0.0998j, 0.995 +0.j    ]],
        [[0.9888+0.j    , 0.    -0.1494j],
         [0.    -0.1494j, 0.9888+0.j    ]]], requires_grad=True)
  >>> op.matrix().shape
  (3, 2, 2)
  ```

  A tape with such an operation will detect the `batch_size` and inherit it:

  ```pycon
  >>> with qml.tape.QuantumTape() as tape:
  >>>     qml.apply(op)
  >>> tape.batch_size
  3
  ```

  A tape may contain broadcasted and unbroadcasted `Operation`s

  ```pycon
  >>> with qml.tape.QuantumTape() as tape:
  >>>     qml.apply(op)
  >>>     qml.RY(1.9, 0)
  >>> tape.batch_size
  3
  ```

  but not `Operation`s with differing (non-`None`) `batch_size`s:

  ```pycon
  >>> with qml.tape.QuantumTape() as tape:
  >>>     qml.apply(op)
  >>>     qml.RY(np.array([1.9, 2.4]), 0)
  ValueError: The batch sizes of the tape operations do not match, they include 3 and 2.
  ```

  When creating a valid broadcasted tape, we can expand it into unbroadcasted tapes with
  the new `broadcast_expand` transform, and execute the three tapes independently.

  ```pycon
  >>> with qml.tape.QuantumTape() as tape:
  >>>     qml.apply(op)
  >>>     qml.RY(1.9, 0)
  >>>     qml.apply(op)
  >>>     qml.expval(qml.PauliZ(0))
  >>> tapes, fn = qml.transforms.broadcast_expand(tape)
  >>> len(tapes)
  3
  >>> dev = qml.device("default.qubit", wires=1)
  >>> fn(qml.execute(tapes, dev, None))
  array([-0.33003414, -0.34999899, -0.38238817])
  ```

  However, devices will handle this automatically under the hood:

  ```pycon
  >>> qml.execute([tape], dev, None)[0]
  array([-0.33003414, -0.34999899, -0.38238817])
  ```

* Boolean mask indexing of the parameter-shift Hessian
  [(#2538)](https://github.com/PennyLaneAI/pennylane/pull/2538)

  The `argnum` keyword argument for `param_shift_hessian`
  is now allowed to be a twodimensional Boolean `array_like`.
  Only the indicated entries of the Hessian will then be computed.
  A particularly useful example is the computation of the diagonal
  of the Hessian:

  ```python
  dev = qml.device("default.qubit", wires=1)
  with qml.tape.QuantumTape() as tape:
      qml.RX(0.2, wires=0)
      qml.RY(-0.9, wires=0)
      qml.RX(1.1, wires=0)
      qml.expval(qml.PauliZ(0))

  argnum = qml.math.eye(3, dtype=bool)
  ```
  ```pycon
  >>> tapes, fn = qml.gradients.param_shift_hessian(tape, argnum=argnum)
  >>> fn(qml.execute(tapes, dev, None))
  array([[[-0.09928388,  0.        ,  0.        ],
        [ 0.        , -0.27633945,  0.        ],
        [ 0.        ,  0.        , -0.09928388]]])
  ```

* Speed up measuring of commuting Pauli operators
  [(#2425)](https://github.com/PennyLaneAI/pennylane/pull/2425)

  The code that checks for qubit wise commuting (QWC) got a performance boost that is noticable
  when many commuting paulis of the same type are measured.

* Added the `qml.ECR` operation to represent the echoed RZX(pi/2) gate.
  [(#2613)](https://github.com/PennyLaneAI/pennylane/pull/2613)

* Added new transform `qml.batch_partial` which behaves similarly to `functools.partial` but supports batching in the unevaluated parameters.
  [(#2585)](https://github.com/PennyLaneAI/pennylane/pull/2585)

  This is useful for executing a circuit with a batch dimension in some of its parameters:

  ```python
  dev = qml.device("default.qubit", wires=1)

  @qml.qnode(dev)
  def circuit(x, y):
     qml.RX(x, wires=0)
     qml.RY(y, wires=0)
     return qml.expval(qml.PauliZ(wires=0))
  ```
  ```pycon
  >>> batched_partial_circuit = qml.batch_partial(circuit, x=np.array(np.pi / 2))
  >>> y = np.array([0.2, 0.3, 0.4])
  >>> batched_partial_circuit(y=y)
  tensor([0.69301172, 0.67552491, 0.65128847], requires_grad=True)
  ```

* The `default.mixed` device now supports backpropagation with the Autograd, TensorFlow, and PyTorch (CPU) interfaces.
  [(#2615)](https://github.com/PennyLaneAI/pennylane/pull/2615)
  [(#2670)](https://github.com/PennyLaneAI/pennylane/pull/2670)
  [(#2680)](https://github.com/PennyLaneAI/pennylane/pull/2680)

  As a result, the default differentiation method for the device is now `"backprop"`. To continue using the old default `"parameter-shift"`, explicitly specify this differentiation method in the QNode.

  ```python
  dev = qml.device("default.mixed", wires=2)

  @qml.qnode(dev, interface="autograd", diff_method="backprop")
  def circuit(x):
      qml.RY(x, wires=0)
      qml.CNOT(wires=[0, 1])
      return qml.expval(qml.PauliZ(wires=1))
  ```
  ```pycon
  >>> x = np.array(0.5, requires_grad=True)
  >>> circuit(x)
  array(0.87758256)
  >>> qml.grad(circuit)(x)
  -0.479425538604203
  ```

**Operator Arithmetic:**

* The adjoint transform `adjoint` can now accept either a single instantiated operator or
  a quantum function. It returns an entity of the same type/ call signature as what it was given:
  [(#2222)](https://github.com/PennyLaneAI/pennylane/pull/2222)
  [(#2672)](https://github.com/PennyLaneAI/pennylane/pull/2672)

  ```pycon
  >>> qml.adjoint(qml.PauliX(0))
  Adjoint(PauliX)(wires=[0])
  >>> qml.adjoint(lambda x: qml.RX(x, wires=0))(1.23)
  Adjoint(RX)(1.23, wires=[0])
  ```

  The adjoint now wraps operators in a symbolic operator class `qml.ops.op_math.Adjoint`. This class
  should not be constructed directly; the `adjoint` constructor should always be used instead.  The
  class behaves just like any other Operator:

  ```pycon
  >>> op = qml.adjoint(qml.S(0))
  >>> qml.matrix(op)
  array([[1.-0.j, 0.-0.j],
        [0.-0.j, 0.-1.j]])
  >>> qml.eigvals(op)
  array([1.-0.j, 0.-1.j])
  ```

* The `ctrl` transform and `ControlledOperation` have been moved to the new `qml.ops.op_math`
  submodule.  The developer-facing `ControlledOperation` class is no longer imported top-level.
  [(#2656)](https://github.com/PennyLaneAI/pennylane/pull/2656)

* A new symbolic operator class `qml.ops.op_math.Pow` represents an operator raised to a power.
  [(#2621)](https://github.com/PennyLaneAI/pennylane/pull/2621)

  ```pycon
  >>> op = qml.ops.op_math.Pow(qml.PauliX(0), 0.5)
  >>> op.decomposition()
  [SX(wires=[0])]
  >>> qml.matrix(op)
  array([[0.5+0.5j, 0.5-0.5j],
       [0.5-0.5j, 0.5+0.5j]])
  ```

* The unused keyword argument `do_queue` for `Operation.adjoint` is now fully removed.
  [(#2583)](https://github.com/PennyLaneAI/pennylane/pull/2583)

* Several non-decomposable `Adjoint` ops are added to the device test suite.
  [(#2658)](https://github.com/PennyLaneAI/pennylane/pull/2658)

* The developer-facing `pow` method has been added to `Operator` with concrete implementations
  for many classes.
  [(#2225)](https://github.com/PennyLaneAI/pennylane/pull/2225)

<h3>Improvements</h3>

* IPython displays the `str` representation of a `Hamiltonian`, rather than the `repr`. This displays
  more information about the object.
  [(#2648)](https://github.com/PennyLaneAI/pennylane/pull/2648)

* The qchem openfermion-dependent tests are localized and collected in `tests.qchem.of_tests`. The
  new module `test_structure` is created to collect the tests of the `qchem.structure` module in
  one place and remove their dependency to openfermion.
  [(#2593)](https://github.com/PennyLaneAI/pennylane/pull/2593)

* Test classes are created in qchem test modules to group the integrals and matrices unittests.
  [(#2545)](https://github.com/PennyLaneAI/pennylane/pull/2545)

* Introduced an `operations_only` argument to the `tape.get_parameters` method.
  [(#2543)](https://github.com/PennyLaneAI/pennylane/pull/2543)

* The `gradients` module now uses faster subroutines and uniform
  formats of gradient rules.
  [(#2452)](https://github.com/XanaduAI/pennylane/pull/2452)

* Wires can be passed as the final argument to an `Operator`, instead of requiring
  the wires to be explicitly specified with keyword `wires`. This functionality already
  existed for `Observable`'s, but now extends to all `Operator`'s.
  [(#2432)](https://github.com/PennyLaneAI/pennylane/pull/2432)

  ```pycon
  >>> qml.S(0)
  S(wires=[0])
  >>> qml.CNOT((0,1))
  CNOT(wires=[0, 1])
  ```

* Instead of checking types, objects are processed in `QuantumTape`'s based on a new `_queue_category` property.
  This is a temporary fix that will disappear in the future.
  [(#2408)](https://github.com/PennyLaneAI/pennylane/pull/2408)

* The `qml.taper` function can now be used to consistently taper any additional observables such as dipole moment,
  particle number, and spin operators using the symmetries obtained from the Hamiltonian.
  [(#2510)](https://github.com/PennyLaneAI/pennylane/pull/2510)

* The `QNode` class now contains a new method `best_method_str` that returns the best differentiation
  method for a provided device and interface, in human-readable format.
  [(#2533)](https://github.com/PennyLaneAI/pennylane/pull/2533)


* Using `Operation.inv()` in a queuing environment no longer updates the queue's metadata, but merely updates
  the operation in place.
  [(#2596)](https://github.com/PennyLaneAI/pennylane/pull/2596)

* Sparse Hamiltonians representation has changed from COOrdinate (COO) to Compressed Sparse Row (CSR) format. The CSR representation is more performant for arithmetic operations and matrix vector products. This change decreases the `expval()` calculation time, for `qml.SparseHamiltonian`, specially for large workflows. Also, the CRS format consumes less memory for the `qml.SparseHamiltonian` storage.
[(#2561)](https://github.com/PennyLaneAI/pennylane/pull/2561)

* A new method `safe_update_info` is added to `qml.QueuingContext`. This method is substituted
  for `qml.QueuingContext.update_info` in a variety of places.
  [(#2612)](https://github.com/PennyLaneAI/pennylane/pull/2612)
  [(#2675)](https://github.com/PennyLaneAI/pennylane/pull/2675)

* `BasisEmbedding` can accept an int as argument instead of a list of bits (optionally).
  [(#2601)](https://github.com/PennyLaneAI/pennylane/pull/2601)

  Example:

  `qml.BasisEmbedding(4, wires = range(4))` is now equivalent to
  `qml.BasisEmbedding([0,1,0,0], wires = range(4))` (because `4=0b100`).

* Introduced a new `is_hermitian` property to determine if an operator can be used in a measurement process.
  [(#2629)](https://github.com/PennyLaneAI/pennylane/pull/2629)

* Added separate requirements_dev.txt for separation of concerns for code development and just using PennyLane.
  [(#2635)](https://github.com/PennyLaneAI/pennylane/pull/2635)

* Add `IsingXY` gate.
  [(#2649)](https://github.com/PennyLaneAI/pennylane/pull/2649)

* The performance of building sparse Hamiltonians has been improved by accumulating the sparse representation of coefficient-operator pairs in a temporary storage and by eliminating unnecessary `kron` operations on identity matrices.
  [(#2630)](https://github.com/PennyLaneAI/pennylane/pull/2630)

* Control values are now displayed distinctly in text and mpl drawings of circuits.
  [(#2668)](https://github.com/PennyLaneAI/pennylane/pull/2668)

* The `TorchLayer`'s `init_method` argument now accepts either a `torch.nn.init` function or a dictionary which should specify a `torch.nn.init`/`torch.Tensor` for each different weight.
  [(#2678)](https://github.com/PennyLaneAI/pennylane/pull/2678)

<h3>Breaking changes</h3>

* Weights with negative shapes will raise an error, weights with `size = 0` will result in creating empty Tensor objects now with `qml.TorchLayer`.
  [(#2678)](https://github.com/PennyLaneAI/pennylane/pull/2678)


* PennyLane does not support TensorFlow `2.1.~` anymore.
  [(#2683)](https://github.com/PennyLaneAI/pennylane/pull/2683)

* The `qml.queuing.Queue` class is now removed.
  [(#2599)](https://github.com/PennyLaneAI/pennylane/pull/2599)

* The `qml.utils.expand` function is now removed; `qml.operation.expand_matrix` should be used instead.
  [(#2654)](https://github.com/PennyLaneAI/pennylane/pull/2654)
  
* The module `qml.gradients.param_shift_hessian` has been renamed to
  `qml.gradients.parameter_shift_hessian` in order to distinguish it from the identically named
  function. Note that the `param_shift_hessian` function is unaffected by this change and can be
  invoked in the same manner as before via the `qml.gradients` module.
  [(#2528)](https://github.com/PennyLaneAI/pennylane/pull/2528)
* The properties `eigval` and `matrix` from the `Operator` class were replaced with the
  methods `eigval()` and `matrix(wire_order=None)`.
  [(#2498)](https://github.com/PennyLaneAI/pennylane/pull/2498)

* `Operator.decomposition()` is now an instance method, and no longer accepts parameters.
  [(#2498)](https://github.com/PennyLaneAI/pennylane/pull/2498)

* Adds tests, adds no-coverage directives, and removes inaccessible logic to improve code coverage.
  [(#2537)](https://github.com/PennyLaneAI/pennylane/pull/2537)

* The base classes `QubitDevice` and `DefaultQubit` now accept data-types for a statevector. This
  enables a derived class (device) in a plugin to choose correct data-types.
  [(#2448)](https://github.com/PennyLaneAI/pennylane/pull/2448)

  ```pycon
  >>> dev = qml.device("default.qubit", wires=4, r_dtype=np.float32, c_dtype=np.complex64)
  >>> dev.R_DTYPE
  <class 'numpy.float32'>
  >>> dev.C_DTYPE
  <class 'numpy.complex64'>
  ```

<h3>Bug fixes</h3>

* Fixed a bug where returning `qml.density_matrix` using the PyTorch interface would return a density matrix with wrong shape.
  [(#2643)](https://github.com/PennyLaneAI/pennylane/pull/2643)

* Fixed a bug to make `param_shift_hessian` work with QNodes in which gates marked
  as trainable do not have any impact on the QNode output.
  [(#2584)](https://github.com/PennyLaneAI/pennylane/pull/2584)

* `QNode`'s now can interpret variations on the interface name, like `"tensorflow"`
  or `"jax-jit"`, when requesting backpropagation.
  [(#2591)](https://github.com/PennyLaneAI/pennylane/pull/2591)

* Fixed a bug for `diff_method="adjoint"` where incorrect gradients were
  computed for QNodes with parametrized observables (e.g., `qml.Hermitian`).
  [(#2543)](https://github.com/PennyLaneAI/pennylane/pull/2543)

* Fixed a bug where `QNGOptimizer` did not work with operators
  whose generator was a Hamiltonian.
  [(#2524)](https://github.com/PennyLaneAI/pennylane/pull/2524)

* Fixes a bug with the decomposition of `qml.CommutingEvolution`.
  [(#2542)](https://github.com/PennyLaneAI/pennylane/pull/2542)

* Fixed a bug enabling PennyLane to work with the latest version of Autoray.
  [(#2549)](https://github.com/PennyLaneAI/pennylane/pull/2549)

* Fixed a bug which caused different behaviour for `Hamiltonian @ Observable` and `Observable @ Hamiltonian`.
  [(#2570)](https://github.com/PennyLaneAI/pennylane/pull/2570)

* Fixes a bug in `DiagonalQubitUnitary._controlled` where an invalid operation was queued
  instead of the controlled version of the diagonal unitary.
  [(#2525)](https://github.com/PennyLaneAI/pennylane/pull/2525)

* Updated the gradients fix [(#2485)](https://github.com/PennyLaneAI/pennylane/pull/2485) to only apply to the `strawberryfields.gbs` device, since
  the original logic was breaking some devices. [(#2595)](https://github.com/PennyLaneAI/pennylane/pull/2595)

<h3>Deprecations</h3>

<h3>Documentation</h3>

* The centralized [Xanadu Sphinx Theme](https://github.com/XanaduAI/xanadu-sphinx-theme)
  is now used to style the Sphinx documentation.
  [(#2450)](https://github.com/PennyLaneAI/pennylane/pull/2450)

* Added reference to `qml.utils.sparse_hamiltonian` in `qml.SparseHamiltonian` to clarify
  how to construct sparse Hamiltonians in PennyLane.
  [(2572)](https://github.com/PennyLaneAI/pennylane/pull/2572)

* Added a new section in the [Gradients and Training](https://pennylane.readthedocs.io/en/stable/introduction/interfaces.html)
  page that summarizes the supported device configurations and provides justification. Also
  added [code examples](https://pennylane.readthedocs.io/en/stable/introduction/unsupported.html)
  for some selected configurations.
  [(#2540)](https://github.com/PennyLaneAI/pennylane/pull/2540)

* Added a note for the [Depolarization Channel](https://pennylane.readthedocs.io/en/stable/code/api/pennylane.DepolarizingChannel.html)
  that specifies how the channel behaves for the different values of depolarization probability `p`.
  [(#2669)](https://github.com/PennyLaneAI/pennylane/pull/2669)

<h3>Contributors</h3>

This release contains contributions from (in alphabetical order):

Amintor Dusko, Ankit Khandelwal, Avani Bhardwaj, Chae-Yeun Park, Christian Gogolin, Christina Lee, David Wierichs,
Edward Jiang, Guillermo Alonso-Linaje, Jay Soni, Juan Miguel Arrazola, Katharine Hyatt, Korbinian Kottmann,
Maria Schuld, Mason Moreland, Mikhail Andrenkov, Romain Moyard, Qi Hu, Samuel Banning, Soran Jahangiri, Utkarsh Azad, Antal Száva,
WingCode
