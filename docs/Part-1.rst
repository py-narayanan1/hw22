.. code:: ipython3

    %load_ext autoreload
    %autoreload 2


.. parsed-literal::

    The autoreload extension is already loaded. To reload it, use:
      %reload_ext autoreload
    

Added some imports in the whatif.py file here , since the python script
was not deciphering them, renamed whatif to hw22_visweswaran

.. code:: ipython3

    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns
    from itertools import product
    import copy

.. code:: ipython3

    from hw22_visweswaran import Model

.. code:: ipython3

    from hw22_visweswaran import get_sim_results_df

1A :Basic observation : Since the demand is being met , that means the
order quantity is same as the demand

.. code:: ipython3

    class SingleProductSPF(Model):
        """ Product SPF Model
            ----------------------
            
            *This model is based off the bookstore model in the whatif package
            * We place orders which are equal to the demand since the demand is always fulfilled
            * The demand is a quadratic function based off the selling price for each month per unit
            * The rest of the parameters are almost same as the bookstore model
            
            Attributes:
            ----------------------
            fixed_constant: fixed rate for creating the whole order irrespective of the number of units per month
                            float or array-like of float(default = 5000)
            var_cost : variable rate of ordering per month 
                        float or array-like of float(default = 100)
            selling_price: variable selling price for each unit per month
                         float or array-like of float(default = 115)
            spf_constant: 4900
            spf_linear: -35
            spf_quadratic 0.06 : All are fixed coefficients for the demand as a function of selling price
            """
        
        def __init__(self,fixed_cost = 5000,var_cost=100,selling_price=115,
                     spf_constant = 4900, spf_linear = -35,spf_quadratic = 0.06):
            self.fixed_cost = fixed_cost
            self.var_cost = var_cost
            self.selling_price = selling_price
            self.spf_constant = spf_constant
            self.spf_linear = spf_linear
            self.spf_quadratic = spf_quadratic
        """demand is based off a formula"""    
        def demand(self):
            return self.spf_quadratic*pow(self.selling_price,2)+self.spf_linear*self.selling_price+self.spf_constant
        """  order cost is based off demand and hence can be used here"""  
        def order_cost(self):
            return self.demand()*self.var_cost+self.fixed_cost
        """  sales revenue again is based off demand , since it is always met and hence can be used again"""  
        def sales_revenue(self):
            return self.demand()*self.selling_price
        """profit is just the  normal formula herr (No minimum , maximum concepts required)"""  
        def profit(self):
            return self.sales_revenue() - self.order_cost()

.. code:: ipython3

    base_model = SingleProductSPF()

Initial check passed

.. code:: ipython3

    base_model.profit()




.. parsed-literal::

    20027.5



1B: Creating the selling price ranges

Keeping the selling price with a step size of 10

.. code:: ipython3

    dt_param_ranges = {'selling_price':np.arange(80,141,10)}

Taking the outputs for the demand and profit

.. code:: ipython3

    outputs = ['demand','profit']

The base model once created , can be used to link all the methods ,
imported from the hw22_visweswaran.py file ,since it adds the methods
since our base model inherits all those methods.

.. code:: ipython3

    hw_model_df = base_model.data_table(dt_param_ranges,outputs)

.. code:: ipython3

    hw_model_df




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>selling_price</th>
          <th>demand</th>
          <th>profit</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>80</td>
          <td>2484.0</td>
          <td>-54680.0</td>
        </tr>
        <tr>
          <th>1</th>
          <td>90</td>
          <td>2236.0</td>
          <td>-27360.0</td>
        </tr>
        <tr>
          <th>2</th>
          <td>100</td>
          <td>2000.0</td>
          <td>-5000.0</td>
        </tr>
        <tr>
          <th>3</th>
          <td>110</td>
          <td>1776.0</td>
          <td>12760.0</td>
        </tr>
        <tr>
          <th>4</th>
          <td>120</td>
          <td>1564.0</td>
          <td>26280.0</td>
        </tr>
        <tr>
          <th>5</th>
          <td>130</td>
          <td>1364.0</td>
          <td>35920.0</td>
        </tr>
        <tr>
          <th>6</th>
          <td>140</td>
          <td>1176.0</td>
          <td>42040.0</td>
        </tr>
      </tbody>
    </table>
    </div>



