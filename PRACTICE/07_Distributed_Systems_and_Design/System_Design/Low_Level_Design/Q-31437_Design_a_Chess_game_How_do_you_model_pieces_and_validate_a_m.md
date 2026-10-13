---
type: "practice"
difficulty: "Deep"
---

# Design a Chess game. How do you model pieces and validate a move?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Extensible Rule Engines](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Extensible_Rule_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

"Design a Chess game — how do you model pieces and validate a move?" This is a classic LLD/OOP interview. The interviewer is probing: can I turn messy real-world rules into a clean object model (classes, responsibilities, relationships), can I place move-validation logic where it belongs, and do I handle the rules that separate a naive solution from a correct one — check, pinned pieces, castling, en passant, promotion? A junior answer hard-codes a giant if/switch on piece type. A strong answer uses polymorphism (each piece computes its own moves), keeps the Board responsible for board-level rules (path, occupancy, and post-move king-safety), and layers validation cleanly.

Clarifying questions I'd raise: two-player local or networked? Do we need move history (for undo, threefold repetition, en passant, castling rights)? Full FIDE rules including stalemate/50-move, or core movement? I'll assume full core rules with history, single-process, extensible for an engine later.

Plan: (1) enumerate entities and responsibilities; (2) model pieces with a Piece base + polymorphic getPseudoLegalMoves (Strategy/Template Method); (3) define the two-phase validation — pseudo-legal (does the piece move like that + path clear + not landing on own piece) then legal (does it leave/keep my king in check); (4) cover special moves; (5) note the class diagram and complexity.

Entities and responsibilities
Game — orchestrates: holds the Board, the two Players, whoseTurn, status (ACTIVE/CHECK/CHECKMATE/STALEMATE/DRAW), and the moveHistory. Exposes makeMove(from, to).
Board — the 8×8 grid of Squares (or Piece[8][8]). Owns board-level queries: pieceAt(pos), isPathClear(from,to), isSquareAttackedBy(pos, color), and applying/undoing a move. This is the right home for rules that need global board knowledge.
Square / Position — a coordinate (row 0–7, col 0–7) and optionally an occupant.
Piece (abstract) — color, hasMoved (needed for castling & pawn double-step), and the polymorphic getPseudoLegalMoves(board) or canMoveShape(from,to). Subclasses: King, Queen, Rook, Bishop, Knight, Pawn.
Move — value object: from, to, movedPiece, capturedPiece, promotion, isCastle, isEnPassant. Storing captured piece and flags makes undo trivial (essential for check-testing and engine search).
Player — color, and (optionally) a strategy for choosing moves (human vs AI). Introducing a PlayerStrategy interface is the clean way to later plug an engine — the Strategy pattern.
Modeling pieces — polymorphism over if piece==KNIGHT

The key OOP decision: each piece knows how it moves. Piece declares getPseudoLegalMoves(board, from); each subclass implements its geometry. Sliders (Rook/Bishop/Queen) share "ray" logic, so I factor a SlidingPiece with directions (this is Template Method — the base walks the rays, subclasses supply the direction vectors). Knight and King are "offset" pieces (fixed jump set). Pawn is the special one (forward-only move, diagonal-only capture, double-step from start, en passant, promotion), so it overrides fully.

This gives Open/Closed compliance: adding a fairy-chess piece is a new subclass, not an edit to a central switch. Directions as data:

Rook: {(±1,0),(0,±1)}; Bishop: {(±1,±1)}; Queen: union of both; King/Knight: fixed offset lists.
Validating a move — the two-phase model (this is the crux)

Move validation is layered, and getting the layering right is what interviewers score:

Phase 1 — pseudo-legal:

It's this player's turn and from holds a piece of that color.
to is on the board and not occupied by a friendly piece.
The move matches the piece's movement shape (getPseudoLegalMoves contains to).
For sliding pieces, the path is clear (board.isPathClear) — knights skip this (they jump).
Pawn-specific: forward moves must be to empty squares; captures must be diagonal onto an enemy (or a valid en passant target from the last move).

Phase 2 — legality (king safety): A pseudo-legal move is illegal if it leaves your own king in check. The robust, bug-free way to check this: make the move on the board, then ask isSquareAttackedBy(myKingSquare, opponentColor), then undo. This single mechanism handles pins (a pinned piece "can move like that" but exposes the king, so it's rejected) and getting out of check (only moves that clear the attack survive) — no special-casing pins is needed. This is why Move stores the captured piece: undo restores state exactly.

isSquareAttackedBy(sq, color) is implemented by asking every piece of color for its pseudo-legal captures and testing whether any hits sq — reusing the same polymorphic move generation, so there's one source of truth for "how pieces attack."

Special moves:

Castling: king & rook hasMoved==false; squares between are empty; and the king's start, path, and destination squares are not attacked (can't castle out of, through, or into check). Uses isSquareAttackedBy.
En passant: legal only immediately after the opponent's adjacent two-square pawn advance — hence the need for moveHistory / a lastMove.
Promotion: pawn reaching the last rank replaces itself with a chosen Queen/Rook/Bishop/Knight.

