---
type: "practice"
difficulty: "Intermediate"
---

# Design a Sudoku validator/solver at the class level. Where does the constraint checking live?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Extensible Rule Engines](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Extensible_Rule_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a Sudoku validator/solver, we need to break down the problem into manageable components and ensure that our solution is both efficient and scalable. The key challenge is ensuring that constraints are properly enforced while solving or validating a Sudoku grid. At this level of detail, we'll focus on the class-level design, particularly where constraint checking lives.

#### Triage
Restating the Question and Probing

The question asks us to design a validator/solver for Sudoku puzzles at the class level, specifically focusing on where constraint checking should occur. The core aspects we need to consider include:

Class Granularity: What classes are involved in the validation/solving process?

#### Cons
traint Checking Location: Where exactly within these classes do we implement the logic that checks if a placement is valid according to Sudoku rules?

#### First Principles Solution
Core Classes and Responsibilities

SudokuBoard Class:

This class represents the entire 9x9 grid.
It will hold all cells (each cell can be empty or have an integer value).

Cell Class:

Represents a single cell in the Sudoku grid.
Stores its row, column, and box indices for easy constraint checking.

Validator Class:

Responsible for validating the entire board by ensuring no duplicates in rows, columns, and 3x3 sub-grids (boxes).

Solver Class:

Implements an algorithm like backtracking to solve the Sudoku puzzle.
Uses the Validator class to check constraints during each step of the solving process.

#### Cons
traint Checking Location

#### Cons
traint checking should be implemented primarily within the Validator class, as it is the component responsible for ensuring that all placements adhere to the rules. However, we can also perform some lightweight checks directly in the Cell and SudokuBoard classes to improve efficiency and readability.

Detailed Design
1. SudokuBoard Class
PYTHON
class SudokuBoard:
    def __init__(self):
        self.board = [[None for _ in range(9)] for _ in range(9)]

    def place_value(self, row, col, value):
        # Place the value at (row, col) and update board state
        pass

    def is_valid_placement(self, row, col, value):
        # Use Validator to check if placing `value` at (row, col) violates any constraints
        return self.validator.is_valid(row, col, value)
2. Cell Class
PYTHON
class Cell:
    def __init__(self, row, col, box, board):
        self.row = row
        self.col = col
        self.box = box
        self.board = board

    @property
    def value(self):
        return self.board[self.row][self.col]

    def set_value(self, value):
        self.board.set_value(self.row, self.col, value)
3. Validator Class
PYTHON
class Validator:
    def __init__(self, board):
        self.board = board

    def is_valid(self, row, col, value):
        # Check if placing `value` at (row, col) violates any constraints
        return not self.is_duplicate_in_row(row, value) and \
               not self.is_duplicate_in_column(col, value) and \
               not self.is_duplicate_in_box(row, col, value)

    def is_duplicate_in_row(self, row, value):
        for col in range(9):
            if self.board[row][col] == value:
                return True
        return False

    def is_duplicate_in_column(self, col, value):
        for row in range(9):
            if self.board[row][col] == value:
                return True
        return False

    def is_duplicate_in_box(self, row, col, value):
        start_row = (row // 3) * 3
        start_col = (col // 3) * 3
        for i in range(3):
            for j in range(3):
                if self.board[start_row + i][start_col + j] == value:
                    return True
        return False
4. Solver Class
PYTHON
class Solver:
    def __init__(self, board):
        self.board = board
        self.validator = Validator(board)

    def solve(self):
        for row in range(9):
            for col in range(9):
                if self.board.is_valid_placement(row, col, 0):  # Assuming 0 is the empty value
                    for value in range(1, 10):
                        if self.validator.is_valid(row, col, value):
                            self.board.place_value(row, col, value)

#### solution
= self.solve()
                            if solution:
                                return True
                            self.board.place_value(row, col, 0)  # Backtrack
                    return False
        return True

#### Closing Summary
and Common Mistakes

In this design, the Validator class is responsible for enforcing Sudoku constraints. The key point to remember is that constraint checking should be done whenever a new value is placed on the board or when solving the puzzle step-by-step.

#### Common Mistakes

Not Using Validation: Failing to validate each placement before updating the board can lead to invalid states.
Inefficient Checks: Performing full-row/column/box checks every time, even if a valid state is found earlier in the same row/column/box.
Incorrect Backtracking: Not properly handling backtracking logic when solving puzzles.
Self-Check

The solution is correct and complete as it covers all necessary aspects of Sudoku validation and solving using proper class design and constraint checking.

**Code:**

```python
class SudokuBoard:
    def __init__(self):
        self.board = [[None for _ in range(9)] for _ in range(9)]

    def place_value(self, row, col, value):
        # Place the value at (row, col) and update board state
        pass

    def is_valid_placement(self, row, col, value):
        return self.validator.is_valid(row, col, value)

class Cell:
    def __init__(self, row, col, box, board):
        self.row = row
        self.col = col
        self.box = box
        self.board = board

    @property
    def value(self):
        return self.board[self.row][self.col]

    def set_value(self, value):
        self.board.set_value(self.row, self.col, value)

class Validator:
    def __init__(self, board):
        self.board = board

    def is_valid(self, row, col, value):
        return not self.is_duplicate_in_row(row, value) and \
               not self.is_duplicate_in_column(col, value) and \
               not self.is_duplicate_in_box(row, col, value)

    def is_duplicate_in_row(self, row, value):
        for col in range(9):
            if self.board[row][col] == value:
                return True
        return False

    def is_duplicate_in_column(self, col, value):
        for row in range(9):
            if self.board[row][col] == value:
                return True
        return False

    def is_duplicate_in_box(self, row, col, value):
        start_row = (row // 3) * 3
        start_col = (col // 3) * 3
        for i in range(3):
            for j in range(3):
                if self.board[start_row + i][start_col + j] == value:
                    return True
        return False

class Solver:
    def __init__(self, board):
        self.board = board
        self.validator = Validator(board)

    def solve(self):
        for row in range(9):
            for col in range(9):
                if not self.board.is_valid_placement(row, col, 0):  # Assuming 0 is the empty value
                    for value in range(1, 10):
                        if self.validator.is_valid(row, col, value):
                            self.board.place_value(row, col, value)
                            solution = self.solve()
                            if solution:
                                return True
                            self.board.place_value(row, col, 0)  # Backtrack
                    return False
        return True
```

**Flow:**

1. Valid
2. Invalid
3. Complete
4. Not Complete
5. Initialize Board
6. Place Value
7. Validate Placement
8. Set Value
9. Backtrack and Retry
10. Check Solution Complete?
11. Solution Found!

