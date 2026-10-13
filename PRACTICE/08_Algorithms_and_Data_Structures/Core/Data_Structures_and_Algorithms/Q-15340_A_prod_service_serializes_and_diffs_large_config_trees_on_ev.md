---
type: "practice"
difficulty: "Deep"
---

# A prod service serializes and diffs large config trees on every reload and CPU spikes; how would you use a Merkle-tree hash to skip unchanged subtrees and where does the hashing cost pay off?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Diff and LCS Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Diff_and_LCS_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of CPU spikes during config tree reloads, we can leverage a Merkle tree hash to identify and skip unchanged subtrees. The core idea is that instead of recomputing the full hash for every subtree, we only need to recompute the hashes for those parts that have changed. This reduces redundant work and optimizes the diffing process.

#### Triage

The question is essentially asking how to use Merkle tree hashing to optimize the comparison of large config trees. The interviewer is probing your understanding of data structures, hash functions, and the trade-offs between computation cost and efficiency gains. We need to make some assumptions:

Tree Structure: The config tree is a hierarchical structure where nodes represent configuration settings.
Hash Function: A suitable hash function (like SHA-256) will be used to generate unique hashes for each node.

#### First Principles Solution
Step 1: Node Hashing

First, we need to compute the hash of every node in the initial state of the config tree. This involves traversing the entire tree and hashing each node’s content (or a serialized representation).

PYTHON
def hash_node(node):
    # Serialize node or use its direct value if it's simple enough
    node_content = serialize_node(node)
    return hashlib.sha256(node_content.encode()).hexdigest()
Step 2: Merkle Tree Construction

Next, we construct the Merkle tree. Each leaf in the tree represents a node from the original config tree. Non-leaf nodes are hashes of their children.

PYTHON
def build_merkle_tree(leaves):
    while len(leaves) > 1:
        new_leaves = [(leaves[i], leaves[i + 1]) for i in range(0, len(leaves), 2)]
        if (len(leaves) % 2) != 0:
            new_leaves.append((leaves[-1], None))

        new_hashes = [hashlib.sha256(f"{left}{right}".encode()).hexdigest() for left, right in new_leaves]
        leaves = new_hashes

    return leaves[0] if leaves else None
Step 3: Tree Diffing with Merkle Root Comparison

When the config tree is reloaded, we again compute its hash and compare it with the previously computed root of the Merkle tree. If they differ, we know that at least one subtree has changed.

PYTHON
def reload_config_tree(config_tree):
    new_root = build_merkle_tree([hash_node(node) for node in config_tree])

    if new_root != current_root:
        # Only recompute hashes of nodes with changed subtrees and compare them
        diffs = find_changed_subtrees(config_tree, new_root)

        return process_diffs(diffs)
    else:
        print("No changes detected.")
Step 4: Implementing Subtree Diffing

We implement a function to identify the specific subtree(s) that have changed by recursively comparing nodes and their hashes.

PYTHON
def find_changed_subtrees(node, new_root):
    if node_hash[node] != node_hash[new_root]:
        return [node]

    children = list(node.children)  # Assuming each node has a .children attribute

    for child in children:
        result = find_changed_subtrees(child, new_root)
        if result:
            return result + [node]

    return []

#### Closing Summary

Using a Merkle tree to hash config subtrees allows us to efficiently identify and process only those parts that have changed. This reduces the computational overhead significantly by avoiding unnecessary recomputation of unchanged subtrees.

#### Common Mistakes/Follow-Ups

#### Assumptions
on Tree Structure: Ensure the tree structure is well-defined and all nodes are properly serialized.
Hash Function Choice: Use a secure, fast hash function that suits your needs.
Edge Cases: Handle cases where node content changes but the overall tree remains structurally the same.
Performance Trade-offs: While Merkle trees save computation time, they introduce additional overhead in building and maintaining the tree.

