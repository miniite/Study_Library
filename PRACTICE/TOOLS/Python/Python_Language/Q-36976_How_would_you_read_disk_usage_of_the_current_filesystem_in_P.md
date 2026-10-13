---
type: "practice"
difficulty: "Intermediate"
---

# How would you read disk usage of the current filesystem in Python and warn if free space drops below a threshold?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Subprocess Safety](<../../../../TOOLS/Python/Python_Language/Subprocess_Safety.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To monitor disk usage in Python, we need to interact with the operating system's utilities or APIs to fetch the required information. The primary task is to parse the output of df (disk free) command and check if the free space falls below a certain threshold. This problem involves understanding file operations, process execution, and handling command-line outputs in Python.

We assume that:

We are using Linux or Unix-like systems where the df command is available.
The threshold value for warning is provided as input to our script.
We want to log any warnings to a file or print them directly to the console.

Let's break down the solution step-by-step:

Execute the df -h / command to get the disk usage of the root filesystem in human-readable format.
Parse the output to extract the free space information.
Compare the extracted free space against the threshold.
Log or print a warning message if the condition is met.

Now, let's implement this in Python:

PYTHON
import subprocess

def check_disk_usage(threshold):
    """
    Checks the disk usage of the root filesystem and logs a warning if it falls below the specified threshold.

    :param threshold: A float representing the free space threshold in GB.
    """
    try:
        # Execute df -h / command
        result = subprocess.run(['df', '-h', '/'], stdout=subprocess.PIPE, check=True)

        # Parse the output
        lines = result.stdout.decode('utf-8').strip().split('\n')
        header = lines[0].split()

        for line in lines[1:]:
            if 'root' in line:  # Check for root filesystem
                fields = line.split()
                free_space_human = fields[3]
                break

        # Convert free space to GB (assuming the output is in G or M)
        free_space_gb = float(free_space_human[:-1]) * (0.001 if free_space_human[-1] == 'M' else 1)

        # Check against threshold
        if free_space_gb < threshold:
            print(f"Warning: Free space on root filesystem is below {threshold} GB")

    except subprocess.CalledProcessError as e:
        print(f"Failed to execute df command: {e}")
    except Exception as e:
        print(f"An error occurred: {e}")

# Example usage
check_disk_usage(10)