As the demand varies quadratically with selling price and profit is
linearly related to demand , hence profit is also quadratically related
to selling price and hence it shows the same curved format with the
selling price

.. code:: ipython3

    sns.set_style("darkgrid")
    sns.scatterplot(x="selling_price", y="profit",data=hw_model_df, palette="viridis").set(title="Selling price vs Profit",xlabel="Selling Price",ylabel="Profit")




.. parsed-literal::

    [Text(0.5, 1.0, 'Selling price vs Profit'),
     Text(0.5, 0, 'Selling Price'),
     Text(0, 0.5, 'Profit')]




.. image:: output_19_1.png


1C: Goal_Seek , Break_Even Analysis

The break even selling price is 102.57.It id the point at which the
profit goes to 0 and also can be seen from the curve above.

.. code:: ipython3

    base_model.goal_seek( 'profit', 0, 'selling_price',80, 140, N=100)




.. parsed-literal::

    102.57578606424767



1D : 2 Way Data Table

For the 2 way data table again , the 2 inputs are varied and they are
combined in a form of list of dictionaries from the parametergrid built
in method

.. code:: ipython3

    dt_param_ranges_2_way = {'selling_price':np.arange(80,141,10),'var_cost':np.arange(85,111,5)}

.. code:: ipython3

    outputs_2way = ['order_cost','profit']

.. code:: ipython3

    hw_model_df_2 = base_model.data_table(dt_param_ranges_2_way,outputs_2way)

.. code:: ipython3

    hw_model_df_2.head()




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>selling_price</th>
          <th>var_cost</th>
          <th>order_cost</th>
          <th>profit</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>80</td>
          <td>85</td>
          <td>216140.0</td>
          <td>-17420.0</td>
        </tr>
        <tr>
          <th>1</th>
          <td>80</td>
          <td>90</td>
          <td>228560.0</td>
          <td>-29840.0</td>
        </tr>
        <tr>
          <th>2</th>
          <td>80</td>
          <td>95</td>
          <td>240980.0</td>
          <td>-42260.0</td>
        </tr>
        <tr>
          <th>3</th>
          <td>80</td>
          <td>100</td>
          <td>253400.0</td>
          <td>-54680.0</td>
        </tr>
        <tr>
          <th>4</th>
          <td>80</td>
          <td>105</td>
          <td>265820.0</td>
          <td>-67100.0</td>
        </tr>
      </tbody>
    </table>
    </div>



The plots make sense , since as the variable cost of production changes
the order cost would increase which would directly affect the profit
which reduces with the increasing variable cost of production

.. code:: ipython3

    profit_dt_g = sns.FacetGrid(hw_model_df_2, col="var_cost", sharey=True, col_wrap=3)
    profit_dt_g = profit_dt_g.map(plt.plot, "selling_price", "profit")



.. image:: output_30_0.png


1E - Change the range of the selling price now

.. code:: ipython3

    dt_param_ranges_e = {'selling_price':np.arange(80,251,10)}

.. code:: ipython3

    outputs_e = ['demand','profit']

.. code:: ipython3

    hw_model_df_e = base_model.data_table(dt_param_ranges_e,outputs_e)

From the graph it is evident that the profit goes to 0 twice in this
range

.. code:: ipython3

    sns.set_style("darkgrid")
    sns.scatterplot(x="selling_price", y="profit",data=hw_model_df_e, palette="viridis").set(title="Selling price vs Profit",xlabel="Selling Price",ylabel="Profit")




.. parsed-literal::

    [Text(0.5, 1.0, 'Selling price vs Profit'),
     Text(0.5, 0, 'Selling Price'),
     Text(0, 0.5, 'Profit')]




.. image:: output_36_1.png


The goal seek fails here , because if we take the left limit its target
is within its local scope and the target for the right limit also falls
within its local scope and since they both fall within the local scope,
their sign does not vary ,i.e. the right limit stays in its own local
scope with the same sign as the left limit when it is subtracted from
its local scope and hence they do not converge to the same 0 , but
differnt zeroes in their corresponding local scopes , giving a greater
than 0 result always and hence return nothing

