First and foremost I would like to thank Marissa MacLellan, Mateusz Hojdysz, Steven Walt, Yi Kang, Aliaksandr Lebedzeu,  Ching Yi Sun, Fedor Baydakov, Yang Yang, Adam Jacobson, Boaz Wu, Lester Tam, and Mitchell Kideckel for their excellent contribution to this project. This project was supported by the Ben Graham Centre for Value Investing and Firenze Holdings.

The hash module in this repository is useful to aggregate accounting data and perform data sanitization to generate historical financial statements in Microsoft Excel. Given this fact, it is viable to build a program that automates pro-forma budgeting. While most calculations in pro-forma models are simple arithmetic, there are several key circular references that require identification of the difference equation in order to find the equilibrium solution. Normally, the Excel built-in iteration system is used to solve for circular reference. It cannot be used once the calculations are conducted off worksheet. Therefore, user-defined functions for solving circular references.

A mathematical approach to pro forma budgeting

The goal of a circular reference solving function is to find the stable equilibrium solution of a difference function in the form of 𝑓(𝑥, 𝑐, 𝑣), where 𝑥 represents the element that the function will solve, 𝑐 is a vector of constants, and 𝑣 is a vector of variables dependent on 𝑥 and 𝑐.

There are three parts of each set of circular reference solving functions:

1. The main body
2. The objective function
3. The optimization function

The main body’s purpose is to take parameters, initialize the optimization process, process singular calculations, and eventually return the equilibrium solution. The objective function is used to call the calculation process portion of the main body. The optimization function calculates the equilibrium solution by utilizing a numerical optimization method which will call the objective function as part of the iterative process.

I) Structure of an Objective Function

- Parameter Intake:

There are three kinds of parameters required for the Main Body Function: constants required to perform one iterative process, the value of the sought after element as optional to determine whether the Main Body Function is called for the first time, and miscellaneous optional parameters to determine the output and optimization set-ups.

- Calculation Process:

When the main body function is called for the first time (i.e. when there is no value assigned to the sought after element), the function will proceed to a subroutine to load the constants into a private global array (the subroutine is usually called LOAD_LINE). Then, the function will proceed into another subroutine (normally named SOLVE_LINE) to call the Optimization function which will return the equilibrium solution.

When the Main Body Function is called with a starting value for the sought after element, the Main Body Function will perform one calculation process and produce an ending value of the sought after element. When the absolute difference between the starting value and the ending value is less than an error term (normally less than 10^-4), the ending value is the equilibrium solution to the circular reference.

II) Structure of an Objective Function

- Parameter Intake

The Objective Function only takes in a value assigned to the sought after element.

- Calculation Process

The Objective Function will call the Main Body Function by passing the element value as the starting value of one calculation process.

III) Structure of an Optimization Function

- Parameter Intake

The Optimization Function takes in the name of the Objective Function in string format, and the initial guessing values.

- Calculation Process

The calculation process varies depending on the optimization method it utilizes; nevertheless, it always involves calling the Objective Function iteratively until there is convergence among the ending values of the sought after element.

IV) Choice of Optimization Method

Normally, there are two families of optimization methods: the Newton family and the Bisection family. A member of the Newton family is the Muller’s method, and a member for the Bisection family is the Nelder-Mead simplex method. The Newton family typically tries to estimate the equilibrium solution via polynomial curve fitting (area for 3-d, etc.) and has faster convergence speeds than the Bisection family. However, lack of convergence will occur if the epsilon is too small, too many calculation steps that increases the machine rounding error, or the function exhibits small variations with large input perturbation. The Bisection family searches the equilibrium solution through zoning. Therefore, once an equilibrium solution is located in a range, the optimization process will continuously narrow that range until the equilibrium solution is found; a method from the Newton’s family may exhibit explosive behavior if the objective function is not structured ‘nicely’.

In practice, Muller’s method is always preferred if it can produce viable results with a small enough error term; otherwise, the Nelder-Mead simplex method should be utilized to solve for the equilibrium point.

Examples:



A. Function for solving Implied Share Price (Executive Compensation and Stock Option)

Data Required

The required parameters are: number of option/warrant tranches, array of the option/warrant tranches with the first column as number of shares and the second column as exercise price, number of convertible security tranches, array of convertible security tranches with the first column as the tranche size and the second column as the conversion price, basic share outstanding, and the implied equity value.

Description of a Single Calculation Process

