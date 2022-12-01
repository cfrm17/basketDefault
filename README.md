# Basket Default Swap and CDO Valuation


A pricing model is presented to calculate Mark-to-Market (MTM) and all sensitivities for basket default swaps and Collateral Debt Obligations (CDOs) (FirstNofM, GiantFirstLoss, GiantFirstLossPayEnd, Caribou, and Reindeer). It is composed of the credit library, BulkCurveGenerator, five outstanding pricing templates, and Scenario Manager. 

Within the modeling framework, the sensitivities measure the change of present value (PV) when certain risk factor is perturbed. For example, recovery rate sensitivity is calculated by perturbing the recovery rate for each obligor in the reference collateral pool of the trade, which can be defined as follow.  Assume the collateral pool be a set of obligors,  , in which each obligor is associated with market implied recovery rate  . Base on this collateral pool, a CDO trade has a tranche structure with m tranches,  .  The recovery rate sensitivity for the jth   tranche with respect to the ith obligor is defined as

(1)	 

where   is the PV of the tranche without recovery rate perturbation.

Currently SH3 supports the following the sensitivity tests:  

1)	Credit spread sensitivity;
2)	Default sensitivity;
3)	Recovery rate sensitivity;
4)	Correlation sensitivity;
5)	1-day theta and 6-month theta.

In spreadsheet model the credit spread sensitivity is defined as the change of PV when the credit spread curve is parallel shocked several basis points for each obligor in the collateral pool. The default sensitivity is calculated by moving the default time of perturbed obligor to the valuation date to reflect the default risk of the obligors in the collateral pool. The correlation sensitivity is usually measured by changing the flat correlation of the collateral pool ten percent. 1-day theta and 6-month theta is defined in model as moving valuation date one day and 6 months forward, respectively, with all other parameters unchanged.

The default sensitivity and the credit spread sensitivity have been vetted before. Except for correlation sensitivity, in SH3 there exist two versions of risk measures for all sensitivity tests. One version is the regular way by perturbing the risk factor and recalculating PV. The sensitivity is then calculated by Eq.(1) directly. The other way, which is more efficient, employs a re-weight approximation. It is denoted as weighted Monte Carlo (WMC) sensitivities.

The changes and new developments are outlined below

1.	Credit Library and BulkCurveGenerator

The Credit Library and BulkCurveGenerator serve the purpose of loading all the input credit spread curves and discount curves, calibrating hazard rate curves, and generating correlated default events using weighted/non-weighted Monte Carlo (MC) simulation. 

There is one major change to the credit library. An entropy threshold, which equals 10, is set in the library.  It has been known that the WMC simulation sometimes would converge to an apparently “wrong” value.  An entropy threshold is thus set to identify a possible “wrong” solution. 
	
All the outstanding basket default swaps and synthetic CDOs are built in five templates, namely, FirstNofM, GiantFirstLoss, GiantFirstLossPayEnd, Caribou, and Reindeer, in which the payoff functions and the WMC sensitivities are implemented. WMC sensitivities include credit spread sensitivity (WMCPCS), default sensitivity (WMCPDS), recovery rate sensitivity (WMCRRS), interest rate sensitivity (WMCIR), and theta (WMCCR). 

The changes to the templates are:

a)	Recently has changed the treatment of settlement risk. The settlement risk is defined as a scenario that one obligor in the collateral pool defaults after the trade date but prior to the settlement date. The new treatment simply ignores the Loss Given Default (LGD) of the obligor which defaults before the settlement date. The Caribou and Reindeer have been built upon the new methodology. It is necessary to bring all the templates consistent.

b)	WMCIR, WMCRR, and WMCCR have been implemented in the templates. However, these measures are not included. Further tests on both methodology and implementation are needed. 

2.	Scenario Manger

Apart from managing the computation of MTM and sensitivities using those pricing templates, Scenario Manager (SM) calculates the sensitivities through regular method. That is, by loading the pricing templates, the unperturbed and perturbed PV for a trade are calculated and then the sensitivity is defined as the difference of the two values. 

The risk measures built in SM are recovery rate sensitivity, interest rate sensitivity, correlation sensitivity, and 1-day and 6-month theta without re-weigh approximation.

•	Re-weight Approximation

In the computation of PV involves the generation of correlated default events for the collateral pool using MC simulation. If the collateral pool is large and the risk factor is the parameter of the individual obligor such as the credit spread, recovery rate, etc, it will take a huge amount of simulation time.

Instead of recalculating the PV using MC simulation, employs a re-weight approximation. The base case is first calculated by using WMC simulation then each path is recorded. When a risk factor is perturbed with a very small amount, we fix the MC path and assume that only the path weight of each MC scenario is changed. The change of PV is then calculated by recalculating the weight of each MC path. This re-weight approximation in essence ignores the effect on perturbed risk factor on joint default event. 

The approximation enables us to do the WMC simulation once in the computation of sensitivity test hence saves simulation time. Another advantage of re-weight approximation is that it greatly increases the convergence of MC, which has a major influence on sensitivities . 

•	Benchmarking/Comparison to Other Models

There are many sensitivity measures and corresponding models, depending on what kind of risk one wants to catch and how the risk factor is defined. In model, two sets of risk measures, with and without re-weight approximation, could serves as a comparison model. The comparison test has been carried out and reported in the next section.

