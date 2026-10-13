---
id: "06_Cloud_Computing/Cloud_Cost/FinOps/Carbon_and_Cost_Trade-offs"
level: 5
type: "deep-dive"
topic: "06_Cloud_Computing/Cloud_Cost/FinOps"
tools: []
---

# Carbon and Cost Trade-offs

**L5 Expert** · [↑ FinOps](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How do you tag resources created by Terraform so cost allocation actually works? Walk me through default_tags versus per-resource tags and what happens to resources Terraform doesn't manage.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26056_How_do_you_tag_resources_created_by_Terraform_so_cost_alloca.md>)
- [Intermediate] [How do you decide the right autoscaling min/max and cooldown so you don't flap and pay for churn? Walk me through tuning it for a spiky workload.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26143_How_do_you_decide_the_right_autoscaling_min_max_and_cooldown.md>)
- [Deep] [Describe how you'd measure and govern the carbon and cost jointly (sustainability-aware FinOps) — choosing regions/times for cheaper AND greener compute. Where do cost and carbon optimization align, and where do they conflict?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-25945_Describe_how_you_d_measure_and_govern_the_carbon_and_cost_jo.md>)
- [Deep] [A team's cost dashboard is green but their cloud carbon footprint just doubled after a region migration. How do you reconcile cost-optimal and carbon-optimal placement when the cheapest region is the dirtiest?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-25950_A_team_s_cost_dashboard_is_green_but_their_cloud_carbon_foot.md>)
- [Deep] [A principal probe on sustainability: leadership wants carbon-aware scheduling AND cost optimization, and you know the cheapest region is often the dirtiest. Design a joint cost-and-carbon placement engine and be explicit about where the two objectives conflict and how you'd let the business set the exchange rate between them.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26207_A_principal_probe_on_sustainability_leadership_wants_carbon-.md>)
- [Deep] [A bar-raiser asks: your org's FinOps dashboards show green while a region migration just doubled your cloud carbon footprint because the cheap region is coal-heavy. How do you architect governance that surfaces the carbon-cost trade before a placement decision is made, not after, and who owns the exchange rate?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26237_A_bar-raiser_asks_your_org_s_FinOps_dashboards_show_green_wh.md>)
- [Deep] [Explain how you'd govern the joint cost-and-carbon of ML training, choosing regions and times for cheaper and greener GPU compute, given that the cheapest GPU capacity is often in carbon-heavy regions. Where do cost and carbon optimization align for ML, and where do they conflict?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26312_Explain_how_you_d_govern_the_joint_cost-and-carbon_of_ML_tra.md>)
- [Deep] [Explain how you'd govern the joint cost-and-carbon of a data platform, choosing regions and schedule windows for cheaper and greener batch compute, given the cheapest region for warehouse capacity is often carbon-heavy. Where do cost and carbon optimization align for data workloads, and where do they conflict?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26615_Explain_how_you_d_govern_the_joint_cost-and-carbon_of_a_data.md>)

## Related concepts

- [Hub-and-spoke Cross-cloud Networking](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Hub-and-spoke_Cross-cloud_Networking.md>) *(Multi Cloud Data)*
- [Multi-cloud Strategy](<../../../Cloud_Foundations/Cloud_Fundamentals/Managed_Services_vs_Self-hosting/Multi-cloud_Strategy.md>) *(Cloud Fundamentals)*
- [Database Cost Optimization](<../../../../01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Database_Cost_Optimization.md>) *(Cost Optimization)*

