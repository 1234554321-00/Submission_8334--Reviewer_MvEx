# Submission_8334--Reviewer_MvEx
8334_When_Students_Surpass_Teachers


### Weakness 3: Large Number of Hyperparameters

We provide comprehensive sensitivity analysis and practical guidelines showing the framework is robust to hyperparameter choices:

Empirical Robustness Evidence:

From our sensitivity analysis (Appendix E, Figure 7):
- K-factor (α): Most critical parameter, but optimal range is wide ([0.3, 0.7]) with graceful degradation (5.7% range across extreme values)
- Curriculum parameters: Show low sensitivity (2.7-3.0% range) with robust default values
- Attention temperature: Moderate sensitivity (3.1% range) with clear optimal region
- Architectural parameters: Very high robustness (0.18-0.42 std dev) across wide ranges (Table 12)

Simplified Tuning Protocol:

We provide a two-tier hyperparameter system:

Tier 1 - Single Critical Parameter (requires tuning):
- K-factor α: Determined by dataset density
  - Dense datasets (ρ > 0.6): α = 0.3-0.5
  - Sparse datasets (ρ < 0.4): α = 0.5-0.7
  - Automatic selection: α = 0.5 × (1 + density_measure)

Tier 2 - Robust Defaults (rarely need tuning):
- Curriculum: α₀=0.8, β₀=0.2, γ=0.5
- Loss weights: λ₁(t)=0.5(t/T)^0.5, λ₂(t)=0.3exp(-t/T), λ₃=0.2
- Optimization: η=0.001, weight decay=10⁻⁴
- Temperature: τₙ=0.1, β=0.1

Comparison with Baselines:

| Method | Key Hyperparameters | Tuning Difficulty |
|--------|---------------------|-------------------|
| HyperGAT | Learning rate, dropout, attention heads, hidden dims, layers | 5 parameters |
| SSGNN | Learning rate, dropout, propagation steps, temperature, layers | 5 parameters |
| CuCoDistill | K-factor α (primary) + robust defaults | 1 primary parameter |

Practical Guidelines Provided:
- Table 13 (page 25): Dataset-specific recommendations
- Section E.8 (page 25): Step-by-step tuning protocol
- Figure 7 (page 29): Visual sensitivity analysis with optimal regions marked

---

## Response to Questions

### Question 1: Is Spectral Curriculum Necessary?

Yes, the spectral curriculum is essential despite having the smallest individual ablation impact. This apparent paradox reflects its role as a training stabilizer and accelerator rather than a performance ceiling raiser:

Three Critical Contributions:

1. Convergence Acceleration (Primary Benefit):
- Table 5 shows spectral curriculum achieves 2.2-2.3× faster convergence compared to training without curriculum
- Reaches 95% final performance in 89-112 epochs vs. 198-245 epochs without curriculum
- This halves total training time while maintaining or improving final performance

2. Training Stability (Prevents Failure Cases):
The 0.9-1.1% ablation impact represents *average* performance, but curriculum prevents catastrophic failures:
- Without curriculum, training exhibits 35% higher variance (Figure 7, right panel)
- Prevents early collapse on difficult examples that cause gradient explosion
- Enables successful training on datasets where random initialization would fail

3. Enables Co-evolutionary Training:
The curriculum's adaptive thresholds coordinate three simultaneous objectives:
```
L_total = λ₁(t)L_distill + λ₂(t)L_contrast + λ₃L_task
```
Without curriculum coordination:
- Distillation and contrastive objectives conflict, causing training instability
- Teacher and student can diverge rather than co-evolve
- Figure 5 (left panel) shows smooth phase transitions (stabilization → transfer → refinement) enabled by curriculum

Computational Efficiency:
- Curriculum overhead: O(|V| log |V|) per epoch (quantile computation)
- Represents <5% of total computation vs. attention mechanisms
- Cost is negligible compared to 2.2× convergence speedup benefit

Comparison with Simple Regularization:
We tested simple alternatives (fixed difficulty thresholds, random curriculum):
- Fixed thresholds: -1.5% to -2.7% performance (Figure 6, right panel)
- Random curriculum: -1.5% to -1.8% performance
- Weight decay alone: Does not provide temporal coordination of multiple objectives

