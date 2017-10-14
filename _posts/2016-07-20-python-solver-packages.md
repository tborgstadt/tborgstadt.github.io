---
layout: post
title: Solvers for Portfolio Optimization of Exchange Traded Funds
description: Optimizing a portfolio of Exchange Traded Funds (ETFs)
tags: [non-linear optimization, cvxopt, cvxpy]
comments: True
---
This project explores two Python packages for solving quadratic equations using Markowitz portfolio optimization as a metaphor. The popular Microsoft Excel Solver add-in is used to compare and verify results. The long-term objective of this research and development is to progress a fully automated optimization tool to aid in managing asset position sizes in a long-term investment portfolio of ETFs. The benefit of such a tool takes the guesswork out of picking assets in which to invest by using a systematic approach using mathematics, statistics, and portfolio theory to make decisions.

This project is the beginning of longer term project for me. The objective here was to simply do some preliminary investigation of solver type packages that could be used to optimize non-linear quadratic problems with Python.

* TOC
{:toc}

### Data Description

Historical ETF market data from Yahoo Finance is used in this analysis. The advantage of using ETFs versus stocks is that each ETF is a basket of assets (like mutual funds) and therefore is diversified. However, unlike mutual funds ETFs trade like stocks and have much lower expenses. Another powerful advantage of using ETFs is the number available and the diversity of ETFs that can be found in any area of interest whether an index, sector, particular industry, commodity, fixed, or currency. If the reader prefers investing in stocks only, the work completed here still applies - simply use the ticker symbols for your desired stocks.

The following ETFs are used in this analysis:

|**Ticker**|**Name**|**Category**|
|SPY|SPDR S&P 500 ETF Trust|Equities - Large Cap|
|SH|ProShares Short S&P500 ETF|Equities - Large Cap|
|MDY|SPDR S&P MidCap 400 ETF|Equities - Mid Cap|
|VB|Vanguard Small-Cap Index Fund|Equities - Small Cap|
|VEU|Vanguard FTSE All-World ex-US ETF|Equities - Global|
|AGG|iShares Barclays Aggregate Bond Fund|Fixed Income|
|TLH|iShares Lehman 10-20 Yr Treasury Bond ETF|Fixed Income|
|RWR|SPDR Dow Jones REIT ETF|Real Estate|
|GLD|SPDR Gold Trust ETF|Commodity|

Daily market adjusted closing price from July 1, 2015, to July 1, 2016, is retrieved from Yahoo Finance for each ETF. The Python pandas package is utilized to generate returns. Returns are calculated based on the last day of the business month, so we have a 12 period series for each ETF.


|**Returns**|**SPY**|**SH**|**MDY**|**VB**|**VEU**|**AGG**|**TLH**|**RWR**|**GLD**|
|8/31/2015|-0.061|0.0591|-0.0566|-0.0583|-0.0775|-0.0034|-0.0054|-0.0591|0.0371|
|9/30/2015|-0.0255|0.0209|-0.032|-0.0454|-0.0403|0.0081|0.0183|0.0339|-0.018|
|10/30/2015|0.0851|-0.0804|0.0555|0.0568|0.0669|0.0007|-0.0078|0.0579|0.0228|
|11/30/2015|0.0037|-0.0053|0.0134|0.018|-0.0132|-0.0039|-0.004|-0.0061|-0.0675|
|12/31/2015|-0.0173|0.0136|-0.042|-0.0419|-0.0252|-0.0019|-0.0036|0.022|-0.0045|
|1/29/2016|-0.0498|0.0494|-0.0554|-0.0755|-0.056|0.0124|0.037|-0.0406|0.0541|
|2/29/2016|-0.0008|-0.0023|0.0124|0.0077|-0.0244|0.0089|0.0186|-0.0085|0.1093|
|3/31/2016|0.0673|-0.0659|0.0848|0.0846|0.083|0.0087|0.0013|0.1036|-0.0084|
|4/29/2016|0.0039|-0.0049|0.0119|0.0177|0.0211|0.0026|-0.0038|-0.0296|0.0511|
|5/31/2016|0.017|-0.0187|0.0225|0.0189|-0.0079|0.0001|0.0029|0.0198|-0.0614|
|6/30/2016|0.0035|-0.005|0.0047|0.0033|-0.0073|0.0194|0.0418|0.0646|0.0897|
|7/29/2016|0.0021|-0.0023|0.002|0.003|0.003|0.0022|0.0071|0.0013|0.0153|
|ER|0.0023|-0.0035|0.0018|-0.0009|-0.0065|0.0045|0.0085|0.0133|0.0183|
|SD|0.0417|0.0401|0.043|0.0471|0.0463|0.0071|0.0168|0.047|0.0543|


