---
type: "practice"
difficulty: "Deep"
---

# In the chess design where does move-validation logic belong and why?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Extensible Rule Engines](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Extensible_Rule_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

This is the focused follow-up to the "design a chess game" question: in the chess design, where does move-validation logic belong, and why? The interviewer is no longer testing whether I can list classes — they're testing judgment about responsibility placement (SRP), cohesion, and coupling. The trap answers are the two extremes: (a) shove all validation into the Piece (so Pawn somehow decides check), or (b) shove all of it into one giant Board.isValid() god-method with a switch(pieceType). Both are wrong. The correct answer is that validation is layered by knowledge scope, and each layer lives with the object that has the knowledge to answer it.

Clarifying framing I'd state: "validate a move" is not one decision — it's a pipeline of decisions, each needing a different amount of context. The right home for each decision is "the smallest object that already owns the data that decision requires." That single principle drives the whole answer.

Plan: define the layers of validation; map each to its owner (Piece → Board → Game) with the why; justify with SRP/cohesion/coupling and the Open/Closed Principle; show the anti-patterns and why they hurt; note the one subtle case (king-safety) that must be at the Board level.

The layers of "is this move valid?"

A move passes several independent checks. Listing them exposes that they need different context:

Turn/ownership — is it this player's turn; does from hold a piece of that color? (needs game state)
Shape/geometry — does a bishop move diagonally, a knight in an L? (needs only the piece's own rule)
Path & occupancy — is the sliding path clear; is to empty or an enemy? (needs the whole board)
King safety / legality — does the move leave my own king in check (pins, escaping check)? (needs the whole board + the king's location + the opponent's attacks)
Special-move context — castling rights, en passant, promotion (needs history + multiple squares)
Terminal status — checkmate/stalemate after the move (needs all the player's legal moves)
Where each belongs — and why

Geometry lives in the Piece subclass. Only a Bishop knows a bishop moves diagonally; only a Knight knows the L-shape. Encapsulating "how I move" inside each piece is textbook encapsulation + polymorphism, and it's what lets us satisfy the Open/Closed Principle: adding a new piece is a new subclass, never an edit to a central validator. If geometry lived in the Board as a switch(pieceType), every new piece would force editing shared code — high coupling, low cohesion, exactly the anti-pattern. So the piece answers pseudoLegalTargets(board, from); note it reads the board (to stop at blockers) but it does not own board rules.

Path, occupancy, attack maps, and king-safety live in the Board. These questions are meaningless from inside a single piece because they require knowledge of every square and every other piece. A Pawn object cannot answer "is my king in check after this move?" — it has no idea where the king is or what attacks it. The Board owns the grid, so it owns isPathClear(from,to), isSquareAttackedBy(sq,color), and the make/undo used to test king safety. Placing king-safety here also means pins and check-escape need no special-case code: the Board applies the move, asks "is my king now attacked?", and undoes — one mechanism, at the only layer that can see enough to run it. This is the single most important placement decision, and getting it right (Board, not Piece) is what an interviewer looks for.

Turn order, status, history, and promotion choice live in the Game. Whose turn it is, whether the result is checkmate/stalemate, and recording the move for en-passant/castling/threefold-repetition are orchestration concerns above the board. The Game is the coordinator that sequences the layers: it calls piece-geometry, then Board path/king-safety, then updates status.

A subtle point — validation is a collaboration, not a location. The best mental model is a pipeline / Chain-of-Responsibility: Game.makeMove → ownership check (Game) → shape check (Piece) → path/occupancy (Board) → king-safety (Board) → commit + status (Game). Each stage lives with its data owner and can short-circuit. Some teams formalize this with a MoveValidator service or a list of Rule objects (Strategy per rule) that the Game runs in order — that's a valid variant and keeps rules testable in isolation. But the knowledge each rule needs still dictates which collaborator it delegates to. Even a dedicated MoveValidator shouldn't re-implement geometry; it should call piece.pseudoLegalTargets and board.isSquareAttackedBy, preserving single-source-of-truth.

Why this split (the principles)
Single Responsibility / high cohesion: each class validates only what it uniquely knows — Piece = shape, Board = board interactions, Game = flow. No god-method.
Low coupling + Open/Closed: geometry in subclasses means new pieces don't ripple into shared code; the Board depends on the abstract Piece, not concrete types.
DRY / single source of truth: isSquareAttackedBy reuses the same pseudoLegalTargets used for moving, so "how a piece attacks" is defined once.
Testability: you can unit-test a Bishop's geometry with no board, test path/king-safety with a small board fixture, and test status transitions at the Game level — clean seams.
Anti-patterns to avoid (and their cost)
Geometry in the Board via switch(type) — violates Open/Closed; every new piece edits shared code; low cohesion. Cost: fragile, merge-conflict-prone core.
King-safety inside Piece — impossible to do correctly (no board-wide view); leads to hand-rolled, buggy pin logic. Cost: illegal moves slip through.
One monolithic validate() — mixes six unrelated concerns; hard to test, hard to extend. Cost: change one rule, risk all.
Rules duplicated between move-gen and attack-detection — drift bugs where a piece "attacks" a square it can't actually move to. Cost: incorrect check detection.
Summary — what a strong answer emphasizes

Validation is layered by knowledge scope, and each layer lives with the object that owns the required data: shape → Piece (polymorphic, Open/Closed), path + occupancy + attack-map + king-safety → Board, turn/status/history → Game, orchestrated by the Game as a short-circuiting pipeline (optionally a MoveValidator/rule-chain). The load-bearing insight is that king-safety must be at the Board level via make/undo + attack-scan, which cleanly subsumes pins and check-escape without special-casing. The justification is SRP, high cohesion, low coupling, Open/Closed, and single-source-of-truth for how pieces move and attack. Follow-up probes usually push on: "why not a giant validator?" (cohesion/OCP), "where do castling/en passant go?" (Board + history, because they need multi-square + history context), and "how would you add a chess variant/new piece?" (new Piece subclass — proving the placement was right).

**Comparison:**

```
Validation concern	Owner	Why it lives there	Principle
Turn / piece ownership	Game	Game holds side-to-move & flow	SRP (orchestration)
Move shape / geometry	Piece subclass	Only the piece knows its rule; polymorphism, no switch(type)	Encapsulation + Open/Closed
Path clear / occupancy	Board	Needs every square between from→to	High cohesion (board data)
King safety (pins, check-escape)	Board (make/undo + attack-scan)	Needs king location + all opponent attacks	Single mechanism, no special cases
Castling / en passant preconditions	Board (+ hasMoved, lastMove)	Multi-square + history context	Correct data locality
Checkmate / stalemate	Game (over Board legal-move gen)	Terminal state after the move	Orchestration
How a piece attacks	reuse Piece.pseudoLegalTargets from Board	One source of truth for movement=attack	DRY
Anti-pattern	Why it's wrong	Cost
Geometry in Board.switch(type)	Violates Open/Closed; edits shared code per new piece	Fragile core
King-safety inside Piece	Piece lacks whole-board view	Illegal moves slip through
One monolithic validate()	Mixes 6 unrelated concerns	Untestable, risky change
Duplicated move vs attack rules	Rules drift	Wrong check detection
```

**Code:**

```java
// The Game orchestrates a short-circuiting validation pipeline;
// each stage delegates to the owner of the required knowledge.
class Game {
    Board board; Color sideToMove; Deque<Move> history = new ArrayDeque<>();

    boolean makeMove(Position from, Position to, PieceType promo) {
        Piece p = board.at(from);

        // Stage 1 — ownership/turn: GAME owns turn state
        if (p == null || p.color != sideToMove) return false;

        // Stage 2 — shape/geometry: PIECE owns its own rule (polymorphic, no switch)
        if (!p.pseudoLegalTargets(board, from).contains(to)) return false;
        //   (path-clear & friendly-occupancy already enforced inside pseudoLegalTargets,
        //    which reads Board but applies the PIECE's geometry)

        Move m = board.buildMove(from, to, promo);   // Board resolves capture/EP/castle/promo

        // Stage 3 — king safety/legality: BOARD owns whole-board view
        board.apply(m);
        boolean selfInCheck = board.isSquareAttackedBy(board.kingOf(p.color), p.color.opp());
        if (selfInCheck) { board.undo(m); return false; }   // rejects pins & non-escapes

        // Stage 4 — commit + status: GAME owns flow/history/terminal state
        history.push(m);
        board.lastMove = m;                          // enables en passant next turn
        sideToMove = sideToMove.opp();
        this.status = board.computeStatus(sideToMove); // CHECK / CHECKMATE / STALEMATE / ACTIVE
        return true;
    }
}

// PIECE: geometry only — the knowledge it uniquely owns
abstract class Piece {
    final Color color;
    abstract List<Position> pseudoLegalTargets(Board b, Position from); // shape + blockers
}

// BOARD: everything that needs the whole board — one source of truth reused by move-gen & check
class Board {
    boolean isSquareAttackedBy(Position target, Color by){
        for (var sqr : allSquaresWithColor(by))
            if (at(sqr).pseudoLegalTargets(this, sqr).contains(target)) return true; // reuse geometry
        return false;
    }
    boolean canCastle(Color c, boolean kingside){       // Board: needs history + multiple squares
        // king & rook !hasMoved; between squares empty;
        // king start/path/dest NOT isSquareAttackedBy(opp)  -> can't castle out/through/into check
        return true;
    }
}
```

**Example input / output:**

```text
Placement test — "move e2 knight, own king on e1, enemy rook on e8" (a PIN):

  Stage 1 (Game): e2 is White, White to move            -> pass
  Stage 2 (Piece): Knight geometry allows e2->g3         -> pass   (Piece can't know about the pin)
  Stage 3 (Board): apply -> kingOf(WHITE)=e1;
                   isSquareAttackedBy(e1, BLACK)=true (rook e8) -> undo -> REJECT
  => Correct rejection happened at the BOARD layer, exactly where whole-board knowledge exists.
     Had we tried to validate this inside Knight, it could not have detected the pin.

Placement test — "add a new variant piece (e.g., Amazon = Queen+Knight)":
  Only action needed: new subclass `Amazon extends Piece` implementing pseudoLegalTargets.
  Board & Game code UNCHANGED  => proves geometry belongs in Piece (Open/Closed satisfied).
```

**Flow:**

1. Board -- whole-board knowledge
2. Game -- turn/status/history
3. yes
4. no
5. Piece subclass -- geometry
6. ownership & turn check
7. shape / pseudo-legal targets
8. commit + compute status
9. path clear + occupancy
10. apply move
11. own king attacked?
12. undo -> reject pin/check