.. code:: ipython3

    base_model.goal_seek( 'profit', 0, 'selling_price',80, 250, N=100)

1F

Keeping the seed the same from the whatif documentation

.. code:: ipython3

    from numpy.random import default_rng
    rg = default_rng(4470)
    rg.random() 




.. parsed-literal::

    0.45855804438027437



From the monte carlo simulation methods , it takes a uniformly
distributed variable cost

.. code:: ipython3

    random_inputs = {'var_cost': rg.uniform(80,120, 100)}

.. code:: ipython3

    sim_outputs = ['profit']

.. code:: ipython3

    model_results = base_model.simulate(random_inputs, sim_outputs)

.. code:: ipython3

    final_df = get_sim_results_df(model_results)

.. code:: ipython3

    final_df['profit']




.. parsed-literal::

    0     43371.982227
    1     39257.449247
    2     30530.501317
    3     -8959.808939
    4      5887.083797
              ...     
    95    51279.331055
    96    30365.628918
    97    -4502.041094
    98    40199.937827
    99     9950.802779
    Name: profit, Length: 100, dtype: float64



.. code:: ipython3

    plt.title("Profit histogram for one uncertain input")
    plt.xlabel("Profit")
    plt.ylabel("Num observations")
    plt.ylim(0, 17)
    plt.hist(final_df['profit'], density=False);



.. image:: output_48_0.png


.. code:: ipython3

    from scipy import stats

There is a 22% chance that the profit is negative

.. code:: ipython3

    print(stats.percentileofscore(final_df['profit'], 0) / 100.0)


.. parsed-literal::

    0.22
    

1F with scenarios

.. code:: ipython3

    scenario_inputs = {'selling_price': np.arange(80, 141, 10)}

.. code:: ipython3

    model_results_scenarios = base_model.simulate(random_inputs,sim_outputs,scenario_inputs)

.. code:: ipython3

    final_df_scenario = get_sim_results_df(model_results_scenarios)

.. code:: ipython3

    final_df_scenario




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>profit</th>
          <th>scenario_num</th>
          <th>selling_price</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>-19925.613514</td>
          <td>0</td>
          <td>80</td>
        </tr>
        <tr>
          <th>1</th>
          <td>-26051.175350</td>
          <td>0</td>
          <td>80</td>
        </tr>
        <tr>
          <th>2</th>
          <td>-39043.526957</td>
          <td>0</td>
          <td>80</td>
        </tr>
        <tr>
          <th>3</th>
          <td>-97835.214506</td>
          <td>0</td>
          <td>80</td>
        </tr>
        <tr>
          <th>4</th>
          <td>-75731.719418</td>
          <td>0</td>
          <td>80</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>95</th>
          <td>64067.062224</td>
          <td>6</td>
          <td>140</td>
        </tr>
        <tr>
          <th>96</th>
          <td>49326.568539</td>
          <td>6</td>
          <td>140</td>
        </tr>
        <tr>
          <th>97</th>
          <td>24750.973733</td>
          <td>6</td>
          <td>140</td>
        </tr>
        <tr>
          <th>98</th>
          <td>56258.032295</td>
          <td>6</td>
          <td>140</td>
        </tr>
        <tr>
          <th>99</th>
          <td>34937.694977</td>
          <td>6</td>
          <td>140</td>
        </tr>
      </tbody>
    </table>
    <p>700 rows × 3 columns</p>
    </div>



.. code:: ipython3

    sns.boxplot(x="selling_price", y="profit", data=final_df_scenario);



.. image:: output_57_0.png


Here each scenario of a different selling price is taken and for each
scenario a set of stats is generated for each set of variable cost and
we can see that as the selling price increases the profit goes more
towards the right which is bound to happen

.. code:: ipython3

    profit_histo_g2 = sns.FacetGrid(final_df_scenario, col='selling_price', sharey=True, col_wrap=3)
    profit_histo_g2 = profit_histo_g2.map(plt.hist, "profit")



.. image:: output_59_0.png