### Exploratory Data Analysis

For purposes of general evaluation of the packages involved, it is assumed that Yahoo Finance data is relatively clean and accurate, so minimal EDA was performed. However, a correlation matrix was created and inspected to ensure the returns calculated were reasonable. For example, Treasury bills (TLH) are inversely correlated with the S&P 500 (SPY).

<figure><a><img src="/images/20160720_corr.png"></a>
<figcaption></figcaption>
</figure>



### Process and Results
First, the portfolio frontier was simulated and visualized by generating 1500 different portfolios by randomly generating asset weights and calculating the portfolio expected return and risk for the given ETFs.

<figure><a><img src="/images/20160720_cvxopt_ef1.png"></a>
<figcaption></figcaption>
</figure>

Once the portfolio frontier was created, the efficient frontier could be added by fitting a 2nd-degree polynomial to the data and generating expected returns along the curve.

<figure><a><img src="/images/20160720_cvxopt_ef2.png"></a>
<figcaption></figcaption>
</figure>

Portfolios with fewer assets, or ETFs, generated more visually appealing frontiers. For example, the following visualization was generated using only 3 ETFs.  As the portfolio increases in the number of assets, the portfolio frontier seems to become less well defined which deserves additional research as to why this might be.

<figure><a><img src="/images/20160720_2cvxopt_ef2.png"></a>
<figcaption></figcaption>
</figure>

Two Python solver packages were installed and implemented to evaluate performance and accuracy compared to results provided by the Excel Solver add-in for the same portfolio of ETFs. Both the Python packages are open source and free.

* [CVXOPT](http://cvxopt.org){:target="blank"} was developed at MIT by J.Dahl, M.Anderson, and L. Vandenberghe beginning in 2004. The latest version was released in 2015.

* [CVXPY](http://www.cvxpy.org/){:target="blank"} was developed by Steven Diamond and Stephen Boyd. It was published in the Journal of Machine Learning Research. And actually it is not a solver in and of itself but rather transforms a problem into a standard form before calling a solver and ultimately deliveres the results from the solver in a standard form.

All three solvers provided identical results. CVXOPT has a sophisticated constraint matrices setup and should be able to handle most any constraint combination. This analysis was minimal and more time will be spent in future work to explore and identify any constraint limitations.

|**Objective 1:**|Maximize Portfolio Return (ERP)|
|**Constraints:**|Sum(Weights) = 1; At least 5% investment in each ETF|

|**Results:**|Excel Solver|CVXOPT|CVXPY|Weights|
|ER|0.011955|0.011955|0.011955|60% GLD, 5% all others|
|SD|0.032858|0.032858|0.032858||


|**Objective 2:**|Minimize Portfolio Risk (SDP)|
|**Constraints:**|Sum(Weights) = 1; At least 5% investment in each ETF|

|**Results:**|Excel Solver|CVXOPT|CVXPY|Weights|
|ER|0.001003|na|0.001003|60% GLD, 5% all others|
|SD|0.004296|na|0.004296|23% SPY, 42% SH, 5% All Others|


### Conclusion

The two Python based solvers evaluated in this very *limited* analysis performed as expected. CVXOPT is a mature package but is equally more sophisticated, whereas CVXPY is more intuitive when it comes to setting up and defining an optimization problem in Python. In other words, it abstracts the extensive matrices mathematics knowledge required by CVXOPT to configure problem constraints. For this reason, CVXPY has a much lower learning curve. Both solvers provided identical results when compared to the Excel Solver add-on.

This project resulted in an automated tool (see included Python HTML notebook) for compiling returns and generating differently weighted portfolios for a set of assets, namely ETFs or stocks. This is the first piece of a larger project to develop a method for rebalancing a long-term investment portfolio that I am progressing and will post additional followup.

Future objectives given this fundamental groundwork include:

* Measure impact of time period for returns based on rebalancing frequency
* Tracking changes in optimal portfolio weights to predict future weights
* Capability to backtest

### Notebook

Jupyter Notebook: [Portfolio Optimization Solvers](/notebooks/20160720-portfolio-optimization-solvers.html){:target="_blank"}.

### References

Starke, Thomas, Ph.D. "The Efficient Frontier: Markowitz Portfolio Optimization in Python." Quantopian. N.p., 9 Mar. 2015. Web. 15 July 2016.


