<H3> Name : MAMTHA I</H3>
<H3>Register No. 212222230076</H3>
<H3> Experiment 1</H3>
<H3>DATE:</H3>
<H1 ALIGN=CENTER> Implementation of Bayesian Networks</H1>

## Aim :

    To create a bayesian Network for the given dataset in Python
    
## Algorithm:

Step 1:
Import necessary libraries: pandas, networkx, matplotlib.pyplot, Bbn, Edge, EdgeType, BbnNode, Variable, EvidenceBuilder, InferenceController<br/>
Step 2:
Set pandas options to display more columns<br/>
Step 3:
Read in weather data from a CSV file using pandas<br/>
Step 4:
Remove records where the target variable RainTomorrow has missing values<br/>
Step 5:
Fill in missing values in other columns with the column mean<br/>
Step 6:
Create bands for variables that will be used in the model (Humidity9amCat, Humidity3pmCat, and WindGustSpeedCat)<br/>
Step 7:
Define a function to calculate probability distributions, which go into the Bayesian Belief Network (BBN)<br/>
Step 8:
Create BbnNode objects for Humidity9amCat, Humidity3pmCat, WindGustSpeedCat, and RainTomorrow, using the probs() function to calculate their probabilities<br/>
Step 9:
Create a Bbn object and add the BbnNode objects to it, along with edges between the nodes<br/>
Step 10:
Convert the BBN to a join tree using the InferenceController<br/>
Step 11:
Set node positions for the graph<br/>
Step 12:
Set options for the graph appearance<br/>
Step 13:
Generate the graph using networkx<br/>
Step 14:
Update margins and display the graph using matplotlib.pyplot<br/>

## Program:

***IMPORTING PYTHON PACKAGES :***

```Python
import pandas as pd # for data manipulation
import networkx as nx # for drawing graphs
import matplotlib.pyplot as plt # for drawing graphs
```
***for creating Bayesian Belief Networks (BBN)***
```Python
from pybbn.graph.dag import Bbn
from pybbn.graph.edge import Edge, EdgeType
from pybbn.graph.jointree import EvidenceBuilder
from pybbn.graph.node import BbnNode
from pybbn.graph.variable import Variable
from pybbn.pptc.inferencecontroller import InferenceController
```

***Set Pandas options to display more columns***
```Python
pd.options.display.max_columns=50
```
***READING THE DATA SET :***
```Python
df=pd.read_csv('/content/weatherAUS.csv', encoding='utf-8')
```
***Drop records where target RainTomorrow=NaN***
```Python
df=df[pd.isnull(df['RainTomorrow'])==False]
```
***Drop the 'Date' column as it is not relevant for the model***
```Python
df = df.drop(columns='Date')
```

***For other columns with missing values, fill them in with column mean for numeric columns only***
```Python
numeric_columns = df.select_dtypes(include=['number']).columns
```
***Use .loc to explicitly modify the original DataFrame***
```Python
df.loc[:, numeric_columns] = df[numeric_columns].fillna(df[numeric_columns].mean())
```

***Create bands for variables that we want to use in the model***
```Python
df['WindGustSpeedCat']=df['WindGustSpeed'].apply(lambda x: '0.<=40'   if x<=40 else
                                                            '1.40-50' if 40<x<=50 else '2.>50')
df['Humidity9amCat']=df['Humidity9am'].apply(lambda x: '1.>60' if x>60 else '0.<=60')
df['Humidity3pmCat']=df['Humidity3pm'].apply(lambda x: '1.>60' if x>60 else '0.<=60')
```
***Show a snaphsot of data***
```Python
print(df)
```

***This function helps to calculate probability distribution, which goes into BBN (note, can handle up to 2 parents)***
```Python
def probs(data, child, parent1=None, parent2=None):
    if parent1==None:
        # Calculate probabilities
        prob=pd.crosstab(data[child], 'Empty', margins=False, normalize='columns').sort_index().to_numpy().reshape(-1).tolist()
    elif parent1!=None:
            # Check if child node has 1 parent or 2 parents
            if parent2==None:
                # Caclucate probabilities
                prob=pd.crosstab(data[parent1],data[child], margins=False, normalize='index').sort_index().to_numpy().reshape(-1).tolist()
            else:
                # Caclucate probabilities
                prob=pd.crosstab([data[parent1],data[parent2]],data[child], margins=False, normalize='index').sort_index().to_numpy().reshape(-1).tolist()
    else: print("Error in Probability Frequency Calculations")
    return prob
```
***Create nodes by using our earlier function to automatically calculate probabilities***
```Python
H9am = BbnNode(Variable(0, 'H9am', ['<=60', '>60']), probs(df, child='Humidity9amCat'))
H3pm = BbnNode(Variable(1, 'H3pm', ['<=60', '>60']), probs(df, child='Humidity3pmCat', parent1='Humidity9amCat'))
W = BbnNode(Variable(2, 'W', ['<=40', '40-50', '>50']), probs(df, child='WindGustSpeedCat'))
RT = BbnNode(Variable(3, 'RT', ['No', 'Yes']), probs(df, child='RainTomorrow', parent1='Humidity3pmCat', parent2='WindGustSpeedCat'))
```

***Create Network***
```Python
bbn = Bbn() \
    .add_node(H9am) \
    .add_node(H3pm) \
    .add_node(W) \
    .add_node(RT) \
    .add_edge(Edge(H9am, H3pm, EdgeType.DIRECTED)) \
    .add_edge(Edge(H3pm, RT, EdgeType.DIRECTED)) \
    .add_edge(Edge(W, RT, EdgeType.DIRECTED))
```

***Convert the BBN to a join tree***
```Python
join_tree = InferenceController.apply(bbn)
```
***Set node positions***
```Python
pos = {0: (-1, 2), 1: (-1, 0.5), 2: (1, 0.5), 3: (0, -1)}
```

***Set options for graph looks***
```Python
options = {
    "font_size": 16,
    "node_size": 4000,
    "node_color": "white",
    "edgecolors": "black",
    "edge_color": "red",
    "linewidths": 5,
    "width": 5,}
```

***Generate graph***
```Python
n, d = bbn.to_nx_graph()
nx.draw(n, with_labels=True, labels=d, pos=pos, **options)
```

***Update margins and print the graph***
```Python
ax = plt.gca()
ax.margins(0.10)
plt.axis("off")
plt.show()
print(probs(df, child='Humidity9amCat'))
print(probs(df, child='Humidity3pmCat', parent1='Humidity9amCat'))
print(probs(df, child='WindGustSpeedCat'))
print(probs(df, child='RainTomorrow', parent1='Humidity3pmCat', parent2='WindGustSpeedCat'))
```
## Output:
![image](https://github.com/user-attachments/assets/46fe24ea-f596-450f-83d7-8557492c70c2)

![image](https://github.com/user-attachments/assets/c806c79b-b9f4-4dc6-98a8-27868619103c)

![image](https://github.com/user-attachments/assets/f34c93e7-d43f-44cf-b5f5-fe010bd5caec)

![image](https://github.com/user-attachments/assets/c1137e68-db4b-4c8b-9483-0a5679d1cb47)

![image](https://github.com/user-attachments/assets/ac2a0f32-0903-43ef-9484-53774ab88b9e)

![image](https://github.com/user-attachments/assets/ff06a7cc-df54-409f-af68-bc2e1e24cb36)


## Result:
   Thus a Bayesian Network is generated using Python