Game status after a move: switch turn; if the mover's opponent is in check and has no legal move, it's checkmate; not in check and no legal move → stalemate (draw). "Has any legal move" reuses Phase 1 + Phase 2 across all that player's pieces.

Where does validation live? (short answer; a sibling question digs deeper)

Split by knowledge scope: shape/geometry → the Piece subclass (only the piece knows its geometry); path, occupancy, attack maps, king-safety, castling/en-passant context → the Board (they need whole-board state); turn order, status, history, promotion choice → the Game. A common mistake is putting king-safety inside Pawn — it can't, it lacks board-wide knowledge. Keep pieces "dumb about the world," and let the Board answer world-level questions.

Complexity and summary

Generating one piece's moves is O(1) for offset pieces, O(board dimension) per ray for sliders. Full legal-move generation for a side is roughly O(pieces × moves × attack-scan) — trivial for a UI; an engine would later add bitboards for speed. A strong answer emphasizes: polymorphic piece movement (no type switch), the two-phase pseudo-legal → make/undo/king-safety validation (which elegantly subsumes pins and check), storing enough in Move to undo, and correctly locating each rule in Piece vs Board vs Game.

#### Common mistakes
/ follow-ups: hard-coding rules in a switch; forgetting you can't castle through check; forgetting en passant / promotion need history; validating "is my king safe" with hand-written pin logic instead of make-move-and-test. Follow-ups: how to add an AI (PlayerStrategy + minimax), how to serialize state (FEN), and draw rules (threefold repetition via a Zobrist-hashed history, 50-move counter).

**Comparison:**

```
Rule / query	Lives in	Reason
How this piece moves (shape)	Piece subclass	Only the piece knows its geometry; polymorphism replaces a type switch
Ray-walking for sliders	SlidingPiece (Template Method)	Shared "walk until blocked" logic, direction vectors supplied by subclass
Path clear / occupancy / attack map / king-safety	Board	Needs whole-board state
Castling / en passant preconditions	Board (+ hasMoved, lastMove)	Depend on history and multiple squares
Turn order, status, history, promotion choice	Game	Orchestration concerns
Human vs AI move choice	PlayerStrategy (Strategy)	Pluggable engine without touching rules
Piece	Move generation type	Complexity/move-gen
Rook/Bishop/Queen	Sliding rays	O(dim) per direction
Knight/King	Fixed offsets	O(1)
Pawn	Special (fwd/capture/2-step/EP/promo)	O(1), needs history
```

**Code:**

