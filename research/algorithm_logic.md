# 🧠 Algorithm Logic & Research Methodology

## 1. The Hallucination Problem
AI Hallucination in code generation refers to LLMs inventing non-existent package names. This occurs because LLMs predict the *next likely token* based on statistical probability rather than verifying the **factually correct library** against a registry.
* **The Vulnerability:** Attackers identify these frequently hallucinated names and "squat" on them by uploading malicious code to PyPI/NPM, turning a simple AI error into a remote code execution (RCE) vector.
* **Example:** An LLM may suggest `google-cloud-storage-v2` (hallucination) instead of the legitimate `google-cloud-storage`.

## 2. Detection Engine Architecture

### A. Typosquatting Detection (Levenshtein Distance)
We utilize the **Levenshtein Distance** algorithm to measure the mathematical "edit distance" between a scanned package and legitimate libraries.

* **The Formula:** The distance between two strings *a* and *b* (of length |*a*| and |*b*| respectively) is given by *lev(a, b)*:

$$
lev(a, b) = \begin{cases} 
|a| & \text{if } |b| = 0, \\
|b| & \text{if } |a| = 0, \\
lev(tail(a), tail(b)) & \text{if } a[0] = b[0], \\
1 + \min \begin{cases}
lev(tail(a), b) \\
lev(a, tail(b)) \\
lev(tail(a), tail(b))
\end{cases} & \text{otherwise.}
\end{cases}
$$

* **Implementation:** Our backend leverages the `rapidfuzz` library, which provides C++ optimized string matching for near-instantaneous processing of large repositories.
* **Strategic Thresholding:**
    * **Ratio >= 85%:** Flagged as **HIGH RISK**. This threshold identifies potential typosquatting (e.g., `reqests` vs `requests`) where the package name is dangerously similar to a known popular library.

### B. Registry Verification Protocol
We perform real-time lookups using the official PyPI and NPM JSON APIs to ensure the ground truth of a package's existence.
* **Registry Query:** `https://pypi.org/pypi/{package_name}/json`
* **Success (Status 200):** Package is verified as existing in the public ecosystem.
* **Failure (Status 404):** Package is flagged as a **High-Probability Hallucination**.

## 3. False Positive Mitigation
To ensure high **Efficiency of Approach**, we implemented a static **Context-Aware Whitelist**.
* **Standard Library Protection:** We filter out 200+ built-in modules (e.g., `os`, `sys`, `math`, `fs`, `path`) before analysis.
* **Internal Dependency Handling:** The system is designed to ignore relative imports (e.g., `from .module import X`), ensuring the scan focuses solely on external supply chain risks.

---
*Developed for Round 1: The Nest - Hack The Winter 2025*
