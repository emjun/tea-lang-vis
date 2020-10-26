# [WIP] Tea Documentation

Tea is a domain specific programming language that automates statistical test
selection and execution. It's currently written as an open-source Python library
for integration into data analysis workflows using Python (3.6 or higher). 

Tea is designed for non-statistical experts who want to perform statistical analyses 
within their specific domain of expertise. If this is you, you're in the right place :)

<a href='https://pypi.org/project/tealang/'>Tea is available on pip!</a> You can
install Tea by typing the following terminal command into your console:
```
pip install tealang
```

Then, open whatever IDE you use for Python, and create a new Python file.
Make sure to add an import statement so that you'll have Tea ready to use
in your file:
```
import tea
```

You're all set! The next section will walk you through the steps of writing 
your own program in Tea.

## How do I write a Tea program?

You'll want to provide Tea with 5 pieces of information: 
* the *dataset* of interest, 
* the *variables* you want to analyze,
* the *study design* (e.g., specifying independent, dependent variables),
* any *assumptions* you may have about the data based on domain knowledge, and
* a *hypothesis*.

Let's unpack each of these in detail...

### Dataset
Tea accepts data either as a CSV or a Pandas DataFrame. Tea assumes data is in 
<a href='http://www.cookbook-r.com/Manipulating_data/Converting_data_between_wide_and_long_format/'>"long format."</a>

```
data_path = "./athlete_events.csv"
tea.data(data_path)
```

#### Designating a key:

Optionally, you may want a column in the dataset to serve as a relational (a.k.a. primary) key. 
This key should be an attribute that acts as a row identifier.

If no key is included, then all rows are treated as individual observations, i.e. all 
variables are between-subjects.

Including a key is helpful when participants have multiple entries in the dataset, i.e.
as for a within-subjects study.

To specify a key, simply add an argument when passing the dataset to Tea:

```
tea.data(data_path, key='ID')
```

The value included as the key name (`'ID'` in the example above) should be the name 
of a *variable* that you define from the dataset.

### Variables
Variables correspond to columns given in the dataset. You only need to define variables 
for the attributes of interest; it's fine to have columns in the dataset that are not
actually used in your Tea program.
 
You can describe a variable by including its column *name* and its *data type*.
Depending on the data type, you'll also need to include additional information 
(detailed below).

#### Defining data types:

Tea recognizes three data types: *nominal*, *ordinal*, and *numeric*.

A variable is *nominal* if its values represent discrete categories, and *ordinal* if 
these categories are ordered. These *categories* should be included in the variable description.
```
{
    'name' : 'Condition',
    'data type' : 'nominal',
    'categories' : ['AR', 'TV']
},
{
    'name' : 'Score',
    'data type' : 'ordinal',
    'categories' : [1,2,3,4,5]
}
```

A variable is *numeric* if it represents continuous values. A optional *range* can be 
included.
``` 
{
    'name' : 'Prob',
    'data type' : 'numeric',
    'range' : [0,1]             # optional
}
```
#### Describing variables for Tea:
You'll want to define your variables as a list, which you can then pass to Tea:
``` 
variables = [
    {
        'name' : 'Condition',
        'data type' : 'nominal',
        'categories' : ['AR', 'TV']
    },
    {
        'name' : 'Score',
        'data type' : 'ordinal',
        'categories' : [1,2,3,4,5]
    }
]
tea.define_variables(variables)
``` 

### Study Design

Next, you'll specify your study design by including the type of the study as well as the 
relevant variables of interest.

Tea supports 2 kinds of studies: *observational study* and *experiment*.

#### Specifying an observational study:
For an observational study, you should describe *contributor variables* and 
*outcome variables*. 
``` 
study_design = {
    'study type': 'observational study',
    'contributor variables': 'Sport',
    'outcome variables': 'Weight',
}
tea.define_study_design(study_design)
```

#### Specifying an experiment:
For an experiment, you should describe *independent variables* and *dependent variables*.
``` 
study_design = {
    'study type': 'experiment',
    'independent variables': 'Condition',
    'dependent variables': 'Score'
}
tea.define_study_design(study_design)
```
#### Including multiple variables:
If you'd like, multiple variables can be specified as a list. For example: 
``` 
experimental_design = {
    'study type': 'observational study',
    'contributor variables': ['Sport', 'Sex'],
    'outcome variables': 'Weight',
}
tea.define_study_design(study_design)
```
This can be useful if you're planning to test several hypotheses using different attributes.

### Assumptions
If you have domain knowledge that would be relevant to testing your hypothesis, 
you can provide this information to Tea explicitly.

*Note: This step is optional! You don't have to include any assumptions in order 
for Tea to run.*

#### Describing a variable's statistical properties:

