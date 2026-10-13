---
id: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Threaded_Comments_Design"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design"
tools: []
---

# Threaded Comments Design

**L4 Advanced** · [↑ Low Level Design](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Basic] [Design a tagging system where any entity can have many tags. How do you model this cleanly?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31570_Design_a_tagging_system_where_any_entity_can_have_many_tags_.md>)
- [Intermediate] [Design a Trie-backed contact/phone-book search. What classes and how does prefix search work?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31508_Design_a_Trie-backed_contact_phone-book_search_What_classes_.md>)
- [Intermediate] [In a Q&A system how would you model voting and compute a user's reputation?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31523_In_a_Q_A_system_how_would_you_model_voting_and_compute_a_use.md>)
- [Intermediate] [Design a leaderboard that updates live as scores change. Which structure gives fast top-K and rank?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31559_Design_a_leaderboard_that_updates_live_as_scores_change_Whic.md>)
- [Deep] [An interviewer asks you to design a 'you may also like' recommendation module at class level with a fallback when the ML service is down; where do you draw the interface so the fallback is a swappable strategy?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31405_An_interviewer_asks_you_to_design_a_you_may_also_like_recomm.md>)
- [Deep] [How would you design the class model for a comment system that must support threaded replies AND efficient 'load more' pagination at any depth without N+1 queries; where does the tree vs flat trade-off bite?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31657_How_would_you_design_the_class_model_for_a_comment_system_th.md>)
- [Deep] [How would you design a poll/survey model so adding a new question type (ranking, matrix) later doesn't require touching the scoring, storage, and rendering code all at once—where's the polymorphism seam?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31665_How_would_you_design_a_poll_survey_model_so_adding_a_new_que.md>)
- [Deep] [Design the class model for a leaderboard that must support seasonal resets, ties broken by earliest-achieved, and 'friends-only' views; how do you structure storage so all three queries stay fast?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31752_Design_the_class_model_for_a_leaderboard_that_must_support_s.md>)

## Related concepts

- [Search Backend Design](<../High_Level_Design/Search_Backend_Design.md>) *(High Level Design)*
- [Inverted Indexes](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Inverted_Indexes.md>) *(Data Structures and Algorithms)*
- [Pagination Strategies](<../System_Design_Core/Pagination_Strategies.md>) *(System Design Core)*

## References

- *Fuzzy Data Matching with SQL* — Jim Lehmer — Tying It All Together
- *System Design Interview Vol. 2* — Alex Xu, Sahn Lam — DESIGN A SEARCH AUTOCOMPLETE SYSTEM