```java
enum Color { WHITE, BLACK; Color opp(){ return this==WHITE?BLACK:WHITE; } }
record Position(int r, int c) { boolean onBoard(){ return r>=0&&r<8&&c>=0&&c<8; } }

abstract class Piece {
    final Color color; boolean hasMoved = false;
    Piece(Color color){ this.color = color; }
    // polymorphic: each piece knows ONLY its own geometry
    abstract List<Position> pseudoLegalTargets(Board b, Position from);
}

abstract class SlidingPiece extends Piece {      // Template Method: base walks rays
    SlidingPiece(Color c){ super(c); }
    abstract int[][] directions();
    List<Position> pseudoLegalTargets(Board b, Position from){
        var out = new ArrayList<Position>();
        for (int[] d : directions()){
            int r=from.r()+d[0], c=from.c()+d[1];
            while (new Position(r,c).onBoard()){
                Piece occ = b.at(new Position(r,c));
                if (occ==null){ out.add(new Position(r,c)); }
                else { if (occ.color!=color) out.add(new Position(r,c)); break; } // capture, then stop
                r+=d[0]; c+=d[1];
            }
        }
        return out;
    }
}
class Rook   extends SlidingPiece { Rook(Color c){super(c);}   int[][] directions(){return new int[][]{{1,0},{-1,0},{0,1},{0,-1}};} }
class Bishop extends SlidingPiece { Bishop(Color c){super(c);} int[][] directions(){return new int[][]{{1,1},{1,-1},{-1,1},{-1,-1}};} }
class Queen  extends SlidingPiece { Queen(Color c){super(c);}  int[][] directions(){return new int[][]{{1,0},{-1,0},{0,1},{0,-1},{1,1},{1,-1},{-1,1},{-1,-1}};} }

class Knight extends Piece {
    Knight(Color c){ super(c); }
    static final int[][] J={{1,2},{2,1},{-1,2},{-2,1},{1,-2},{2,-1},{-1,-2},{-2,-1}};
    List<Position> pseudoLegalTargets(Board b, Position from){
        var out=new ArrayList<Position>();
        for(int[] j:J){ var p=new Position(from.r()+j[0],from.c()+j[1]);
            if(p.onBoard()){ var occ=b.at(p); if(occ==null||occ.color!=color) out.add(p); } }
        return out;
    }
}
// King (offsets, + castling handled by Board), Pawn (fwd move / diagonal capture /
// double-step / en passant / promotion) implemented similarly, Pawn overriding fully.

class Move { Position from,to; Piece moved, captured; boolean isEnPassant,isCastle; Piece promo; }

class Board {
    Piece[][] sq = new Piece[8][8];
    Move lastMove;                                  // needed for en passant
    Piece at(Position p){ return sq[p.r()][p.c()]; }

    boolean isSquareAttackedBy(Position target, Color by){
        for(int r=0;r<8;r++) for(int c=0;c<8;c++){
            Piece p = sq[r][c];
            if(p!=null && p.color==by)
                if(p.pseudoLegalTargets(this, new Position(r,c)).contains(target)) return true;
        }
        return false;
    }
    Position kingOf(Color col){ /* scan for the King */ return null; }

    // THE VALIDATION ENTRY POINT
    boolean isLegal(Move m){
        Piece p = at(m.from);
        if(p==null || p.color!=/*sideToMove*/p.color) return false;
        if(!p.pseudoLegalTargets(this, m.from).contains(m.to)) return false; // phase 1: shape+path+occupancy
        // phase 2: make, test own-king safety, undo
        apply(m);
        boolean selfInCheck = isSquareAttackedBy(kingOf(p.color), p.color.opp());
        undo(m);
        return !selfInCheck;   // rejects pinned-piece moves & non-escapes from check automatically
    }
    void apply(Move m){ m.captured=at(m.to); sq[m.to.r()][m.to.c()]=m.moved; sq[m.from.r()][m.from.c()]=null; }
    void undo (Move m){ sq[m.from.r()][m.from.c()]=m.moved; sq[m.to.r()][m.to.c()]=m.captured; }
}
```

**Example input / output:**

```text
Setup: White Rook a1, White King e1, Black Bishop pinning along a1-e1?
Example 1 — legal slide:
  Board: White Bishop c1 (empty diagonal to h6)
  move c1->h6  -> phase1: shape OK (bishop diagonal), path clear -> phase2: own king not attacked -> LEGAL

Example 2 — PINNED piece (no special-case code needed):
  White King e1, White Knight e2, Black Rook e8 (e-file open behind the knight)
  move e2->g3  -> phase1: knight shape OK
                -> phase2: apply -> Black Rook e8 now attacks e1 (king) -> selfInCheck=true -> ILLEGAL
  (the make/undo + isSquareAttackedBy caught the pin with zero pin-specific logic)

Example 3 — castling guard:
  White e1 O-O, but f1 is attacked by a black bishop
  -> reject: cannot castle THROUGH an attacked square (isSquareAttackedBy f1)

Example 4 — checkmate detection:
  After White's move, Black king in check AND no black move survives phase2 -> status=CHECKMATE
```

**Data:**

```

Class relationships (composition): Game 1--1 Board Board 1--* Square/Piece[8][8] Game 1--2 Player Piece <|-- King,Queen,Rook,Bishop,Knight,Pawn Game 1--* Move (history) SlidingPiece <|-- Rook,Bishop,Queen (Template Method) Player 1--1 PlayerStrategy (Human | AIEngine) -- Strategy pattern for future engine

CODE
Copy
```text
Standard start position in FEN (compact serializable state):
rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1
   (piece placement / side-to-move / castling-rights / en-passant-square / halfmove / fullmove)
```

**Flow:**

1. no
2. yes
3. no
4. yes
5. yes
6. no
7. no + in check
8. no + not in check
9. yes
10. Game.makeMove from,to
11. from holds side-to-movepiece?
12. reject
13. piece.pseudoLegalTargetscontains to?shape+path+occupancy
14. Board.apply move
15. own king attacked byopponent?
16. undo -> reject: pin / still incheck
17. commit move, push history,switch turn
18. opponent has any legal move?
19. CHECKMATE
20. STALEMATE draw
21. status ACTIVE / CHECK

