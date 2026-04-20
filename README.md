# Self-Pruning Neural Network using Learnable Gates

## Overview

This project implements a **self-pruning neural network** using a custom `PrunableLinear` layer.
The network learns not only the weights but also **which connections are necessary**, enabling automatic pruning during training.

The key idea is to associate each weight with a **learnable gate** that controls whether the connection is active or suppressed.

---

## 1. PrunableLinear Layer

Instead of using the standard linear layer, we define a custom layer where each weight has a corresponding **gate value**:

[
\text{output} = (W \odot \sigma(G)) \cdot x + b
]

Where:

* ( W ) = weight matrix
* ( G ) = gate scores (learnable parameters)
* ( \sigma(G) ) = sigmoid function → converts scores into values between 0 and 1
* ( \odot ) = element-wise multiplication

### Key Idea

* If gate ≈ 1 → connection is active
* If gate ≈ 0 → connection is effectively removed

This allows the network to **learn which weights are important**.

---

## 2. Sparsity Regularization (L1 Loss)

To encourage pruning, we add an **L1 penalty on gate values**:

[
\text{Total Loss} = \text{CrossEntropyLoss} + \lambda \cdot \sum \text{gates}
]

### Why L1 Works

* L1 applies a **constant pressure toward zero**
* Even small gate values are pushed further down
* This leads to **exact or near-zero gates**
* Result: **sparse network**

Unlike L2 regularization, which only shrinks values, L1 actively promotes **true sparsity**.

---

## 3. Training Setup

* **Dataset:** CIFAR-10
* **Model:** Fully connected neural network using `PrunableLinear` layers
* **Optimizer:** Adam
* **Loss:** CrossEntropy + λ × Sparsity Loss
* **Epochs:** 10

---

## 4. Evaluation Metrics

### 1. Test Accuracy

Measures classification performance on unseen data.

### 2. Sparsity Level

[
\text{Sparsity} = \frac{\text{Number of gates} < 10^{-2}}{\text{Total gates}} \times 100
]

This indicates how many connections have been effectively pruned.

---

## 5. Experimental Results

| Lambda (λ) | Test Accuracy (%) | Sparsity Level (%) |
| ---------- | ----------------- | ------------------ |
| 1e-6       | 55.97%            | 0.01%              |
| 1e-5       | 55.71%            | 0.43%              |
| 1e-4       | 56.16%            | 1.45%              |

---

## 6. Analysis of λ Trade-off

The parameter λ controls the balance between:

* **Model performance (accuracy)**
* **Model compression (sparsity)**

### Observations

* **Low λ (1e-6):**

  * Sparsity penalty is negligible
  * Model behaves like a standard neural network
  * Almost no pruning occurs

* **Moderate λ (1e-5):**

  * Slight increase in sparsity
  * Minimal impact on accuracy
  * Model begins removing unimportant connections

* **Higher λ (1e-4):**

  * Increased pruning effect
  * Sparsity improves further
  * Accuracy remains stable

### Key Insight

The results demonstrate the **trade-off between sparsity and accuracy**:

* Increasing λ increases pruning pressure
* However, in this setup, sparsity remains relatively low
* This indicates that the regularization strength is still modest

### Important Conclusion

> The network successfully learns to prune itself, but only a small fraction of weights are removed under the current λ values. Stronger regularization would result in more aggressive pruning.

---

## 7. Gate Distribution Visualization

The histogram of gate values shows how the network distributes importance across connections.

### Expected Behavior

* Values near **0** → pruned connections
* Values away from 0 → important connections

In this experiment:

* Most gates remain away from zero
* This aligns with the low sparsity values observed

---

## 8. Gradient Flow Correctness

The entire pruning mechanism is fully differentiable:

* Gradients flow through:

  * weights ( W )
  * gate scores ( G )
* Sigmoid ensures smooth gradient propagation
* No custom backward implementation is required

This confirms that the model correctly learns both:

* **what to learn (weights)**
* **what to remove (gates)**

---

## 9. Conclusion

This project demonstrates a **differentiable pruning mechanism** using learnable gates and L1 regularization.

### Key Takeaways

* Custom `PrunableLinear` successfully integrates gating mechanism
* L1 regularization encourages sparsity
* The network can prune itself during training
* λ controls the sparsity–accuracy trade-off

Although pruning is currently limited, the framework is fully functional and can be extended to achieve higher sparsity with stronger regularization or deeper architectures.

---

## 10. Future Improvements

* Increase λ for stronger pruning
* Use convolutional layers for better performance
* Apply structured pruning (neurons instead of weights)
* Fine-tune after pruning for accuracy recovery

---

**Final Remark:**
This approach provides a simple yet powerful foundation for **model compression and efficient deep learning systems**.