Conclusion: The spectral curriculum is a high-impact, low-cost component that:
- Reduces training time by 2.2×
- Prevents training failures
- Enables stable co-evolutionary dynamics
- Adds minimal computational overhead (<5%)

The small ablation impact (0.9-1.1%) reflects that it works *behind the scenes* to enable other components to reach their full potential.

---

### Question 2: DBLP t-SNE Paradox - Why Degraded Clustering Leads to Better Accuracy?

This is a observation that highlights a fundamental distinction between visualization quality and task performance. The apparent paradox resolves when considering task-relevant feature emphasis:

1. Clustering Metric vs. Classification Objective Misalignment:

The silhouette score measures unsupervised clustering quality in the full embedding space:
```
Silhouette = (b - a) / max(a, b)
where a = intra-cluster distance, b = nearest-cluster distance
```

However, classification accuracy depends on task-relevant feature separation along decision boundaries, not global clustering quality.

2. Student Model's Selective Feature Emphasis:

The student model's information bottleneck (via top-K selection) performs implicit feature selection:
- Preserves: Task-discriminative features near classification boundaries
- Compresses: Task-irrelevant features that improve clustering aesthetics but not classification

Evidence from DBLP Analysis:

Looking at Table 2 (page 6), DBLP shows:
- Teacher accuracy: 87.2%
- Student accuracy: 87.8% (+0.6%)
- Dominant mechanism: Spectral regularization

The student's top-K constraint filters noisy author-venue connections while preserving essential paper-topic relationships for classification.

3. Visualization Artifacts:

t-SNE optimizes for local neighborhood preservation but can distort global structure:
- Teacher's high silhouette score (0.614) may reflect preservation of irrelevant hierarchical structure (author affiliations, venue prestige)
- Student's lower score (0.327) indicates compression of these decorative features
- Classification requires topic boundaries, not perfect author-venue cluster separation

Direct Evidence - Task-Specific Performance:

We analyzed feature importance for DBLP classification:
- Topic keywords: Critical for classification (student preserves 94% of teacher's attention)
- Co-authorship: Moderately useful (student preserves 67%)
- Venue information: Low importance (student preserves 41%)

The student strategically compresses low-utility features, degrading unsupervised clustering but improving supervised classification.

Analogous Phenomenon in Literature:

This mirrors findings in neural network compression literature:
- Hinton et al. (2015): Compressed models can outperform teachers by avoiding overfitting to training noise
- Information Bottleneck Theory (Tishby & Zaslavsky, 2015): Optimal representations compress task-irrelevant information

Mathematical Formulation:

Student optimizes: min I(X; Z_s) subject to I(Z_s; Y) ≥ I_min

This compression can reduce unsupervised quality metrics while improving task performance.

The DBLP t-SNE result is not a contradiction but a validation of our theoretical claim: the student's regularization mechanisms filter task-irrelevant structure, degrading unsupervised clustering aesthetics while sharpening task-relevant decision boundaries.

---

### Question 3: Citation Error in Baseline Description

Thank you for catching this error. You are absolutely correct. We have corrected Section D.2.2 (page 21):

"Hyper-SAGNN (Zhang et al., 2019b): A self-attention based graph neural network for hypergraphs that employs hierarchical attention to capture multi-scale patterns.

HyGCL-AdT (Qian et al., 2024): A dual-level hypergraph contrastive learning framework with adaptive temperature scaling. Their approach employs a hierarchical contrast mechanism that captures individual node behaviors in local contexts while simultaneously modeling group-wise interactions of nodes within hyperedges from a community perspective."

This clarifies that Zhang et al. (2019b) describes Hyper-SAGNN, while the subsequent description refers to HyGCL-AdT (Qian et al., 2024).

---

We hope these detailed clarifications fully address your concerns and more clearly highlight the novelty and practical value of our contributions. Given these explanations and the new evidence provided, we respectfully ask the reviewer to reconsider their evaluation and update the overall score accordingly.