The consistence among five pricing templates is tested (see https://finpricing.com/lib/FiZeroBond.html).  Because the notional of the first tranche is set to be the LGD of an obligor, the first tranche can be viewed as a FTD trade. By making appropriate changes to the templates we build the test trade in five templates. In Caribou and Reindeer we set the all the thresholds to zero. In GiantFirstLossPayatEnd we turn off the option of moving default cash flow to the maturity. By doing this, the Test-I built in five templates becomes equivalent. Then by using the same random seed and number of scenario, we compare the generated MTM and all the sensitivities for Test-I using five templates.

Test results have shown that all pricing templates generate EXACTLY the same results for MTM, credit spread sensitivity, default sensitivity, correlation sensitivity, theta, interest rate sensitivity. For recovery rate sensitivity, test results have shown that all pricing templates except FirstNofM generate EXACTLY the same results.  Because the FTD trade has a different definition on notional amount, the notional of the first tranche in FTD trade will change while that in the other four templates remains unchanged when the recovery rate is perturbed. Hence recovery rate sensitivity for a FTD trade is different from that generated by the other four templates.  Test results, denoted as model, are shown in the tables of the next section. 

Through this process of testing we are confident that, therefore, all the outstanding pricing templates in spreadsheet model are consistent with each other.

We also conclude that we do not need to test the five pricing templates individually by independently building a test model for each template.  A test model in which a GiantFirstLoss and a FirstNofM are built is good enough.

Taken Test-I as the test trade, which is built as of GiantFirstLoss and a FirtNofM, we check the implementation of sensitivity tests without re-weigh approximation by building an independent test model. 

For each sensitivity test eight sets of results were simulated using different random seeds (4359 ~ 4366). The mean and an error bound are then calculated and compared. The full results for different random seeds can be found in Appendix V. Because we are comparing results generated by MC simulation, we define an error bound, which is computed as

(2)	 

with

(3)	 ,

where max and min are the maximum and minimum prices in eight simulations with different random seeds respectively.

The interest rate sensitivity, correlation sensitivity, and 1-day theta, calculated by the test model and model, respectively, are shown and compared in Tables 1~3.  In both models, the interest rate sensitivity is calculated by parallel shocking interest rate 10 basis points up; the correlation sensitivity is computed by increasing the correlation 0.1 up; and the 1-day theta is done by moving valuation date one day forward.

The GiantFirstLoss recovery rate sensitivities, calculated by the test model and model, respectively, are shown and compared in Table 4(a). This is the one built in the GiantFirstLoss trade in which the notional of the tranche won’t change when the recovery rate is shocked. The FTD recovery rate sensitivities, calculated by the test model and the model, respectively, are given in Table 4(b).  Because the perturbed obligor has a smaller LGD, the FTD recovery rate sensitivity tends to have a smaller value compared to GiantFirstLoss one, although their MTM are exactly the same. This expectation is confirmed in Tables 4(a) and 4(b). 

The credit spread sensitivity is employed as the test risk measure to investigate re-weight approximation. The credit spread sensitivities of 0~3% tranche and 15~30% tranche change versus shocking amount, when the re-weight approximation is switched on and off, are shown in Figures 1(a) and 1(b).  The detailed results for all tranches are given in Appendix VI. It can be seen that in general the discrepancy is small in both figures, although it tends to be larger as the shock becomes larger. This suggests that the re-weight approximation is good for the Test-II.

Because the re-weight approximation ignores the possible change of joint default events when the risk factor is perturbed, we change the correlation intensity to test the extent of this approximation. We switch the correlation from 0.3 to 0.7.  The same sets of results for 0~3% tranche and 15~30% tranche are shown in Figures 2(a) and 2(b), respectively. It can be seen that, for a large correlation of the collateral pool, the re-weight approximation turns out to be not good at all.  

The above observation indicates that, when the correlation is weak, the re-weight approximation is quite good. However, when the correlation is strong, the re-weigh approximation remains good only when the shock is small enough. As indicated in Appendix VI, in both cases, the general trend of change remains the same, although the mezz tranches are not as sensitive as the two tranches shown in the Figs 1~2. 

Given the fact that most current trades have a correlation around 0.3 and the shock is less than 10 basis points, we conclude that re-weight approximation is acceptable.

In WMC simulation, the entropy is defined as

(4)	 ,

where N is the number of scenario and   is the weight for each scenario. It can be proved that by definition the regular MC simulation has zero entropy because  . As indicated in Ref. 2, a “wrong” solution corresponds to a case in which very large weights are assigned to a few paths, deviating from  significantly. This can be flagged by an exceptional large entropy value.

Based on the above testing results, we have concluded that SH3 is adequate for the computation of MTM and sensitivity tests for basket default swaps and Collateral Debt Obligations (CDOs) (FirstNofM, GiantFirstLoss, GiantFirstLossPayEnd, Caribou, and Reindeer). 

Tests results have shown that the sensitivity tests serve their intended purposes and are implemented correctly. The re-weigh approximation in the sensitivity tests appears to be quite good, when the perturbation is small and the correlation is weak. However, when the correlation becomes strong, we should be aware of the limitation of re-weight approximation.

In order to ensure consistency among all pricing templates, the settlement risk treatment in FirstNofM, GiantFirstLoss, and GiantFirstLossPayEnd are updated in SH3 and tested. Test results suggest that the change is indeed material for new trades while has no effect on the on-the-run trades. 

Tests have shown that the entropy threshold (=10) implemented in the credit library is appropriate. In model, sometimes WMC converges to an apparent “wrong” solution, although the probability tends to decrease as the number of scenarios increases. The threshold is thus set to identify a possible “wrong” solution. However, it should be noted that the entropy is only the output of the WMC. It is neither a controlling parameter of WMC nor an indication of how “good” the optimization is. It will be ideal if the problem can be solved fundamentally. 

It should also be noted that interest rate sensitivity, recovery rate sensitivity, and theta with re-weight approximation  (WMCIR, WMCRRS, and WMCCR) have been implemented in each pricing template.  


