# A/B Test Report: 
Globox A/B test for the food category 

## Purpose

The Globox company as an e-commerce has witnessed growth in its food and drink category and the company is planning to create further awareness to this category and in turn increase revenue. The purpose is to determine if a new display banner on the homepage about the new food and drink category offerings will generate more convertions, boost customer spend and as a result improve sales.

## Hypotheses

The null and alternative hypotheses for the A/B test are as follows:

Null Hypothesis, Ho : There is no significant difference in the key performance indicators (KPI)
between the groups. Any observed difference is only due to randomness

Alternative Hypothesis, H1 : There is a meaningful increase in performance due to the
treatment as observed with the treatment group KPI.

Mathematically,
                Ho : d = Pb - Pa = 0
                H1 : d = Pb - Pa > 0
where Pa and Pb are the key performance indicators for the control and treatment groups
respectively.

## Methodology

### Test Design

    The test aimed to assess the impact of a new food banner on Globox e-commerce visitors. Visitors were randomly assigned to either a control group or a treatment group. The control group viewed the homepage without the new food banner, while the treatment group saw the homepage with the new banner. Interactions with food and drink were then analyzed in terms of conversion rate and average spending per group. The primary objective was to determine if the new banner led to improvements in these metrics.

- **Population:**  A total number of 48943 users were examined in the test groups out of which 24343 was randomly allocated into the control group while 24600 into the treatment group. 

- **Duration:** The start and end dates of the experiments are 2023-01-25 and 2023-02-06 respectively, making it a duration of 12 days in total.

- **Success Metrics:** The success of the display banner was evaluated by comparing conversion rates between the test and control groups, with a significance level of 5% and a confidence interval of 95%. Additionally, the average total spend per group was analyzed, also at a significance level of 5% and a confidence interval of 95%.

## Results
### Data Analysis
- **Pre-Processing Steps:** The analysis involved exploring data retrieved from a relational database comprising three tables: the activity table, groups table, and users' table. These tables contained information about users' group demographics, purchasing behavior during the test period, and the random assignments to different groups.

Queries were executed to extract key metrics such as conversion rates, conversion counts, total spending, and average spending. To enhance the understanding of relationships among elements in the three tables, Tableau was utilized for visualizing how test metrics, including conversion rates and averages, correlated with users' devices, gender, and country. This comprehensive approach was undertaken as part of the A/B testing process.

Below are the sql queries

```sql

--Calculating the start and end dates of the experiment
Select 
    min(join_dt) as start_date, 
    max(join_dt) as end_date, 
    max(join_dt)-min(join_dt) as duration
from groups
;

Number of users were in the control and treatment groups?
select 
     count(case
            when grp.group = 'A' then 'control'
          end) as control_users,     
    count(case
          	when grp.group = 'B' then 'treatment'
          end) as treatment_users
from groups as grp

where grp.group IN ('A', 'B')
;
--control_users: 24343. treatment_users: 24600

/**Calculating the conversion rate of all users 
(Total numnber of conversion (spent)/ total users) * 100
**/
Select
   (count(distinct act.uid)/count(distinct users.id)::numeric) *100
from users 
left join activity as act
on users.id = act.uid
;

--Calculating the user conversion rate for the control and treatment groups

with count_conv as (select grp.group, grp.uid,

    case
       when sum(act.spent) > 0 then 1   
       else 0
    end as conversions    
	
from groups as grp 
left join activity as act
on grp.uid = act.uid

group by grp.uid
)
select cc.group, avg(conversions)*100 
from count_conv as cc
group by cc.group
;

/**Calculating the average amount spent per user 
for the control and treatment groups, 
including users who did not convert**/

select grp.group,
       sum(act.spent)/count(distinct grp.uid) as avg_spent
from groups as grp
left join activity as act on grp.uid = act.uid

group by grp.group
;

--Calculating the total number of conversions per group --
select grp.group, count(*)
from groups as grp
left join activity as act on grp.uid = act.uid
where grp.group IN('A', 'B')

group by grp.group
			
having sum(act.spent) is not null
;

/**  
Query that returns the user ID, the user’s country, the user’s gender, the user’s device type, the user’s test group, 
whether or not they converted (spent > $0), and how much they spent in total ($0+).
This is the sample data used for the statistical test**/

SELECT u.id, 
	u.country, 
  u.gender,
  grp.device,
  grp.group,
  COALESCE(SUM(act.spent),0) AS total_spent,
  CASE WHEN grp.group = 'A' AND SUM(act.spent) > 0 THEN 1 ELSE 0 END AS A_converted,
  CASE WHEN grp.group = 'B' AND SUM(act.spent) > 0 THEN 1 ELSE 0 END AS B_converted
FROM users AS u
LEFT JOIN groups AS grp ON u.id = grp.uid
LEFT JOIN activity AS act ON u.id = act.uid
GROUP BY u.id, u.country, u.gender, grp.device, grp.group
;

```