A *numeric* variable could be normally distributed across all entries:
```
assumptions = {
    'normal distribution': [['Prob']]
}
```
You could also assume a variable's normal distribution within the categories of 
another variable, as shown below.
```
assumptions = {
    'groups normally distributed': [['So', 'Prob']]
}
```
Here, `So` is a nominal variable indicating 
whether a state is Southern or non-Southern. `Prob` is assumed to be normally 
distributed within the group of Southern states, as well as within the non-Southern 
group.

Additionally, a *numeric* variable can be assumed to have equal variance across
the categories of another variable. Below, we assume that the variance of 
`Prob` within the group of Southern states is equal to its variance within the non-Southern 
states. 
```
assumptions = {
    'equal variance': [['So', 'Prob']]
}
```
Multiple assumptions of the same type can be entered as a list:
```
assumptions = {
    'normal distribution': [['Prob'], ['Ineq']]
    'equal variance': [['So', 'Prob'], ['So', 'Ineq']]
}
```

#### Specifying applicable data transformations:
Assumptions could also include any known data transformations that can 
be applied to a variable. Tea currently supports log transformations on a *numeric* variable.
```
assumptions = {
    'log normal distribution': [['Ineq']]
}
```

#### Providing an alpha value: 
As part of the assumptions, you can also specify the *Type I (False Positive) Error 
Rate* (a.k.a. "significance threshold", or *alpha*), which represents the largest p-value at which the null 
hypothesis should be rejected. The default alpha value is 0.05.
``` 
assumptions = {
    'Type I (False Positive) Error Rate': 0.01
}
```
In the above example, the resulting p-value must be *less* than 0.01 in order to 
reject the null hypothesis.

#### Passing assumptions to Tea:
You can declare all assumptions together before passing them to Tea, as shown below:
```
assumptions = {
    'groups normally distributed': [['So', 'Prob']],
    'Type I (False Positive) Error Rate': 0.05,
}
tea.assume(assumptions)     # strict mode by default
```  
Note: Tea checks your assumptions about variables' properties and will issue warnings 
for those that are not verified by statistical testing. To specify how Tea should handle 
these cases, you can select one of two modes: *strict* (default) or *relaxed*. In *strict* mode, 
Tea will override user assumptions if they are determined unverifiable. In *relaxed* 
mode, Tea will proceed with all user assumptions regardless of whether or not they
are verified.

This mode can be set as shown:
```
tea.assume(assumptions, 'relaxed')     # select relaxed mode
```

### Hypothesis

Hypotheses in Tea include the variables of interest, as well as the hypothesized relationship between them.

#### Describing different hypothesis types:
Tea currently allows you to express the following range of hypotheses:
* One-sided comparison between groups
```
tea.hypothesize(['Region', 'Imprisonment'], ['Region: Southern > Northern']) 
```
Here, `Region` is the name of a *nominal* or *ordinal* variable that has `Southern` 
and `Northern` as *categories*. This hypothesis describes a higher rate of `Imprisonment` 
(an *ordinal* or *numeric* variable) in the `Southern` category compared to the `Northern`
category.

##### Partial orders
```
tea.hypothesize(['Region', 'Imprisonment'], ['Region: Southern > Southwest', 'Region: Northeast > Midwest']) 
```
Tea also allows your hypothesis to include multiple comparisons, which can be useful if 
you are analyzing multiple independent variables (e.g., `Region` has several distinct categories) 
and one dependent variable (e.g., `Imprisonment`).

When you provide these comparisons as a single hypothesis, Tea will take them into consideration 
for performing corrections, such as adjusting the p-value, in order to minimize a false 
discovery rate.

##### Two-sided comparisons
```
tea.hypothesize(['Region', 'Imprisonment'], ['Region: Southern != Northern']) 
```
A two-sided comparison allows for directionality in two ways: in order for this 
hypothesis to hold, `Imprisonment` could be such that either `Southern > Northern` 
or `Southern < Northern`. 

##### Positive linear relationships
```
tea.hypothesize(['Region', 'Imprisonment'], ['Imprisonment ~ Region'])  # positive by default 
tea.hypothesize(['Region', 'Imprisonment'], ['Imprisonment ~ +Region']) 
```
Here, both `Region` and `Imprisonment` are *ordinal* or *numeric* variables. 
This hypothesis describes a positive correlation between the two&mdash;i.e., 
`Imprisonment` increases as the `Region` increases.

##### Negative linear relationships
```
tea.hypothesize(['Region', 'Imprisonment'], ['Imprisonment ~ -Region']) 
```
Similarly, you could describe a negative relationship between two *ordinal* or 
*numeric* variables.

Note: for linear relationships, it does not matter which variable is declared as
independent vs. dependent when defining the study design.

