# Quantum Metric Learning with Reuploading

Hybrid quantum-classical framework for learning image embeddings using parametric quantum circuits with data reuploading. Combines CNNs with quantum computing to create discriminative representations for clustering and similarity search.

## 📋 Contents

- [Overview](#-overview)
- [Quick Start](#-quick-start)
- [How It Works](#-how-it-works)
- [Usage](#-usage)
- [Experiments](#-experiments)
- [Customization](#-customization)
- [Metrics](#-metrics)

---

## 🎯 Overview

This project explores **quantum circuits** for learning compact image representations. The system transforms images into embeddings where similar images are close and different images are distant in the learned space.

### Key Features

- **Hybrid Architecture**: CNN (dimensionality reduction) + Quantum Circuit (embedding generation)
- **Data Reuploading**: Features loaded into quantum circuits in multiple sequential layers
- **Two Architectures**: Standard (all features per layer) and Partial (sliding window)
- **26 Experiments**: 13 quantum architectures × 2 datasets (MNIST/FashionMNIST)
- **Triplet Loss Training**: Learns embeddings by minimizing anchor-positive distance while maximizing anchor-negative distance

---

## 🚀 Quick Start

### Installation

```bash
# Clone repository
git clone <repository-url>
cd Quantum-Metric-Learning-with-Reuploading

# Create virtual environment
python -m venv .venv

# Activate (Windows)
.\.venv\Scripts\Activate.ps1

# Activate (Linux/Mac)
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Launch Jupyter
jupyter notebook "Quantum Metric Learning.ipynb"
```

### Requirements

- Python 3.8+
- Qiskit ≥ 1.0.2 (quantum framework)
- PyTorch ≥ 2.0 (deep learning)
- scikit-learn ≥ 1.4.1 (metrics)
- GPU optional (CUDA for acceleration)

---

## 🔍 How It Works

### Pipeline

**1. Feature Extraction (CNN)**: Images (28×28 pixels) → Compact features (8 or 12 dimensions)

**2. Quantum Processing**: Features → Quantum circuit (encoder gates + variational ansatz + entanglement) → Probability distribution embedding

**3. Triplet Loss Learning**: Model receives triples (anchor, positive, negative) and learns to minimize `d(anchor, positive)` while maximizing `d(anchor, negative)`

### Architectures

**Standard Reuploading**: All features loaded in each layer. Allows repeated processing with different parameters for higher expressivity.

**Partial Reuploading**: Sliding window approach. Each layer loads different features on partially overlapping qubits (e.g., Layer 1: qubits 0-1 with features 0-3, Layer 2: qubits 1-2 with features 4-7). Processes more total features with fewer qubits.

### 26 Experiments

Test combinations of:

- Datasets: MNIST, FashionMNIST
- Reuploading: Standard vs Partial
- Encoders: YZ, RxRy, RxRz, RyRx, RzRx
- Ansatz: MPS (RealAmplitudes) vs TwoLocal
- Entanglement: With/without CNOT gates

---

## 🎮 Usage

### Three Independent Experiment Sets

The notebook uses three separate configuration variables, each with a specific purpose:

**`EXPERIMENTS_TO_RUN`**: Experiments for **expressivity analysis**. The system analyzes how well these quantum circuits can cover the Hilbert space before training.

**`EXPERIMENT_TO_TRAIN`**: Experiments to **train**. The system builds circuits, loads data, and trains these models. Checkpoints are automatically saved.

**`EXPERIMENT_TO_EVALUATE`**: Experiments to **evaluate and generate JSON results**. The system computes embeddings, performs clustering, calculates metrics (silhouette, purity, etc.), and saves results to `evaluation_results.json`.

### Key Constraints

- **All three sets are independent**: You can run different experiments in each category, or leave some empty
- **Only evaluation requirement**: To evaluate an experiment, it must have at least one checkpoint (trained weights) saved in its model_results folder
- **Train time logging**: To include training time and training loss in the evaluation JSON, you must train and evaluate the **same experiment in the same run**. If you train in one session and evaluate in another, training metrics won't be available in the JSON

### Basic Workflow

The notebook is designed to run **sequentially from top to bottom**.

**1. Setup**: Import libraries, set seed, check GPU availability

**2. Select Experiments**: Configure all three sets in "EXPERIMENT CONFIGURATION":

```python
# Expressivity analysis on all experiments
EXPERIMENTS_TO_RUN = EXPERIMENTS

# Train only MNIST experiments
EXPERIMENT_TO_TRAIN = filter_experiments(datasets=['MNIST'])

# Evaluate only specific experiments (must have checkpoints)
EXPERIMENT_TO_EVALUATE = [get_experiment(1), get_experiment(15)]
```

**3. Load Datasets**: Auto-loads based on all three configurations

**4. Expressivity Analysis**: Analyzes circuits from `EXPERIMENTS_TO_RUN`

**5. Training**: Trains experiments from `EXPERIMENT_TO_TRAIN`, saves checkpoints

**6. Evaluation**: Evaluates experiments from `EXPERIMENT_TO_EVALUATE`, generates `evaluation_results.json` with clustering metrics and (if trained in same run) training time and loss

### Common Scenarios

- **Full Pipeline**: Set all three to same experiments → Analyze expressivity → Train → Evaluate
- **Train Only**: Set `EXPERIMENT_TO_TRAIN`, leave `EXPERIMENTS_TO_RUN` and `EXPERIMENT_TO_EVALUATE` empty
- **Evaluate Existing**: Set only `EXPERIMENT_TO_EVALUATE` to models with checkpoints already trained
- **Analyze Then Train Selectively**: Set `EXPERIMENTS_TO_RUN` to all experiments → Train subset in `EXPERIMENT_TO_TRAIN`

---

## ⚙️ Experiments

### Experiment Structure

Each experiment defines:

- **ID & Name**: Unique identifier and descriptive name
- **Dataset**: MNIST or FashionMNIST
- **Circuit Type**: Standard or Partial reuploading
- **Circuit Parameters**: Features, encoder sequence, ansatz type, CNOT gates
- **Training Parameters**: Epochs, batch size, learning rate, optimizer, margin, checkpoint interval
- **Checkpoint Path**: Save location

### 13 Unique Architectures

**Standard** (6 variants): 8 features, full reuploading, varying encoder sequences and ansatz types

**Partial** (7 variants): 12 features (4/layer, 3 layers), sliding window, different fixed encoders and ansatz

Each architecture tested on both datasets = 26 total experiments.

### Adding Custom Experiment

Add dictionary to `EXPERIMENTS` list in notebook with all required parameters. System auto-detects and allows filtering by ID.

---

## 🛠️ Customization

### Modify Circuit Architecture

- **Layers**: Change encoder sequence length (standard) or `layers` parameter (partial)
- **Encoder Gates**: Choose from YZ, RxRy, RxRz, RyRx, RzRx combinations
- **Entanglement**: Toggle CNOT gates (increases expressivity but harder to simulate)
- **Custom Gates**: Modify circuit construction functions to add Hadamard, Toffoli, etc.

### Modify Hybrid Model

- **CNN Architecture**: Change filters, kernel sizes, add layers
- **Regularization**: Add dropout, batch normalization
- **Feature Count**: Adjust CNN output and quantum circuit input

### Modify Training

- **Loss Function**: Adjust triplet loss margin or implement custom losses
- **Optimizer**: Switch between SGD, Adam, AdamW, RMSprop
- **Learning Rate**: Implement schedulers or warmup
- **Early Stopping**: Adjust patience and minimum delta

### Use Different Datasets

- **Similar Format**: EMNIST, KMNIST work with minimal changes
- **Different Format**: Add transforms to convert to 28×28 grayscale
- **Data Augmentation**: Add random rotations, translations, flips

### Add Metrics

Include ARI, NMI, Davies-Bouldin Index, Calinski-Harabasz Score in evaluation functions.

---

## 📊 Metrics

### Clustering Metrics

| Metric                     | Range   | Interpretation                               |
| -------------------------- | ------- | -------------------------------------------- |
| **Silhouette Score** | [-1, 1] | Cluster cohesion/separation. Higher = better |
| **Purity**           | [0, 1]  | Cluster purity. 1 = perfect                  |
| **ARI**              | [-1, 1] | Clustering similarity (chance-corrected)     |
| **NMI**              | [0, 1]  | Shared information                           |

### Expressivity Metrics

| Metric                       | Description                                  |
| ---------------------------- | -------------------------------------------- |
| **Expressivity Score** | Hilbert space coverage uniformity (0-1)      |
| **Mean Fidelity**      | Average state overlap (lower = more diverse) |
| **Std Fidelity**       | Fidelity distribution spread                 |
| **Expr/Param**         | Expressivity normalized by parameters        |

### Interpreting Results

- **Good Embeddings**: Silhouette >0.4, Purity >0.7, well-separated t-SNE clusters
- **Overfitting**: High train metrics, low test metrics → Add regularization
- **Underfitting**: Low train and test metrics → Increase model capacity or training time

### Checkpoints

Auto-saved in `model_results/triplet_loss/<experiment_name>/`:

- `checkpoint_epoch_X.pt`: Model weights
- `evaluation_results.json`: Evaluation metrics

---

## 📝 Notes

**Performance**: Training is compute-intensive. Single epoch may take minutes even with GPU. Reduce batch size if OOM errors occur.

**Reproducibility**: Seed set globally for Python, NumPy, PyTorch, Qiskit. Minor variations possible across hardware/library versions.

**Limitations**: Quantum simulator (not real quantum hardware). Classical simulation limits qubit count due to exponential complexity.

---

## 📧 Contact

Open GitHub issue for questions or suggestions.

---

## 🙏 Acknowledgments

Built with:

- **Qiskit** - IBM quantum computing framework
- **PyTorch** - Deep learning framework
- **scikit-learn** - Machine learning library
