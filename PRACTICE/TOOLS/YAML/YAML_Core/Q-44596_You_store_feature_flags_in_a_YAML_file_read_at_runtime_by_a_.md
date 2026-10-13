---
type: "practice"
difficulty: "Deep"
---

# You store feature flags in a YAML file read at runtime by a service. What concurrency, hot-reload, and schema-versioning concerns shape how you design that YAML and its loader?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Linting YAML](<../../../../TOOLS/YAML/YAML_Core/Linting_YAML.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the concurrency, hot-reload, and schema-versioning concerns when storing feature flags in a YAML file read at runtime by a service, we need to ensure our approach is robust, scalable, and adaptable. Here’s how these concerns shape the design:

Concurrency: With multiple services potentially reading from the same YAML file, concurrent access must be handled carefully to avoid race conditions.
Hot-reload: The ability to update feature flags without restarting the service ensures minimal downtime and allows for quick A/B testing and deployments.
Schema-versioning: Managing changes in the structure of the YAML over time is crucial to maintain backward compatibility.

#### Triage
What the Question is Asking

The question revolves around designing a system where feature flags are stored in a YAML file, with mechanisms to handle concurrency, enable hot-reloading, and manage schema evolution. The interviewer is probing for an understanding of how these requirements interplay and affect the overall architecture.

#### First-Principles Solution
1. Concurrency Management

To ensure safe concurrent access to the YAML file, we can use a read-only lock or a concurrent map implementation that supports thread-safe operations. For example, in Go, you could use sync.RWMutex:

GO
var flags sync.Map // Using sync.Map for thread-safe read and write operations

func loadFlags(yamlFile string) {
    var mutex sync.Mutex
    content, err := ioutil.ReadFile(yamlFile)
    if err != nil {
        log.Fatal(err)
    }
    yaml.Unmarshal(content, &flags)
}
2. Hot-Reload Mechanism

To support hot-reload without service restarts, we need to periodically check for changes in the YAML file and reload it when necessary. One approach is using a file watcher or a dedicated monitoring service like inotifywait on Unix systems.

BASH
while true; do
    inotifywait -e modify path/to/flags.yaml && loadFlags(path/to/flags.yaml)
done

In Go, this can be implemented with the fsnotify package:

GO
package main

import (
	"log"
	"os"
	"time"

	"github.com/fsnotify/fsnotify"
)

func main() {
	// Watch the YAML file for changes
	watcher, err := fsnotify.NewWatcher()
	if err != nil {
		log.Fatal(err)
	}
	defer watcher.Close()

	done := make(chan bool)
	go func() {
		for {
			select {
			case event, ok := <-watcher.Events:
				if !ok {
					return
				}
				if event.Name == "path/to/flags.yaml" {
					loadFlags("path/to/flags.yaml")
				}
			case err, ok := <-watcher.Errors:
				if !ok {
					return
				}
				log.Println("error:", err)
			}
		}
	}()

	err = watcher.Add("path/to/flags.yaml")
	if err != nil {
		log.Fatal(err)
	}

	<-done
}
3. Schema-Versioning Strategy

To manage schema changes, we can employ a versioned schema approach where each feature flag has a version number and backward compatibility rules are defined.

YAML
features:
  - name: betaFeature
    enabled: true
    version: 1 # Version for BC check

  - name: experimentalFeature
    enabled: false
    version: 2

When loading the YAML, we can map new versions to old ones:

GO
func loadFlags(yamlFile string) {
	content, err := ioutil.ReadFile(yamlFile)
	if err != nil {
		log.Fatal(err)
	}
	var flags map[string]featureFlag

	err = yaml.Unmarshal(content, &flags)
	if err != nil {
		log.Fatal(err)
	}

	for name, flag := range flags {
		oldFlag, exists := knownFlags[name]
		if !exists || oldFlag.version < flag.version {
			knownFlags[name] = flag // Update the versioned map
		}
	}
}

#### Closing Summary
and Common Mistakes

In summary, designing a system to handle concurrency, hot-reload, and schema-versioning for feature flags in YAML involves careful implementation of thread-safe access, file monitoring, and dynamic schema evolution. Common mistakes include:

Not using proper locking mechanisms: This can lead to race conditions.
Failing to implement hot-reload correctly: Services may experience unexpected behavior or crashes.
Ignoring backward compatibility during schema changes: This can cause runtime errors in services.

To ensure robustness, always validate and test the system under different scenarios, including concurrent access and schema transitions.

**Code:**

```go
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"sync"
)

var knownFlags = map[string]featureFlag{}

type featureFlag struct {
	Name     string `yaml:"name"`
	Enabled  bool   `yaml:"enabled"`
	Version  int    `yaml:"version"`
}

func loadFlags(yamlFile string) {
	var mutex sync.Mutex
	content, err := ioutil.ReadFile(yamlFile)
	if err != nil {
		log.Fatal(err)
	}
	err = yaml.Unmarshal(content, &knownFlags)
	if err != nil {
		log.Fatal(err)
	}

	mutex.Lock()
	for name, flag := range knownFlags {
		oldFlag, exists := map[string]featureFlag(name: "legacyName", enabled: false, version: 1)[name]
		if !exists || oldFlag.version < flag.version {
			map[string]featureFlag(name: "legacyName", enabled: false, version: 1)[name] = flag
		}
	}
	mutex.Unlock()
}

func main() {
	loadFlags("path/to/flags.yaml")
	fmt.Println(knownFlags)
}
```