#### Providing multiple hypotheses:
As long as all variables of interest are defined and included in the study design,
you can specify as many hypotheses as you'd like:
``` 
results_1 = tea.hypothesize(['Sport', 'Weight'], ['Sport:Wrestling < Swimming'])
results_2 = tea.hypothesize(['Sex', 'Weight'], ['Sex:F < M'])
```

#### Testing against the Null Hypothesis:
Tea currently supports Null Hypothesis Significance Testing (NHST), which tests your 
hypothesis against the *null hypothesis* that the entries in your dataset occurred by 
random chance.

The statistical tests selected by Tea will determine whether the null hypothesis should 
be accepted or rejected&mdash;i.e., whether you can conclude that there is indeed a relationship   
between the variables specified in your hypothesis.

## How do I interpret the output of my Tea program?

At runtime, Tea "compiles" the above 5 pieces of information into logical constraints, 
which it uses to select and execute a set of valid statistical tests for testing your hypothesis.

#### Selecting valid tests:
A test is considered valid if and only if *all* the assumptions it makes about 
the data (e.g., normal distribution, equal variance between groups, etc.) hold.

Tea outputs its decision process for every statistical test it considers.

Here's an example of what the output could look like for a statistical test 
that Tea finds to be *invalid*:
```
Currently considering kendalltau_corr
Testing assumption: is_bivariate.
Property holds.
Testing assumption: is_continuous_or_ordinal.
Property FAILS
```
Once Tea checks a certain assumption, it remembers whether or not that property 
holds. This allows Tea to more quickly evaluate other tests that require the same 
assumption(s). For instance, the following test was directly deemed unsatisfiable 
because Tea had already found that its properties failed: 
```
Currently considering spearman_corr
Test is unsat.
```
A *valid* statistical test would have output showing that all required 
assumptions are satisfied:
```
Currently considering mannwhitney_u
Testing assumption: has_one_x.
Property holds.
Testing assumption: has_one_y.
Property holds.
Testing assumption: has_independent_observations.
Property holds.
Testing assumption: is_categorical.
Property holds.
Testing assumption: has_two_categories.
Property holds.
Testing assumption: is_continuous_or_ordinal.
Property holds.
```

#### Executing valid tests:
The moment you've been waiting for... Tea will execute each valid statistical test
to test your hypothesis!

These results are displayed as output and also returned by Tea's `hypothesize()` function. For each test, the 
results include the name of the test, its assumptions about the variables, and the 
calculated statistics.

For example, the results of executing the Student's T Test would look something 
like this:
``` 
Test: students_t
***Test assumptions:
Exactly two variables involved in analysis: So, Prob
Exactly one explanatory variable: So
Exactly one explained variable: Prob
Independent (not paired) observations: So
Variable is categorical: So
Variable has two categories: So
Continuous (not categorical) data: Prob
Equal variance: So, Prob
Groups are normally distributed: So, Prob

***Test results:
name = Student's T Test
test_statistic = 4.20213
p_value = 0.00012
adjusted_p_value = 0.00006
alpha = 0.05
dof = 45
Effect size:
Cohen's d = 1.24262
A12 = 0.83669
Null hypothesis = There is no difference in means between So = 0 and So = 1 on Prob.
Interpretation = t(45) = 4.20213, p = 0.00006. Reject the null hypothesis at alpha = 0.05. The mean of Prob for So = 1 (M=0.06371, SD=0.02251) is significantly greater than the mean for So = 0 (M=0.03851, SD=0.01778). The effect size is Cohen's d = 1.24262, A12 = 0.83669. The effect size is the magnitude of the difference, which gives a holistic view of the results [1].
[1] Sullivan, G. M., & Feinn, R. (2012). Using effect size—or why the P value is not enough. Journal of graduate medical education, 4(3), 279-282.
```

Tea then states whether or not the null hypothesis should be rejected based 
on the results for that particular test execution.

## Examples

To help you become more familiar with Tea, we've 
provided some sample Tea programs with their respective datasets: 

* [`ar_tv_tea.py`](./examples/AR_TV/ar_tv_tea.py) (dataset: [`ar_tv_long.csv`](./examples/AR_TV/ar_tv_long.csv))

* [`crime_tea.py`](./examples/US_Crime/crime_tea.py) (dataset: [`USCrime.csv`](./examples/US_Crime/UScrime.csv))

* [`olympics_tea.py`](./examples/Olympics/olympics_tea.py) (dataset: [`athlete_events.csv`](./examples/Olympics/athlete_events.csv))

* [`drug_small_tea.py`](./examples/Drugs/drug_small_tea.py) (Pandas DataFrame using a subset of this dataset: [`drug.csv`](./examples/Drugs/drug.csv))

* [`co2_tea.py`](./examples/Co2/co2_tea.py) (dataset: [`co2.csv`](./examples/Co2/co2.csv))
 

## How does Tea decide which statistical analysis tests to run?
[TODO]