1. Calculate the total additional shares if all in the money options are exercised.
2. Calculate the total share repurchase based on exercised in the money options.
3. Calculate the number of additional share result from conversion or convertible
securities.
4. Total shares after dilution = basic shares outstanding + additional shares from option exercising + additional shares from convertible security conversion
5. Update the Implied Share Price Implied Share Price = Implied equity value / Total Share after dilution

Choice of Optimization Method

Upon testing, Muller’s method is viable with an error term of 10^-3, which is sufficient when calculating share price as the result is accurate at the cent level.




B. Function for solving cost of debt

Data Required

The required parameters are: an array of future operating lease commitments, current year and previous year of long term debt, short term debt, and capital lease obligation, current year interest expense, and effective interest rate from the previous year and two years ago.

Description of a Single Calculation Process

1. Aggregate the total present value of the operating lease commitments using a for loop
￼
2. Calculate the operating lease depreciation =
PV of operating lease commitments / Number of years of commitment
￼
3. Calculate the adjustment to operational earning
Operational earning adj. = PV of Operating Lease Next – Operating lease depreciation
4. Update capital lease obligation
Capital lease obligation Current = CLO Current + ￼PV of operating lease commitments

5. Calculate the updated current year of effective interest rate
Effective interest rate current = total interest payment / total amount of debt outstanding

6. (Optional) Update the cost of debt by averaging the three year trailing effective interest rate.

Choice of Optimization Method

Upon testing, Mullers method can produce an equilibrium solution with an error term of 10^-5, which is sufficient.
￼



C. Function for solving Cash Balance

Data Required

The parameter-intakes are segregated into three arrays – asset items, liability and equity items, net income items, and miscellaneous items – previous year total asset, effective tax rate, effective interest rate, LTD/last year assets, and money market interest rate.

Description of a Single Calculation Process
1. Calculate the value of specified asset – all asset items except cash
2. Calculate the value of long term debt based on current year and previous year total asset.
3. Calculate the interest expense based on long term debt.
4. Calculate the value of net income and addition to retained earnings, and then the updated value of retained earnings.
5. Calculate the value of specified liabilities – all liability and shareholder’s equity item except short term debt.
6. Calculate the value of net required financing
7. Finally derive an updated value for cash based on the net required financing.

Choice of Optimization Methods.

Upon testing, Muller’s method can produce an equilibrium solution with an error term of 10^-5, which is sufficient.




D. Function for solving Net Income

Data Required

The parameter-intakes are segregated into four arrays: non-debt and miscellaneous elements, revolver elements, term loan elements, non-term loan elements. Array is used instead of individual elements to maintain the total parameters intake under the limit of 30. The segregation of different type of debt related elements is to facilitate later calculations by determining the number of debt items in each category. This set is to ensure the flexibility of the function to accommodate changes of the debt structure.

Description of a Single Calculation Process

1. From a given value of Net Income, derive the drawdown/repayment of the revolver facility.
2. Calculate the optional payments to each term loan recursively based on the amount of cash leftover, with more description in the section below.
3. Calculate the total cash flow from financing activities, and therefore the cash balance.
4. Calculate interest income based on cash balance, interest expense based on revolver, term-loans, non-term loans balance, and commitment fee from unused revolver balance.
5. Finally derive an updated value for Net Income.

Calculating Optional Payments

The optional payments for each term loan are considered according to their rankings; for example, optional payment for subordinate debt is only considered if there is cash left over after paying the optional payment for senior debt. Therefore, recursive calculation is logically sound in this situation.
The optional payment is calculated for the most senior debt first; the function is exited after all the optional payments are calculated, in the order of seniority. The number of recursive calling is adjusted based on the number of term-loans to ensure the flexibility of this model.

Choice of Optimization Methods

The Nelder-Mead simplex method is chosen to solve for convergence in this case. The NM method is chosen to combat convergence issue at a cost of convergence speed compare to techniques such as Newton’s method or Muller’s method. The 𝑓 produces approximately 1000th of change in magnitude given a change in the input parameter, this phenomenon causes Newton’s and similar method will experience convergence issue for an acceptable epsilon (around 10^4, since the unit used in the model is in millions of dollars); they will produce a guess near the correct answer, and the subsequent guess will diverge. The NM method, however, once identifies the area that contains the correct answer, will stay in that area until a satisfactory answer is identified. Therefore, the NM method is used to solve for Net Income instead of Muller’s methods.

