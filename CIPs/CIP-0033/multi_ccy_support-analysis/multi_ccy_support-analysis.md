# Table of Contents

1. [Mental Model of the Dynamics to Maintain the Peg](#mental-model-of-the-dynamics-to-maintain-the-peg)
2. [Mathematical Model of the Dynamics](#mathematical-model-of-the-dynamics)
   1. [Equilibrium State](#equilibrium-state)
   2. [Independent Mentos](#independent-mentos)
   3. [Multi Mento](#multi_mento)
3. [Comparison of Speed of Convergence](#comparison-of-speed-of-convergence)
   1. [Model Parameter](#model-paramter)
   2. [Assumptions / Limitations](#assumptions-/-limitations)
   3. [Single Depeg](#single-depeg)
   4. [Double Depeg same Direction](#double-depeg-same-direction)
   5. [Double Depeg opposite Direction](#double-depeg-opposite-direction)
4. [Open Problems](#open-problems)
   1. [Independent Mentos - Open Problems](#independent-mentos---open-problems)
   2. [Multi Mento - Open Problems](#multi-mento---open-problems)

# Mental Model of the Dynamics to Maintain the Peg:

We are providing a mental model to get familiar with the different dynamics behind Independent Mentos and Multi Mento.

Assume cUSD/USD is depegged while cEUR/EUR is pegged.

**Independent Mentos:**

According to the stability protocol the on-Chain price will be overwritten with the Celo/USD price creating arbitrage opportunities. We believe that due to arbitrage trades the Celo and the cUSD supply in the market will change inversely proportionally. The prices of Celo/cUSD and Celo/USD will both change because they are sensitive to changes of the Celo and/or cUSD supply; Celo/cUSD should see faster price adjustments than Celo/USD as it is sensitive to Celo and cUSD while Celo/USD is only sensitive to Celo

Celo/EUR and Celo/cEUR are also sensitive to Celo supply. So both prices will change too. Likely, the price change will be similar, such that cEUR/EUR stays pegged.

**Multi Mento:**

The dynamics will be similar up to the point that trades in Celo and cUSD with Mento will affect the onChain Celo/cEUR price as the Celo/cEUR price is sensitive to the Celo tank. As a consequence Celo/cUSD trades can cause 'unintended' arbitrage opportunities for the Celo/cEUR pair; if this opportunity is taken the cEUR/EUR is likely to deviate from the peg temporarily to find the peg together with cUSD/USD again.

# Mathematical Model of the Dynamics

The model setup can be decomposed into two components: the on-chain market and the open market. The on-chain market is following the Mento mechanism. The on-chain prices of CELO at time <img src="svgs/4f4f4e395762a3af4575de74c019ebb5.svg?invert_in_darkmode" align=middle width=7.01538824999999pt height=23.89849409999998pt/> quoted in cUSD and cEUR are denoted by <img src="svgs/2b13a7f40ad287b7d7d69a4f6b515f10.svg?invert_in_darkmode" align=middle width=43.65656879999999pt height=29.14072199999998pt/> and <img src="svgs/235541b6c416ba628645ee07600d1e1b.svg?invert_in_darkmode" align=middle width=42.94017584999999pt height=29.14072199999998pt/>. On-chain cUSD, cEUR and CELO are represented by buckets. In the case of Independent Mentos each Mento mechanism features its own independent CELO bucket, in the case of Multi Mento the mechanism features only one CELO bucket. We denote the amount of cUSD, cEUR and CELO in each bucket at time <img src="svgs/4f4f4e395762a3af4575de74c019ebb5.svg?invert_in_darkmode" align=middle width=7.01538824999999pt height=23.89849409999998pt/> by

<p align="center"><img src="svgs/3454f86a197737004ceda8738244341a.svg?invert_in_darkmode" align=middle width=423.29912429999996pt height=54.55161075pt/></p>

At the time of an oracle update <img src="svgs/4f4f4e395762a3af4575de74c019ebb5.svg?invert_in_darkmode" align=middle width=7.01538824999999pt height=23.89849409999998pt/> the amount of CELO <img src="svgs/fa6d5af4b47e6d38004828a2b3298ae1.svg?invert_in_darkmode" align=middle width=45.20324639999998pt height=32.98742850000002pt/> is chosen as a fraction <img src="svgs/c8c26ca1f5fffbd6682651c557e977f1.svg?invert_in_darkmode" align=middle width=20.38508354999999pt height=26.982157799999985pt/>, <img src="svgs/0165638328af14a6a842e595a77576cc.svg?invert_in_darkmode" align=middle width=97.69409129999998pt height=29.14072199999998pt/> of the CELO amount in the reserve

<p align="center"><img src="svgs/34998b16128e8d9234123b04e33046c1.svg?invert_in_darkmode" align=middle width=483.32614005pt height=86.7813609pt/></p>

and the Celo Dollar amount is set to

<p align="center"><img src="svgs/5693ba131dd9bdd7ecbd5f170cd71266.svg?invert_in_darkmode" align=middle width=594.70269495pt height=54.55161075pt/></p>

Due to the constant-product market maker the following equations are satisfied at all times <img src="svgs/6f9bad7347b91ceebebd3ad7e6f6f2d1.svg?invert_in_darkmode" align=middle width=9.10647659999999pt height=16.728925200000024pt/> and <img src="svgs/4f4f4e395762a3af4575de74c019ebb5.svg?invert_in_darkmode" align=middle width=7.01538824999999pt height=23.89849409999998pt/> within one oracle update cycle:

<p align="center"><img src="svgs/417ae2986b1a1a6e0893ca1d6845d7d7.svg?invert_in_darkmode" align=middle width=687.57044265pt height=54.55161075pt/></p>

For the open market we are considering cUSD, cEUR, CELO, US Dollar and Euro each represented by a hypothetical tank with tank size <img src="svgs/570a71afd662ccebfa2b7b533eb2b873.svg?invert_in_darkmode" align=middle width=53.506258649999985pt height=29.14072199999998pt/>, <img src="svgs/8a3074808cf0aaa624ace9d2f54c7c36.svg?invert_in_darkmode" align=middle width=52.34328644999999pt height=29.14072199999998pt/>, <img src="svgs/959c4d1fce551779cfbf603b42cdc98b.svg?invert_in_darkmode" align=middle width=52.157102399999985pt height=29.14072199999998pt/>, <img src="svgs/953ab006a4b51d111c20921f52f3d869.svg?invert_in_darkmode" align=middle width=62.32893614999999pt height=29.14072199999998pt/> and <img src="svgs/c3906b7e4fb13d9d6772aad35d9625a1.svg?invert_in_darkmode" align=middle width=64.2176691pt height=29.14072199999998pt/>. We denote the market prices by <img src="svgs/7c335633c10604860e360e97babe0e9b.svg?invert_in_darkmode" align=middle width=48.803410499999984pt height=32.98742850000002pt/>, <img src="svgs/a2123b0a550204f8c7a8f43807c7746f.svg?invert_in_darkmode" align=middle width=48.08701559999999pt height=29.14072199999998pt/>, <img src="svgs/d0104543ad6c6f23c631ec6a461e894a.svg?invert_in_darkmode" align=middle width=65.32110974999998pt height=32.98742850000002pt/> , <img src="svgs/5544b14a046a83cca6715a1b59ef2c85.svg?invert_in_darkmode" align=middle width=64.90376099999999pt height=29.14072199999998pt/> and <img src="svgs/d9e89c29b99a58c8b6d2e470a7a3a80c.svg?invert_in_darkmode" align=middle width=64.90376099999999pt height=29.14072199999998pt/>. This convention follows the rule that a symbol '<img src="svgs/cbfb1b2a33b28eab8a3e59464768e810.svg?invert_in_darkmode" align=middle width=17.61935954999999pt height=26.550400500000023pt/>' denotes the price of currency in sub-script quoted in the currency in super-script. We are assuming that all arbitrage opportunities within the currency triangle of the open market will immediately vanish due to arbitrage trading. In particular, we are assuming

<p align="center"><img src="svgs/0dedfed65a0722d88e7ac1220684f82a.svg?invert_in_darkmode" align=middle width=649.84576995pt height=45.52082144999999pt/></p>

Tank sizes and market prices are related by the assumption that the relative sizes of the
tanks are given by:

<p align="center"><img src="svgs/6962dbfa0ca7316c4ecbe611e8b8336e.svg?invert_in_darkmode" align=middle width=537.1936401pt height=153.22920015pt/></p>

## Equilibrium State

Under the assumption that the Celo Dollar or Celo Euro price at time <img src="svgs/4f4f4e395762a3af4575de74c019ebb5.svg?invert_in_darkmode" align=middle width=7.01538824999999pt height=23.89849409999998pt/> have deviated from the peg it follows that the CELO price quoted in US Dollar <img src="svgs/d0104543ad6c6f23c631ec6a461e894a.svg?invert_in_darkmode" align=middle width=65.32110974999998pt height=32.98742850000002pt/> or in Euro <img src="svgs/5544b14a046a83cca6715a1b59ef2c85.svg?invert_in_darkmode" align=middle width=64.90376099999999pt height=29.14072199999998pt/> are different from the CELO price quoted in cUSD <img src="svgs/7c335633c10604860e360e97babe0e9b.svg?invert_in_darkmode" align=middle width=48.803410499999984pt height=32.98742850000002pt/> or cEUR <img src="svgs/a2123b0a550204f8c7a8f43807c7746f.svg?invert_in_darkmode" align=middle width=48.08701559999999pt height=29.14072199999998pt/>, respectively

<p align="center"><img src="svgs/b72e286d6d124417a15511b19328c9e8.svg?invert_in_darkmode" align=middle width=554.2939428pt height=101.48585849999999pt/></p>

By construction, at time <img src="svgs/4f4f4e395762a3af4575de74c019ebb5.svg?invert_in_darkmode" align=middle width=7.01538824999999pt height=23.89849409999998pt/> of an oracle update the on-chain CELO prices <img src="svgs/2b13a7f40ad287b7d7d69a4f6b515f10.svg?invert_in_darkmode" align=middle width=43.65656879999999pt height=29.14072199999998pt/> and <img src="svgs/235541b6c416ba628645ee07600d1e1b.svg?invert_in_darkmode" align=middle width=42.94017584999999pt height=29.14072199999998pt/> (quoted in Celo Dollar and Celo Euro) are equal to the market CELO prices <img src="svgs/d0104543ad6c6f23c631ec6a461e894a.svg?invert_in_darkmode" align=middle width=65.32110974999998pt height=32.98742850000002pt/> and <img src="svgs/5544b14a046a83cca6715a1b59ef2c85.svg?invert_in_darkmode" align=middle width=64.90376099999999pt height=29.14072199999998pt/>

<p align="center"><img src="svgs/3da92257edd5b12e933347a4e013d7a5.svg?invert_in_darkmode" align=middle width=350.65282199999996pt height=22.32185865pt/></p>

Therefore, the on-chain CELO prices <img src="svgs/2b13a7f40ad287b7d7d69a4f6b515f10.svg?invert_in_darkmode" align=middle width=43.65656879999999pt height=29.14072199999998pt/> and <img src="svgs/235541b6c416ba628645ee07600d1e1b.svg?invert_in_darkmode" align=middle width=42.94017584999999pt height=29.14072199999998pt/> are different from the market CELO prices <img src="svgs/7c335633c10604860e360e97babe0e9b.svg?invert_in_darkmode" align=middle width=48.803410499999984pt height=32.98742850000002pt/> and <img src="svgs/a2123b0a550204f8c7a8f43807c7746f.svg?invert_in_darkmode" align=middle width=48.08701559999999pt height=29.14072199999998pt/>

<p align="center"><img src="svgs/7e9dfcf58951c64e3f9089fd8f06d1e7.svg?invert_in_darkmode" align=middle width=317.31837735pt height=22.32185865pt/></p>

From the perspective of arbitrage traders four cases of price mismatches are profitable:

- <img src="svgs/a6c8754673d0d834c576ba97772db82c.svg?invert_in_darkmode" align=middle width=118.36263269999999pt height=32.98742850000002pt/> which is equivalent to <img src="svgs/9b0003d9b9537f816e87a1931d438bc2.svg?invert_in_darkmode" align=middle width=239.82004605pt height=32.98742850000002pt/>
- <img src="svgs/86b80e17081557b828f20d79fe1097b6.svg?invert_in_darkmode" align=middle width=118.36263269999999pt height=32.98742850000002pt/> which is equivalent to <img src="svgs/79688598d8daf4e0db5bc4238ef02a8c.svg?invert_in_darkmode" align=middle width=239.82004605pt height=32.98742850000002pt/>
- <img src="svgs/8429174a04e1614f191b4ff6593614bd.svg?invert_in_darkmode" align=middle width=116.92984679999999pt height=29.14072199999998pt/> which is equivalent to <img src="svgs/0d68c608211a1e4d5189d187e47a053d.svg?invert_in_darkmode" align=middle width=239.40269535pt height=32.98742850000002pt/>
- <img src="svgs/fa8283687b9545cc92e8be437e9ee092.svg?invert_in_darkmode" align=middle width=116.92984679999999pt height=29.14072199999998pt/> which is equivalent to <img src="svgs/879b53759e21bc847a438ad3cd117bf2.svg?invert_in_darkmode" align=middle width=238.98534464999997pt height=29.14072199999998pt/>

In four cases and combinations we are assuming that arbitrage traders will take advantage of the arbitrage opportunity until it vanishes at time <img src="svgs/f3ec94de872d1ee0320751f36fdfc33b.svg?invert_in_darkmode" align=middle width=14.386292699999988pt height=23.89849409999998pt/>, i.e. until <img src="svgs/75d6b526db6b9e6a6d97c664267ee4d0.svg?invert_in_darkmode" align=middle width=135.04710765pt height=32.98742850000002pt/> and <img src="svgs/955652e90789321f17e4b511f14c360d.svg?invert_in_darkmode" align=middle width=133.61431979999998pt height=29.14072199999998pt/>. We refer to this state as the equilibrium state.

## Independent Mentos

The equilibrium state of one single module Mento is described by the following set of equations of unknown time t_e quantities:

<p align="center"><img src="svgs/e872cdb7d58a9cca2890c85ecc86daff.svg?invert_in_darkmode" align=middle width=299.36645115pt height=119.01111105pt/></p>

We have chosen the Celo Dollar Mento but the set of equation in another currency is analogous. A more detailed analysis including fees is provided in [https://celo.org/papers/Celo_Stability_Analysis.pdf](https://celo.org/papers/Celo_Stability_Analysis.pdf) .

The set of equations of two Mento modules (cUSD and cEUR) is given by

<p align="center"><img src="svgs/3ea70267f1995aa7ed94aded04d3ee49.svg?invert_in_darkmode" align=middle width=442.90514774999997pt height=212.61133335pt/></p>

:warning: We have not been able so far to perform a feasable analysis of equilibrium dynamics based on these equations (see [Open Problems](#open-problems)). Instead we are trying to approximate the dynamics of Independent Mentos by assuming the Mento modules would operate sequentially.

This means, instead of allowing the two arbitrage cycles to be operating in parallel, we assume that the cUSD arbitrage cycle of the Celo Dollar Mento would operate and finish and then the cEUR arbitrage cycle would operate and finish.

## Multi Mento

The equilibrium state of Multi Mento is described by the following set of equations of unknown time <img src="svgs/f3ec94de872d1ee0320751f36fdfc33b.svg?invert_in_darkmode" align=middle width=14.386292699999988pt height=23.89849409999998pt/> quantities:

<p align="center"><img src="svgs/86ff88b399582e7fc7cc84f21fd51ab9.svg?invert_in_darkmode" align=middle width=325.2193932pt height=180.3815832pt/></p>

We fully solved this set of equation without fees. The case with fees is also solved apart from the case of one currency being above the peg and the second below.

# Comparison of Speed of Convergence

## Model Parameter

| Tanks            | Amount             |
| ---------------- | ------------------ |
| USD Market Tank  | 30000              |
| EUR Market Tank  | 20000              |
| cUSD Market Tank | 20000              |
| cEUR Market Tank | 20000, 40000, 5000 |
| Celo Market Tank | 10000              |
| Reserve          | 1000000            |
| Reserve Fraction | 0.01               |

## Assumptions / Limitations

- We were not able to set up a feasible model for Independent Mentos Therefore, we assume they would operate sequentially. This means, instead of allowing the two arbitrage cycles to be operating in parallel, we assume that the cUSD arbitrage cycle of the Celo Dollar Mento would operate and finish and then the cEUR arbitrage cycle would operate and finish.
- We assumed the same reserve fraction for cUSD Mento and cEUR Mento for Independent Mentos, but twice the reserve fraction for Multi Mento, because we wanted to have comparable overall Celo utilisation for the two approaches, i.e. the sum of the Celo tank in the cEUR Mento and the cUSD Mento is equal to single Celo tank in Multi Mento. (For uniform tank sizes across stable coins and approaches refer to [Additional Parameter Settings](multi_ccy_support-analysis_additional_parameter_settings.md))
- We set fees to zero, as we were not able to solve the set of equations for Multi Mento for a double depeg in opposite directions yet with fees yet.
- All observations have to be considered as statements that are only true
  within this model setup. They cannot be directly transferred to real market situations. However,
  they help to gauge expectations of the effect of the two approaches and to compare them against each other.

:warning: For additional parameter settings refer to the following sub-page:

[Additional Parameter Settings](multi_ccy_support-analysis_additional_parameter_settings.md)

## Single Depeg

cUSD/USD = 1.5 and cEUR/EUR = 1

![Scenario_Single_Depeg_cUSD-USD.png](pngs/Scenario_Single_Depeg_cUSD-USD.png)

The overall speed of convergence (<img src="svgs/9c2169ada0c95d4c963f3b8666aad4c4.svg?invert_in_darkmode" align=middle width=147.73836479999997pt height=29.14072199999998pt/>) is of the same scale. Multi Mento has faster short term convergence in terms of maximum deviation (<img src="svgs/0725b6d6cd05471d6a87af57cb11e093.svg?invert_in_darkmode" align=middle width=183.89472074999998pt height=29.14072199999998pt/>). In the tail, the speed of convergence of Independent Mentos is slightly faster overall as well as in terms of maximum deviation. As the on-chain price of cEUR is sensitive to cUSD transactions due to the shared CELO bucket, the cEUR price depegs because of cUSD arbitrage trades. Once cUSD and cEUR price have reached the same value, the price sensitivity is the same. We have also discussed this behavior in our [mental model of the dynamics.]()

## Double Depeg Same Direction

cUSD/USD = 1.5 and cEUR/EUR = 4

![Scenario_Double_Depeg_same_Direction.png](pngs/Scenario_Double_Depeg_same_Direction.png)

The overall speed of convergence (<img src="svgs/9c2169ada0c95d4c963f3b8666aad4c4.svg?invert_in_darkmode" align=middle width=147.73836479999997pt height=29.14072199999998pt/>) is of the same scale. Multi Mento has faster short term convergence in terms of maximum deviation (<img src="svgs/0725b6d6cd05471d6a87af57cb11e093.svg?invert_in_darkmode" align=middle width=183.89472074999998pt height=29.14072199999998pt/>). In the tail, the speed of convergence of Independent Mentos slightly faster overall as well as in terms of maximum deviation. As the on-chain price of cEUR is sensitive to cUSD transactions due to the shared CELO bucket, the depeg of cEUR price is further initially because of cUSD arbitrage trades. Once cUSD and cEUR have reached the same value, the price sensitivity is the same. We have also discussed this behavior in our [mental model of the dynamics.](#mental-model-of-the-dynamics-to-maintain-the-peg)

## Double Depeg Opposite Direction

cUSD/USD = 1.5 and cEUR/EUR = 0.5

![Scenario_Double_Depeg_opposite_Direction.png](pngs/Scenario_Double_Depeg_opposite_Direction.png)

In this case the speed of convergence of Multi Mento is faster than the speed of Independent Mentos in terms of sum of absolute values as well as maximum deviation. In this scenario the dynamic of Multi Mento is benefitting from cross-currency price sensitivity which adds speed to the convergence toward the peg. This is in contrast to the two cases we have discussed before, where cross-currency sensitivity increased the deviation from peg of one stable coin during the first update cycles until both prices matched.

# Open Problems

In this section we are listing problems of the mathematical model, that we have not been able to fully analyse yet:

- Simultaneous dynamics of Independent Mentos with and without fees
- Simultaneous dynamcs of Multi Mento with fees:
  - correct application of fees in case simultaneous expansion in one currency and contraction in the other currency.

## Independent Mentos - Open Problems

As discussed above we have not been able yet to make use of the solutions of the set of equations of the equilibrium state. We are introducing some further notation to shorten expressions in the equations above

<p align="center"><img src="svgs/29e2c8f19c0d8366a1ce1256a08d0e6f.svg?invert_in_darkmode" align=middle width=298.2070221pt height=264.09656715pt/></p>

where <img src="svgs/f5b547f9c43d2938d9dd62eed75955f3.svg?invert_in_darkmode" align=middle width=17.48475689999999pt height=26.982157799999985pt/> and <img src="svgs/2e7c71652eb2dff20576a8bf8b3e3577.svg?invert_in_darkmode" align=middle width=18.201128399999988pt height=26.982157799999985pt/> are the constant products of the initial cUSD and cEUR Mentos and <img src="svgs/e3dbc0462e2dfb11f9f13bd319e6f4a6.svg?invert_in_darkmode" align=middle width=38.74917344999999pt height=29.14072199999998pt/>, <img src="svgs/fc6e8ee3b9c044330025f3e3ac6f193b.svg?invert_in_darkmode" align=middle width=37.58620124999999pt height=29.14072199999998pt/>and <img src="svgs/6612c049b4b3b858aa7914eba81f9b00.svg?invert_in_darkmode" align=middle width=37.400017199999986pt height=29.14072199999998pt/> are the initial total amounts of cUSD, cEUR and CELO.

Starting with

<p align="center"><img src="svgs/ac201f20f7f10249d0bc1fd9f5552187.svg?invert_in_darkmode" align=middle width=147.3082416pt height=49.4948181pt/></p>

and by using the 'constant product' equation (fourth equation) and the sixth equation for the total number cEUR, we obtain

<p align="center"><img src="svgs/7f4c258032e1ad04a5804ef791dd6f4e.svg?invert_in_darkmode" align=middle width=375.1557264pt height=109.64884125pt/></p>

We retrieve a similar equation for cUSD

<p align="center"><img src="svgs/a02c60a0f5195caf38336d27e71b0ece.svg?invert_in_darkmode" align=middle width=279.18301710000003pt height=44.8165263pt/></p>

As <img src="svgs/235502ab7ad7b384cf1975383bf67f8b.svg?invert_in_darkmode" align=middle width=116.84815544999998pt height=32.98742850000002pt/> and <img src="svgs/b951517de72c4cb91bc12144e7c2d82a.svg?invert_in_darkmode" align=middle width=116.84815544999998pt height=32.98742850000002pt/> it follows

<p align="center"><img src="svgs/b75827a511b7997d191f074c0b2f3d71.svg?invert_in_darkmode" align=middle width=299.0687427pt height=125.1119493pt/></p>

Inserting these solutions into the quadratic expression yields the following solutions

<p align="center"><img src="svgs/c6241c12b5d295fcd15cf40f435823ed.svg?invert_in_darkmode" align=middle width=452.2220703pt height=108.94411319999999pt/></p>

and

<p align="center"><img src="svgs/a9b1d341422bc550bb5bb54bc941fd7d.svg?invert_in_darkmode" align=middle width=452.9384652pt height=108.94411319999999pt/></p>

Polynomial equations for the other unkown quantities (<img src="svgs/e7730a85b3bfc893697cd8d13d28bb7d.svg?invert_in_darkmode" align=middle width=54.894641099999994pt height=32.98742850000002pt/>, <img src="svgs/5b0a0edbc72504aee923c8b295db408b.svg?invert_in_darkmode" align=middle width=53.73166694999999pt height=32.98742850000002pt/>, <img src="svgs/4f8ce442ab4532f76df204d0e2508210.svg?invert_in_darkmode" align=middle width=60.49934084999999pt height=29.14072199999998pt/> etc.) at <img src="svgs/f3ec94de872d1ee0320751f36fdfc33b.svg?invert_in_darkmode" align=middle width=14.386292699999988pt height=23.89849409999998pt/> can be obtained from these two polynomials. In general, polynomials of degree 4 are solvable. Unfortunately, solutions are rather lenghty and often imaginary. We were not able to perform a feasable analysis of the equilibrium dynamics based on these solutions.

## Multi Mento - Open Problems

As mentioned above, the price of each currency can be in two states that will be profitable for an arbitrage trader. As a pair of two currency there can be four different scenarios that can be profitable

- one currency is depegged
- cUSD/USD and cEUR/EUR are above 1
- cUSD/USD and cEUR/EUR are below 1
- cUSD/USD is above 1 and cEUR/EUR is below 1 (or equivalently cUSD/USD is below 1 and cEUR/EUR is above 1)

We have been able to cover all of them in our analysis excluding fees and all but one scenario including fees.

The set equations of the scenario in which fees are included, one price is above 1 and the other one price is below 1 has not been solved yet.

### [WIP] cUSD/USD above 1 and cEUR/EUR below 1

<p align="center"><img src="svgs/86ff88b399582e7fc7cc84f21fd51ab9.svg?invert_in_darkmode" align=middle width=325.2193932pt height=180.3815832pt/></p>
