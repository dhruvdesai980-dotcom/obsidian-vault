```
does stg bring any info from eq_position or fi_position

cdo will need to write SQL knowledge to do roll up 
daily to monthly > they will have dbt Wizard to do it..
they can ask wizard can do the rollup , can we confirm it can be done .. for stg_layer as well.

using anchors can we reuse entire position or its fields .. not just description .. and other inner fields

can anchors go from stg to semantics..

how can i carry forward the meta info from eq and fi semantics models into the common stg layer for cdo position.. so tht it can eventually make its way to 
cdo position

when we do stg_layer funk to do diff rollup
will that actually create a mat view in databricks only

anchor for one attribute 
anchor for one metric 

can you override the parts of anchor..
to bring your own columns for the metric participants..

take a metric that is based on two colujmns
anchor it

then try to use it but change the column name to something else..and see the formula still can be used..,

Also, one thing I want to highlight is:

Stratey team keeps on breaking something while fixing something

for example, free-form sql

unity metrics.. in databricks.. lets check that. 

----
semantics built into BI  - mosaic models
semantics built into Databricks - unity metrics
dbt is vendor agnostic. 
collibra --

we will do the build out for all 3 and take it to execs

customer-> positions 

smae ER in all of them.

data-first / contract-first 

feature matrix 

- data first vs contract first 
- semantic coverage --> 
		shared semantics ?
		  if yes then what coverage
		  --> business definitions, 
          --> reusable metrics 
          --> ontology support ?
       --> impact analysis
       --> version control
       --> integration with Bi tools 
           --> which tools 	   
		reusable metrics
- shared glossary 
- reusable metrics
- enforceability via contracts 
- contract first assumptions are right..

-data first then mosaic
--contract first then dbt
--platform agnostic then also dbt


-----can exposure show i depend on the full semantic model 
```