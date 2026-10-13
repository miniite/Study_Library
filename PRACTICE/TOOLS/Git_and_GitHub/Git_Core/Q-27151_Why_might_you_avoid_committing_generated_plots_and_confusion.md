---
type: "practice"
difficulty: "Intermediate"
---

# Why might you avoid committing generated plots and confusion-matrix images to Git, and what would you do instead?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Versioning Experiments with Git](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_Experiments_with_Git.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working as an ML engineer, it's common to generate various visualizations like plots and confusion matrices during the development process. These artifacts can be valuable for debugging, presenting results, or understanding model behavior. However, committing generated plots and confusion-matrix images directly into Git has several downsides that we might want to avoid.

Firstly, these files are often large in size, which increases the repository’s footprint unnecessarily. Secondly, visualizations can change significantly as experiments progress, making it hard to track changes meaningfully through version control. Lastly, images generated on one machine may look different when rendered on another due to differences in font rendering or display settings, leading to inconsistencies.

Instead of committing these files directly into Git, we should consider using a more structured approach that leverages the strengths of Git while maintaining the benefits of visualizations for debugging and presentation purposes. Here’s what you can do:

Document the Process: Commit the code that generates the plots and confusion matrices. This way, future developers or yourself in the future will know exactly how these visualizations were created.
Store the Raw Data: Instead of committing images directly, commit the raw data used to generate them. Tools like Jupyter Notebooks can be useful for this purpose as they allow you to keep your code and data together with the visualizations.

Let's dive into a practical example using Python and Jupyter Notebooks:

PYTHON
# Example of generating plots in a Jupyter Notebook
import matplotlib.pyplot as plt

def plot_confusion_matrix(confusion_matrix, class_names):
    plt.figure(figsize=(10, 8))
    sns.heatmap(confusion_matrix, annot=True, fmt='g', cmap="viridis")
    plt.title("Confusion Matrix")
    plt.ylabel('True label')
    plt.xlabel('Predicted label')
    plt.show()

# Generate and display the confusion matrix
confusion_matrix = np.array([[10, 2], [1, 9]])
plot_confusion_matrix(confusion_matrix, ["Class A", "Class B"])

To store this in Git without committing the image:

PYTHON
# Save the raw data to a file
np.savetxt("confusion_matrix.txt", confusion_matrix)

def plot_confusion_matrix(confusion_matrix_path, class_names):
    confusion_matrix = np.loadtxt(confusion_matrix_path)
    plt.figure(figsize=(10, 8))
    sns.heatmap(confusion_matrix, annot=True, fmt='g', cmap="viridis")
    # Rest of the plotting code remains the same

# Generate and save the plot
plot_confusion_matrix("confusion_matrix.txt", ["Class A", "Class B"])

By storing only the raw data in Git, we ensure that our repository stays lightweight and avoids unnecessary bloat. When needed, we can regenerate the plots using the saved code.

This approach not only keeps your Git repository clean but also makes it easier to reproduce results across different environments.

**Code:**

```python
import matplotlib.pyplot as plt
import numpy as np
import seaborn as sns

def plot_confusion_matrix(confusion_matrix, class_names):
    plt.figure(figsize=(10, 8))
    sns.heatmap(confusion_matrix, annot=True, fmt='g', cmap="viridis")
    plt.title("Confusion Matrix")
    plt.ylabel('True label')
    plt.xlabel('Predicted label')
    plt.show()

# Generate and display the confusion matrix
confusion_matrix = np.array([[10, 2], [1, 9]])
plot_confusion_matrix(confusion_matrix, ["Class A", "Class B"])

# Save the raw data to a file
np.savetxt("confusion_matrix.txt", confusion_matrix)

def plot_confusion_matrix(confusion_matrix_path, class_names):
    confusion_matrix = np.loadtxt(confusion_matrix_path)
    plt.figure(figsize=(10, 8))
    sns.heatmap(confusion_matrix, annot=True, fmt='g', cmap="viridis")
    # Rest of the plotting code remains the same

# Generate and save the plot
plot_confusion_matrix("confusion_matrix.txt", ["Class A", "Class B"])
```

**Example input / output:**

```

To demonstrate the process, let’s assume we have a confusion matrix generated and saved as confusion_matrix.txt. We will commit this file along with the code that generates it.

PLAINTEXT
Copy
# Commit message
$ git add .
$ git commit -m "Add raw data for confusion matrix generation"

When a colleague or future you wants to regenerate these plots, they can simply run the following commands:

BASH
Copy
$ git checkout <commit-hash>
$ python3 plot_confusion_matrix.py

This approach ensures that only necessary files are committed and that visualizations can be generated consistently across different environments.
```

**Data:**

```plaintext
"confusion_matrix.txt"
10 2
1 9

This data would be committed to Git.
```

