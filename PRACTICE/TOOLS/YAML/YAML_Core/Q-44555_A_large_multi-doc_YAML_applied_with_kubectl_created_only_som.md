---
type: "practice"
difficulty: "Intermediate"
---

# A large multi-doc YAML applied with kubectl created only some resources. How do you determine whether a stray '---' or an empty document caused the parser to skip resources?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kustomize Patching](<../../../../TOOLS/YAML/YAML_Core/Kustomize_Patching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a situation where kubectl fails to apply all expected resources from a multi-doc YAML file, we need to pinpoint the exact cause. The two common culprits are either an extraneous --- (which is used as a document separator in YAML) or an empty document that causes the parser to skip subsequent documents. Here’s how you can systematically approach this problem.

Firstly, understand that kubectl apply -f <filename> applies each document independently. If any document fails validation or contains errors, kubectl will log an error and move on to the next one. This means it's crucial to ensure that every document is well-formed.

To diagnose whether a stray --- or an empty document caused the issue, follow these steps:

Review YAML Syntax: Ensure each document in your multi-doc YAML file is properly formatted. A document should start with indentation and end with a closing brace or sequence.
Check for Trailing ---: A trailing --- will cause the parser to ignore any remaining documents, as it signifies the end of the current document stream. Remove unnecessary --- to ensure all documents are processed.
Identify Empty Documents: An empty document (a single ---) will be ignored by the YAML parser and subsequent documents won't get applied.

Now let's walk through a step-by-step solution:

Step 1: Validate Each Document Individually

You can validate each document separately using tools like yq or an online YAML validator. For instance, if your multi-doc YAML is in a file named resources.yaml, you can split it into individual documents and test them one by one.

SH
# Split the multi-doc YAML into single-doc YAML files
cat resources.yaml | tr -d '\n' | sed 's/---/\n---/g' > temp.yaml

# List of document names after splitting
ls *.yaml
Step 2: Test Each Document with kubectl apply

Once you have separated the documents, test each one individually:

SH
for doc in $(ls *.yaml); do
    echo "Applying $doc"
    kubectl apply -f "$doc" --dry-run=client | yq e .
done
Step 3: Review kubectl Logs

If a document fails, check the logs for specific error messages:

SH
kubectl get pods -o json | jq '.items[] | select(.metadata.name == "pod-name")'

This will give you insights into why that particular pod or resource failed to apply.

Step 4: Debugging and Fixing

If an empty document is causing issues, remove it. If a trailing --- is the problem, ensure all documents are properly terminated:

SH
# Fixing by removing extraneous ---
cat resources.yaml | sed '/---$/d' > fixed-resources.yaml

kubectl apply -f fixed-resources.yaml
Step 5: Revert and Test Again

After fixing any issues, revert back to your original resources.yaml file and re-run the application command:

SH
kubectl apply -f resources.yaml

By following these steps, you can systematically identify whether a stray --- or an empty document is causing the issue in your YAML files.

#### Common Mistakes
Overlooking Trailing ---: Always ensure there are no trailing --- at the end of your multi-doc YAML file.
Assuming All Documents Are Correct: Even if one document fails, kubectl might still apply other documents. Ensure each document is independently validated.