- **Statistical Tests Used:** 
The first hypothesis test was to compare the conversion rates between the 2 groups. Because this is testing the difference in proportion between the groups, I made use of Two-sample z-test.

The second hypothesis test was to compare the average spent between the 2 groups. I made use of the two-sample t-test.

- **Results Overview:** After concluding the z-test on the first hypothesis on conversion rate between the two groups, the probability value came out to be 0.0001  which is below the stated 0.05 significant level. The t-test conducted on the average spent at the other side shows the opposite as the probability value stood at 0.94 which is way above the 0.05 significant level.

### Findings7

## Interpretation
- **Outcome of the Test(s):** Based on the result of the first z-test, we reject the null hypothesis that there is no difference in the user conversion rate between the two groups. However, the t-test result indicated that we should fail to reject the null hypothesis that there is no difference in the average spent between the groups.

- **Confidence Level:** Based on stattistics, there is a 95% confidence level that the conversion rate estimate between the 2 groups will fall between 0.0035 and 0.0107 confidence interval. State the statistical confidence

Also, there is a 95% confidence level that the average spent estimate between the 2 groups will fall between -0.438 and 0.471 confidence interval. State the statistical confidence

Below is the chat of the confidence interval:
![Alt text](image.png)

## Conclusions
- **Key Takeaways:** 

The introduction of the new display banner on the homepage has shown a statistically significant increase in the conversion rate. This suggests that the banner effectively captures users' attention and drives them towards making purchases within the food and drink category.

Surprisingly, there was no statistically significant difference in the average spending between the control and treatment groups. This implies that while the banner may attract more users to engage with the food and drink category, it doesn't necessarily influence the amount they spend per transaction.

The analysis included a novelty effect test to ensure the data's integrity and reliability. The absence of any significant novelty effect indicates that the observed changes are likely genuine and not influenced by temporary factors.
![alt text](<Novelty average-1.png>)
![alt text](<Novelty conversion.png>)

- **Limitations/Considerations:** 

While the sample size used in the A/B test was sufficient for detecting statistically significant differences, increasing the sample size further could provide more precise estimates and reduce the margin of error. Consider conducting future tests with larger sample sizes to enhance the reliability of the results.

## Recommendations
- **Next Steps:** 
The statistically significant increase in conversion rates provides a strong rationale for launching the new display banner. It's likely to enhance user engagement within the food and drink category, potentially leading to increased sales over time.

Launching the banner is worthwhile even if only one success measure shows a significant increase. Moreover,the banner is not difficult to launch and manage. Therefore, I recommend that we launch the experiment.
- **Further Analysis:** 
Supplement quantitative data with qualitative insights gathered from user surveys, interviews, or usability tests. Qualitative analysis can provide valuable context and deeper understanding of user motivations, preferences, and pain points, helping to refine the design and messaging of the display banner for maximum impact.

## Link to other contributing analysis
spreadsheets statistical analysis : https://docs.google.com/spreadsheets/d/1KTmuilnAC91nNfgj6wyNM3qQotfwxMEimbTsN-AOzQQ/edit?usp=sharing

python analysis link : https://colab.research.google.com/drive/15WPtkfrQkJ85faJ1Oce_vjjK32gnKnoF?usp=sharing

tableau dashboard link: https://public.tableau.com/views/GloBoxDashboard_17052848551170/Dashboard1?:language=en-US&publish=yes&:display_count=n&:origin=viz_share_link




