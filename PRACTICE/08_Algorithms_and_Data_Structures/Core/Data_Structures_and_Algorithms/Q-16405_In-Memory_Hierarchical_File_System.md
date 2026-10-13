---
type: "practice"
difficulty: "Other"
---

# In-Memory Hierarchical File System

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Trie](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Trie.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design a class that simulates a minimal in-memory file system supporting directories and text files organized under a single root /.

Implement the class with these operations:

FileSystem() — initialize an empty file system that contains only the root directory /.
list(path) —
If path names a directory, return the names of the files and sub-directories located directly inside it (one level only, not recursive), in ascending lexicographic order.
If path names a file, return a list containing only that file's own name.
makeDir(path) — create the directory given by path. Any missing intermediate directories along the path must also be created. The target directory is guaranteed not to already exist.
writeToFile(path, content) — if the file at path does not exist, create it holding content; if it already exists, append content to its current contents. Missing intermediate directories along the path are created as needed.
readFromFile(path) — return the full contents of the file at path.
Rules and guarantees
Every path is absolute: it begins with /. Only the root itself is written as /; no other path ends with /.
Directory and file names consist only of lowercase English letters, and two entries in the same directory never share a name.
All calls are given valid arguments: you will never be asked to list or readFromFile a path that does not exist. makeDir and writeToFile are the only operations allowed to create new directories/files (as described above).
Return
list returns a list of strings — lexicographically sorted names for a directory, or the single file name for a file. readFromFile returns the file's content string. makeDir and writeToFile return nothing.

#### Cons
traints
1 <= path.length <= 100
1 <= content.length <= 50
At most 300 total calls are made across list, makeDir, writeToFile, and readFromFile.

#### Example
s & test cases
TEXT
Input: On a freshly constructed FileSystem, call list("/") -> Output: []
Why: Nothing has been created yet, so the root directory has no entries. (Empty-root edge case.)

Input: makeDir("/m/n/o"); writeToFile("/m/n/o/p", "hello"); list("/"); readFromFile("/m/n/o/p") -> Output: list("/") returns ["m"]; readFromFile("/m/n/o/p") returns "hello"
Why: makeDir builds the whole chain m -> n -> o even though none existed; the file p is created with "hello"; the root's only direct child is the directory "m".

Input: writeToFile("/f", "abc"); writeToFile("/f", "de"); readFromFile("/f") -> Output: "abcde"
Why: The second write to an existing file appends rather than overwrites, giving "abc" + "de". (Append semantics.)

Input: writeToFile("/a/b/c", "x"); list("/a/b/c") -> Output: ["c"]
Why: The path "/a/b/c" resolves to a file, so list returns a single-element list of that file's own name, not a directory listing. (Listing a file path.)

Input: makeDir("/root/zeta"); makeDir("/root/alpha"); makeDir("/root/mid"); list("/root") -> Output: ["alpha", "mid", "zeta"]
Why: Directory entries are returned in ascending lexicographic order regardless of the order in which they were created. (Sorted-listing case.)

Input: makeDir("/a/b/c"); writeToFile("/a/b/c/d", "hi"); list("/a/b/c") -> Output: ["d"]
Why: After adding a file d under an existing directory c, listing c shows only its single direct child d. (File and directory coexisting under one node.)

#### Approach

Model the file system as a trie (prefix tree) over path components. Each node represents one directory or file and holds: a map from child name to child node, an isFile flag, and (for files) the accumulated content string. All four operations share a single helper that splits an absolute path on / (ignoring empty pieces) and walks node by node from the root, optionally creating missing nodes along the way — which is exactly the "create intermediate directories" requirement of makeDir and writeToFile. list distinguishes a file target (return just its own last-component name) from a directory (return its direct children). The only ordering requirement is that a directory listing be lexicographically sorted; you can satisfy this either by keeping children in an ordered map (C++ std::map, Java TreeMap, iterated in sorted order) or by storing them in a hash map and sorting the names on each list call (Python/JS/C) — both are correct since call volume is small. writeToFile appends to (never overwrites) existing content. Pattern: trie/tree-of-maps design.

Complexity

Let L be the number of components in a path (path length <= 100, so L is tiny), and let k be the number of entries in the directory being listed. makeDir, writeToFile, and readFromFile walk the path one component at a time, so they are O(L) per call when children are stored in a hash map (Python/JS) and O(L * log k) with an ordered tree map (C++/Java), plus O(A) to append content of length A in writeToFile. list is O(L + k log k) when sorting on read (Python/JS/C), or O(L * log k + k) when the children are already kept sorted in a balanced map (C++/Java). Space is O(total number of files + directories + total content length) — one trie node per path component ever created.

Python
PYTHON
from typing import List

class Node:
    __slots__ = ("children", "is_file", "content")
    def __init__(self):
        self.children = {}      # name -> Node
        self.is_file = False
        self.content = ""

class FileSystem:
    def __init__(self):
        self.root = Node()

    def _parts(self, path: str) -> List[str]:
        return [p for p in path.split("/") if p]

    def _walk(self, parts: List[str], create: bool = False):
        node = self.root
        for part in parts:
            if part not in node.children:
                if not create:
                    return None
                node.children[part] = Node()
            node = node.children[part]
        return node

    def list(self, path: str) -> List[str]:
        parts = self._parts(path)
        node = self._walk(parts)
        if node.is_file:
            return [parts[-1]]
        return sorted(node.children.keys())      # sort on read

    def makeDir(self, path: str) -> None:
        self._walk(self._parts(path), create=True)

    def writeToFile(self, path: str, content: str) -> None:
        node = self._walk(self._parts(path), create=True)
        node.is_file = True
        node.content += content                  # append

    def readFromFile(self, path: str) -> str:
        return self._walk(self._parts(path)).content
JavaScript
JAVASCRIPT
class FileNode {

#### cons
tructor() {
        this.children = new Map();   // name -> FileNode
        this.isFile = false;
        this.content = "";
    }
}

class FileSystem {

#### cons
tructor() {
        this.root = new FileNode();
    }

    _parts(path) {
        return path.split("/").filter(Boolean);
    }

    _walk(parts, create) {
        let node = this.root;
        for (const part of parts) {
            if (!node.children.has(part)) {
                if (!create) return null;
                node.children.set(part, new FileNode());
            }
            node = node.children.get(part);
        }
        return node;
    }

    list(path) {

#### cons
t parts = this._parts(path);

#### cons
t node = this._walk(parts, false);
        if (node.isFile) return [parts[parts.length - 1]];
        return [...node.children.keys()].sort();   // sort on read
    }

    makeDir(path) {
        this._walk(this._parts(path), true);
    }

    writeToFile(path, content) {

#### cons
t node = this._walk(this._parts(path), true);
        node.isFile = true;
        node.content += content;                   // append
    }

    readFromFile(path) {
        return this._walk(this._parts(path), false).content;
    }
}
Java
JAVA
import java.util.*;

class FileSystem {
    private static class Node {
        TreeMap<String, Node> children = new TreeMap<>();  // sorted keys -> lexicographic listing
        boolean isFile = false;
        StringBuilder content = new StringBuilder();
    }

    private final Node root = new Node();

    public FileSystem() {}

    private List<String> split(String path) {
        List<String> parts = new ArrayList<>();
        for (String s : path.split("/")) if (!s.isEmpty()) parts.add(s);
        return parts;
    }

    private Node walk(List<String> parts, boolean create) {
        Node node = root;
        for (String part : parts) {
            Node next = node.children.get(part);
            if (next == null) {
                if (!create) return null;
                next = new Node();
                node.children.put(part, next);
            }
            node = next;
        }
        return node;
    }

    public List<String> list(String path) {
        List<String> parts = split(path);
        Node node = walk(parts, false);
        if (node.isFile) return new ArrayList<>(List.of(parts.get(parts.size() - 1)));
        return new ArrayList<>(node.children.keySet());  // TreeMap keys are sorted
    }

    public void makeDir(String path) {
        walk(split(path), true);
    }

    public void writeToFile(String path, String content) {
        Node node = walk(split(path), true);
        node.isFile = true;
        node.content.append(content);   // append
    }

    public String readFromFile(String path) {
        return walk(split(path), false).content.toString();
    }
}
C++
CPP
#include <string>
#include <vector>
#include <map>
#include <memory>
#include <sstream>
using namespace std;

class FileSystem {
    struct Node {
        map<string, shared_ptr<Node>> children;  // ordered map -> lexicographic listing
        bool isFile = false;
        string content;
    };
    shared_ptr<Node> root = make_shared<Node>();

    static vector<string> split(const string& path) {
        vector<string> parts;
        stringstream ss(path);
        string part;
        while (getline(ss, part, '/'))
            if (!part.empty()) parts.push_back(part);
        return parts;
    }

    shared_ptr<Node> walk(const vector<string>& parts, bool create) {
        shared_ptr<Node> node = root;
        for (const string& part : parts) {
            auto it = node->children.find(part);
            if (it == node->children.end()) {
                if (!create) return nullptr;
                node->children[part] = make_shared<Node>();
                node = node->children[part];
            } else {
                node = it->second;
            }
        }
        return node;
    }

public:
    FileSystem() {}

    vector<string> list(const string& path) {
        vector<string> parts = split(path);
        shared_ptr<Node> node = walk(parts, false);
        if (node->isFile) return {parts.back()};
        vector<string> names;
        for (const auto& kv : node->children) names.push_back(kv.first);
        return names;  // map iteration is already sorted
    }

    void makeDir(const string& path) { walk(split(path), true); }

    void writeToFile(const string& path, const string& content) {
        shared_ptr<Node> node = walk(split(path), true);
        node->isFile = true;
        node->content += content;   // append
    }

    string readFromFile(const string& path) {
        return walk(split(path), false)->content;
    }
};
C
C
#include <stdlib.h>
#include <string.h>

typedef struct Node {
    char** names;          // child names (parallel array to kids)
    struct Node** kids;    // child nodes
    int count, cap;
    int isFile;
    char* content;         // NUL-terminated, grown on append
} Node;

typedef struct { Node* root; } FileSystem;

static Node* nodeCreate(void) {
    Node* n = calloc(1, sizeof(Node));
    n->content = calloc(1, 1);   // empty string
    return n;
}

FileSystem* fileSystemCreate(void) {
    FileSystem* fs = malloc(sizeof(FileSystem));
    fs->root = nodeCreate();
    return fs;
}

// Return existing child by name, or create it when create != 0, else NULL.
static Node* childGet(Node* node, const char* name, int create) {
    for (int i = 0; i < node->count; i++)
        if (strcmp(node->names[i], name) == 0) return node->kids[i];
    if (!create) return NULL;
    if (node->count == node->cap) {
        node->cap = node->cap ? node->cap * 2 : 4;
        node->names = realloc(node->names, node->cap * sizeof(char*));
        node->kids  = realloc(node->kids,  node->cap * sizeof(Node*));
    }
    node->names[node->count] = strdup(name);
    node->kids[node->count]  = nodeCreate();
    return node->kids[node->count++];
}

// Split an absolute path into components; returns the count, remembers the last name.
static int splitPath(const char* path, char parts[][64]) {
    int cnt = 0, len = 0;
    for (const char* p = path; ; p++) {
        if (*p == '/' || *p == '\0') {
            if (len > 0) { parts[cnt][len] = '\0'; cnt++; len = 0; }
            if (*p == '\0') break;
        } else {
            parts[cnt][len++] = *p;
        }
    }
    return cnt;
}

// Walk to the node named by path; create missing links when create != 0.
static Node* walk(FileSystem* fs, const char* path, int create, char* lastName) {
    char parts[64][64];
    int n = splitPath(path, parts);
    Node* node = fs->root;
    if (lastName) lastName[0] = '\0';
    for (int i = 0; i < n; i++) {
        node = childGet(node, parts[i], create);
        if (!node) return NULL;
        if (lastName) strcpy(lastName, parts[i]);
    }
    return node;
}

static int cmpStr(const void* a, const void* b) {
    return strcmp(*(const char* const*)a, *(const char* const*)b);
}

// Returns a lexicographically sorted list of names; caller frees each string and the array.
char** fileSystemList(FileSystem* fs, char* path, int* retSize) {
    char lastName[64];
    Node* node = walk(fs, path, 0, lastName);
    if (node->isFile) {
        char** res = malloc(sizeof(char*));
        res[0] = strdup(lastName);
        *retSize = 1;
        return res;
    }
    char** res = malloc((node->count ? node->count : 1) * sizeof(char*));
    for (int i = 0; i < node->count; i++) res[i] = strdup(node->names[i]);
    qsort(res, node->count, sizeof(char*), cmpStr);
    *retSize = node->count;
    return res;
}

void fileSystemMakeDir(FileSystem* fs, char* path) { walk(fs, path, 1, NULL); }

void fileSystemWriteToFile(FileSystem* fs, char* path, char* content) {
    Node* node = walk(fs, path, 1, NULL);
    node->isFile = 1;
    size_t oldLen = strlen(node->content), addLen = strlen(content);
    node->content = realloc(node->content, oldLen + addLen + 1);
    memcpy(node->content + oldLen, content, addLen + 1);   // append, keep NUL
}

char* fileSystemReadFromFile(FileSystem* fs, char* path) {
    return walk(fs, path, 0, NULL)->content;
}

#### Follow-up
questions
Add remove(path) to delete a file or an entire directory subtree, and move(src, dst) to rename/relocate a path.
Support relative paths and the special components . and .., resolving them during the walk.
Make the structure thread-safe for concurrent readers and writers; discuss per-node vs. global locking and the trade-offs.
Track and expose metadata (size, creation/modification time) and answer an efficient total-size (du) query over a subtree.

