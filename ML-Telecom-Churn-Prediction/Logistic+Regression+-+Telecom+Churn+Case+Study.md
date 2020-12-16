---
jupyter:
  jupytext:
    formats: ipynb,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.6.0
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

## Telecom Churn Case Study
With 21 predictor variables we need to predict whether a particular customer will switch to another telecom provider or not. In telecom terminology, this is referred to as churning and not churning, respectively.


### Step 1: Importing and Merging Data

```python
# Suppressing Warnings
import warnings
warnings.filterwarnings('ignore')
```

```python
# Importing Pandas and NumPy
import pandas as pd, numpy as np
```

```python
# Importing all datasets
churn_data = pd.read_csv("churn_data.csv")
churn_data.head()
```

```python
customer_data = pd.read_csv("customer_data.csv")
customer_data.head()
```

```python
internet_data = pd.read_csv("internet_data.csv")
internet_data.head()
```

#### Combining all data files into one consolidated dataframe

```python
# Merging on 'customerID'
df_1 = pd.merge(churn_data, customer_data, how='inner', on='customerID')
```

```python
# Final dataframe with all predictor variables
telecom = pd.merge(df_1, internet_data, how='inner', on='customerID')
```

### Step 2: Inspecting the Dataframe

```python
telecom.OnlineBackup.value_counts()
```

```python
# Let's see the head of our master dataset
telecom.head()
```

```python
# Let's check the dimensions of the dataframe
telecom.shape
```

```python
# let's look at the statistical aspects of the dataframe
telecom.describe()
```

```python
# Let's see the type of each column
telecom.info()
```

### Step 3: Data Preparation


#### Converting some binary variables (Yes/No) to 0/1

```python
# List of variables to map

varlist =  ['PhoneService', 'PaperlessBilling', 'Churn', 'Partner', 'Dependents']

# Defining the map function
def binary_map(x):
    return x.map({'Yes': 1, "No": 0})

# Applying the function to the housing list
telecom[varlist] = telecom[varlist].apply(binary_map)
```

```python
telecom.head()
```

#### For categorical variables with multiple levels, create dummy features (one-hot encoded)

```python
# Creating a dummy variable for some of the categorical variables and dropping the first one.
dummy1 = pd.get_dummies(telecom[['Contract', 'PaymentMethod', 'gender', 'InternetService']], drop_first=True)

# Adding the results to the master dataframe
telecom = pd.concat([telecom, dummy1], axis=1)
```

```python
telecom.head()
```

```python
# Creating dummy variables for the remaining categorical variables and dropping the level with big names.

# Creating dummy variables for the variable 'MultipleLines'
ml = pd.get_dummies(telecom['MultipleLines'], prefix='MultipleLines')
# Dropping MultipleLines_No phone service column
ml1 = ml.drop(['MultipleLines_No phone service'], 1)
#Adding the results to the master dataframe
telecom = pd.concat([telecom,ml1], axis=1)

# Creating dummy variables for the variable 'OnlineSecurity'.
os = pd.get_dummies(telecom['OnlineSecurity'], prefix='OnlineSecurity')
os1 = os.drop(['OnlineSecurity_No internet service'], 1)
# Adding the results to the master dataframe
telecom = pd.concat([telecom,os1], axis=1)

# Creating dummy variables for the variable 'OnlineBackup'.
ob = pd.get_dummies(telecom['OnlineBackup'], prefix='OnlineBackup')
ob1 = ob.drop(['OnlineBackup_No internet service'], 1)
# Adding the results to the master dataframe
telecom = pd.concat([telecom,ob1], axis=1)

# Creating dummy variables for the variable 'DeviceProtection'. 
dp = pd.get_dummies(telecom['DeviceProtection'], prefix='DeviceProtection')
dp1 = dp.drop(['DeviceProtection_No internet service'], 1)
# Adding the results to the master dataframe
telecom = pd.concat([telecom,dp1], axis=1)

# Creating dummy variables for the variable 'TechSupport'. 
ts = pd.get_dummies(telecom['TechSupport'], prefix='TechSupport')
ts1 = ts.drop(['TechSupport_No internet service'], 1)
# Adding the results to the master dataframe
telecom = pd.concat([telecom,ts1], axis=1)

# Creating dummy variables for the variable 'StreamingTV'.
st =pd.get_dummies(telecom['StreamingTV'], prefix='StreamingTV')
st1 = st.drop(['StreamingTV_No internet service'], 1)
# Adding the results to the master dataframe
telecom = pd.concat([telecom,st1], axis=1)

# Creating dummy variables for the variable 'StreamingMovies'. 
sm = pd.get_dummies(telecom['StreamingMovies'], prefix='StreamingMovies')
sm1 = sm.drop(['StreamingMovies_No internet service'], 1)
# Adding the results to the master dataframe
telecom = pd.concat([telecom,sm1], axis=1)
```

```python
telecom.head()
```

#### Dropping the repeated variables

```python
# We have created dummies for the below variables, so we can drop them
telecom = telecom.drop(['Contract','PaymentMethod','gender','MultipleLines','InternetService', 'OnlineSecurity', 'OnlineBackup', 'DeviceProtection',
       'TechSupport', 'StreamingTV', 'StreamingMovies'], 1)
```

```python
telecom = telecom.loc[~telecom.index.isin([488, 753, 936, 1082, 1340, 3331, 3826, 4380, 5218, 6670, 6754])]
```

```python
# telecom['TotalCharges'].sample(40)
# telecom['TotalCharges'].str.replace('.', '', 1).str.contains('\D',regex=True).sum()
# telecom[telecom['TotalCharges'].str.replace('.', '', 1).str.contains('\D',regex=True)].TotalCharges.index
```

```python
#The varaible was imported as a string we need to convert it to float
telecom['TotalCharges'] = telecom['TotalCharges'].str.strip().astype('float64')
```

```python
telecom.info()
```

Now you can see that you have all variables as numeric.


#### Checking for Outliers

```python
# Checking for outliers in the continuous variables
num_telecom = telecom[['tenure','MonthlyCharges','SeniorCitizen','TotalCharges']]
```

```python
# Checking outliers at 25%, 50%, 75%, 90%, 95% and 99%
num_telecom.describe(percentiles=[.25, .5, .75, .90, .95, .99])
```

From the distribution shown above, you can see that there no outliers in your data. The numbers are gradually increasing.


#### Checking for Missing Values and Inputing Them

```python
# Adding up the missing values (column-wise)
telecom.isnull().sum()
```

It means that 11/7043 = 0.001561834 i.e 0.1%, best is to remove these observations from the analysis

```python
# Checking the percentage of missing values
round(100*(telecom.isnull().sum()/len(telecom.index)), 2)
```

```python
# Removing NaN TotalCharges rows
telecom = telecom[~np.isnan(telecom['TotalCharges'])]
```

```python
# Checking percentage of missing values after removing the missing values
round(100*(telecom.isnull().sum()/len(telecom.index)), 2)
```

Now we don't have any missing values


### Step 4: Test-Train Split

```python
from sklearn.model_selection import train_test_split
```

```python
# Putting feature variable to X
X = telecom.drop(['Churn','customerID'], axis=1)

X.head()
```

```python
# Putting response variable to y
y = telecom['Churn']

y.head()
```

```python
# Splitting the data into train and test
X_train, X_test, y_train, y_test = train_test_split(X, y, train_size=0.7, test_size=0.3, random_state=100)
```

### Step 5: Feature Scaling

```python
from sklearn.preprocessing import StandardScaler
```

```python
scaler = StandardScaler()

X_train[['tenure','MonthlyCharges','TotalCharges']] = scaler.fit_transform(X_train[['tenure','MonthlyCharges','TotalCharges']])

X_train.head()
```

```python
### Checking the Churn Rate
churn = (sum(telecom['Churn'])/len(telecom['Churn'].index))*100
churn
```

We have almost 27% churn rate


### Step 6: Looking at Correlations

```python
# Importing matplotlib and seaborn
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline
```

```python
# Let's see the correlation matrix 
plt.figure(figsize = (20,10))        # Size of the figure
sns.heatmap(telecom.corr(),annot = True)
plt.show()
```

#### Dropping highly correlated dummy variables

```python
X_test = X_test.drop(['MultipleLines_No','OnlineSecurity_No','OnlineBackup_No','DeviceProtection_No','TechSupport_No',
                       'StreamingTV_No','StreamingMovies_No'], 1)
X_train = X_train.drop(['MultipleLines_No','OnlineSecurity_No','OnlineBackup_No','DeviceProtection_No','TechSupport_No',
                         'StreamingTV_No','StreamingMovies_No'], 1)
```

#### Checking the Correlation Matrix


After dropping highly correlated variables now let's check the correlation matrix again.

```python
plt.figure(figsize = (20,10))
sns.heatmap(X_train.corr(),annot = True)
plt.show()
```

### Step 7: Model Building
Let's start by splitting our data into a training set and a test set.


#### Running Your First Training Model

```python
import statsmodels.api as sm
```

```python
# Logistic regression model
logm1 = sm.GLM(y_train,(sm.add_constant(X_train)), family = sm.families.Binomial())
logm1.fit().summary()
```

### Step 8: Feature Selection Using RFE

```python
from sklearn.linear_model import LogisticRegression
logreg = LogisticRegression()
```

```python
from sklearn.feature_selection import RFE
rfe = RFE(logreg, 15)             # running RFE with 13 variables as output
rfe = rfe.fit(X_train, y_train)
```

```python
rfe.support_
```

```python
list(zip(X_train.columns, rfe.support_, rfe.ranking_))
```

```python
col = X_train.columns[rfe.support_]
```

```python
X_train.columns[~rfe.support_]
```

##### Assessing the model with StatsModels

```python
X_train_sm = sm.add_constant(X_train[col])
logm2 = sm.GLM(y_train,X_train_sm, family = sm.families.Binomial())
res = logm2.fit()
res.summary()
```

```python
# Getting the predicted values on the train set
y_train_pred = res.predict(X_train_sm)
y_train_pred[:10]
```

```python
y_train_pred = y_train_pred.values.reshape(-1)
y_train_pred[:10]
```

##### Creating a dataframe with the actual churn flag and the predicted probabilities

```python
y_train_pred_final = pd.DataFrame({'Churn':y_train.values, 'Churn_Prob':y_train_pred})
y_train_pred_final['CustID'] = y_train.index
y_train_pred_final.head()
```

##### Creating new column 'predicted' with 1 if Churn_Prob > 0.5 else 0

```python
y_train_pred_final['predicted'] = y_train_pred_final.Churn_Prob.map(lambda x: 1 if x > 0.5 else 0)

# Let's see the head
y_train_pred_final.head()
```

```python
from sklearn import metrics
```

```python
# Confusion matrix 
confusion = metrics.confusion_matrix(y_train_pred_final.Churn, y_train_pred_final.predicted )
print(confusion)
```

```python
# Predicted     not_churn    churn
# Actual
# not_churn        3270      365
# churn            579       708  
```

```python
# Let's check the overall accuracy.
print(metrics.accuracy_score(y_train_pred_final.Churn, y_train_pred_final.predicted))
```

#### Checking VIFs

```python
# Check for the VIF values of the feature variables. 
from statsmodels.stats.outliers_influence import variance_inflation_factor
```

```python
# Create a dataframe that will contain the names of all the feature variables and their respective VIFs
vif = pd.DataFrame()
vif['Features'] = X_train[col].columns
vif['VIF'] = [variance_inflation_factor(X_train[col].values, i) for i in range(X_train[col].shape[1])]
vif['VIF'] = round(vif['VIF'], 2)
vif = vif.sort_values(by = "VIF", ascending = False)
vif
```

There are a few variables with high VIF. It's best to drop these variables as they aren't helping much with prediction and unnecessarily making the model complex. The variable 'PhoneService' has the highest VIF. So let's start by dropping that.

```python
col = col.drop('PhoneService', 1)
col
```

```python
# Let's re-run the model using the selected variables
X_train_sm = sm.add_constant(X_train[col])
logm3 = sm.GLM(y_train,X_train_sm, family = sm.families.Binomial())
res = logm3.fit()
res.summary()
```

```python
y_train_pred = res.predict(X_train_sm).values.reshape(-1)
```

```python
y_train_pred[:10]
```

```python
y_train_pred_final['Churn_Prob'] = y_train_pred
```

```python
# Creating new column 'predicted' with 1 if Churn_Prob > 0.5 else 0
y_train_pred_final['predicted'] = y_train_pred_final.Churn_Prob.map(lambda x: 1 if x > 0.5 else 0)
y_train_pred_final.head()
```

```python
# Let's check the overall accuracy.
print(metrics.accuracy_score(y_train_pred_final.Churn, y_train_pred_final.predicted))
```

So overall the accuracy hasn't dropped much.


##### Let's check the VIFs again

```python
vif = pd.DataFrame()
vif['Features'] = X_train[col].columns
vif['VIF'] = [variance_inflation_factor(X_train[col].values, i) for i in range(X_train[col].shape[1])]
vif['VIF'] = round(vif['VIF'], 2)
vif = vif.sort_values(by = "VIF", ascending = False)
vif
```

```python
# Let's drop TotalCharges since it has a high VIF
col = col.drop('TotalCharges')
col
```

```python
# Let's re-run the model using the selected variables
X_train_sm = sm.add_constant(X_train[col])
logm4 = sm.GLM(y_train,X_train_sm, family = sm.families.Binomial())
res = logm4.fit()
res.summary()
```

```python
y_train_pred = res.predict(X_train_sm).values.reshape(-1)
```

```python
y_train_pred[:10]
```

```python
y_train_pred_final['Churn_Prob'] = y_train_pred
```

```python
# Creating new column 'predicted' with 1 if Churn_Prob > 0.5 else 0
y_train_pred_final['predicted'] = y_train_pred_final.Churn_Prob.map(lambda x: 1 if x > 0.5 else 0)
y_train_pred_final.head()
```

```python
# Let's check the overall accuracy.
print(metrics.accuracy_score(y_train_pred_final.Churn, y_train_pred_final.predicted))
```

The accuracy is still practically the same.


##### Let's now check the VIFs again

```python
vif = pd.DataFrame()
vif['Features'] = X_train[col].columns
vif['VIF'] = [variance_inflation_factor(X_train[col].values, i) for i in range(X_train[col].shape[1])]
vif['VIF'] = round(vif['VIF'], 2)
vif = vif.sort_values(by = "VIF", ascending = False)
vif
```

All variables have a good value of VIF. So we need not drop any more variables and we can proceed with making predictions using this model only

```python
# Let's take a look at the confusion matrix again 
confusion = metrics.confusion_matrix(y_train_pred_final.Churn, y_train_pred_final.predicted )
confusion
```

```python
# Actual/Predicted     not_churn    churn
        # not_churn        3269      366
        # churn            595       692  
```

```python
# Let's check the overall accuracy.
metrics.accuracy_score(y_train_pred_final.Churn, y_train_pred_final.predicted)
```

## Metrics beyond simply accuracy

```python
TP = confusion[1,1] # true positive 
TN = confusion[0,0] # true negatives
FP = confusion[0,1] # false positives
FN = confusion[1,0] # false negatives
```

```python
# Let's see the sensitivity of our logistic regression model
TP / float(TP+FN)
```

```python
# Let us calculate specificity
TN / float(TN+FP)
```

```python
# Calculate false postive rate - predicting churn when customer does not have churned
print(FP/ float(TN+FP))
```

```python
# positive predictive value 
print (TP / float(TP+FP))
```

```python
# Negative predictive value
print (TN / float(TN+ FN))
```

### Step 9: Plotting the ROC Curve


An ROC curve demonstrates several things:

- It shows the tradeoff between sensitivity and specificity (any increase in sensitivity will be accompanied by a decrease in specificity).
- The closer the curve follows the left-hand border and then the top border of the ROC space, the more accurate the test.
- The closer the curve comes to the 45-degree diagonal of the ROC space, the less accurate the test.

```python
def draw_roc( actual, probs ):
    fpr, tpr, thresholds = metrics.roc_curve( actual, probs,
                                              drop_intermediate = False )
    auc_score = metrics.roc_auc_score( actual, probs )
    plt.figure(figsize=(5, 5))
    plt.plot( fpr, tpr, label='ROC curve (area = %0.2f)' % auc_score )
    plt.plot([0, 1], [0, 1], 'k--')
    plt.xlim([0.0, 1.0])
    plt.ylim([0.0, 1.05])
    plt.xlabel('False Positive Rate or [1 - True Negative Rate]')
    plt.ylabel('True Positive Rate')
    plt.title('Receiver operating characteristic example')
    plt.legend(loc="lower right")
    plt.show()

    return None
```

```python
fpr, tpr, thresholds = metrics.roc_curve( y_train_pred_final.Churn, y_train_pred_final.Churn_Prob, drop_intermediate = False )
```

```python
list(zip(fpr,tpr,thresholds))
```

```python
draw_roc(y_train_pred_final.Churn, y_train_pred_final.Churn_Prob)
```

### Step 10: Finding Optimal Cutoff Point


Optimal cutoff probability is that prob where we get balanced sensitivity and specificity

```python
# Let's create columns with different probability cutoffs 
numbers = [float(x)/10 for x in range(10)]
for i in numbers:
    y_train_pred_final[i]= y_train_pred_final.Churn_Prob.map(lambda x: 1 if x > i else 0)
y_train_pred_final.head()
```

```python
# Now let's calculate accuracy sensitivity and specificity for various probability cutoffs.
cutoff_df = pd.DataFrame( columns = ['prob','accuracy','sensi','speci'])
from sklearn.metrics import confusion_matrix

# TP = confusion[1,1] # true positive 
# TN = confusion[0,0] # true negatives
# FP = confusion[0,1] # false positives
# FN = confusion[1,0] # false negatives

num = [0.0,0.1,0.2,0.3,0.4,0.5,0.6,0.7,0.8,0.9]
for i in num:
    cm1 = metrics.confusion_matrix(y_train_pred_final.Churn, y_train_pred_final[i] )
    total1=sum(sum(cm1))
    accuracy = (cm1[0,0]+cm1[1,1])/total1
    
    speci = cm1[0,0]/(cm1[0,0]+cm1[0,1])
    sensi = cm1[1,1]/(cm1[1,0]+cm1[1,1])
    cutoff_df.loc[i] =[ i ,accuracy,sensi,speci]
print(cutoff_df)
```

```python
# Let's plot accuracy sensitivity and specificity for various probabilities.
cutoff_df.plot.line(x='prob', y=['accuracy','sensi','speci'])
plt.show()
```

#### From the curve above, 0.3 is the optimum point to take it as a cutoff probability.

```python
y_train_pred_final['final_predicted'] = y_train_pred_final.Churn_Prob.map( lambda x: 1 if x > 0.3 else 0)

y_train_pred_final.head()
```

```python
# Let's check the overall accuracy.
metrics.accuracy_score(y_train_pred_final.Churn, y_train_pred_final.final_predicted)
```

```python
confusion2 = metrics.confusion_matrix(y_train_pred_final.Churn, y_train_pred_final.final_predicted )
confusion2
```

```python
TP = confusion2[1,1] # true positive 
TN = confusion2[0,0] # true negatives
FP = confusion2[0,1] # false positives
FN = confusion2[1,0] # false negatives
```

```python
# Let's see the sensitivity of our logistic regression model
TP / float(TP+FN)
```

```python
# Let us calculate specificity
TN / float(TN+FP)
```

```python
# Calculate false postive rate - predicting churn when customer does not have churned
print(FP/ float(TN+FP))
```

```python
# Positive predictive value 
print (TP / float(TP+FP))
```

```python
# Negative predictive value
print (TN / float(TN+ FN))
```

 


 


 


 


## Precision and Recall

```python
#Looking at the confusion matrix again
```

```python
confusion = metrics.confusion_matrix(y_train_pred_final.Churn, y_train_pred_final.predicted )
confusion
```

##### Precision
TP / TP + FP

```python
confusion[1,1]/(confusion[0,1]+confusion[1,1])
```

##### Recall
TP / TP + FN

```python
confusion[1,1]/(confusion[1,0]+confusion[1,1])
```

Using sklearn utilities for the same

```python
from sklearn.metrics import precision_score, recall_score
```

```python
?precision_score
```

```python
precision_score(y_train_pred_final.Churn, y_train_pred_final.predicted)
```

```python
recall_score(y_train_pred_final.Churn, y_train_pred_final.predicted)
```

### Precision and recall tradeoff

```python
from sklearn.metrics import precision_recall_curve
```

```python
y_train_pred_final.Churn, y_train_pred_final.predicted
```

```python
p, r, thresholds = precision_recall_curve(y_train_pred_final.Churn, y_train_pred_final.Churn_Prob)
```

```python
plt.plot(thresholds, p[:-1], "g-")
plt.plot(thresholds, r[:-1], "r-")
plt.show()
```

### Step 11: Making predictions on the test set

```python
X_test[['tenure','MonthlyCharges','TotalCharges']] = scaler.transform(X_test[['tenure','MonthlyCharges','TotalCharges']])
```

```python
X_test = X_test[col]
X_test.head()
```

```python
X_test_sm = sm.add_constant(X_test)
```

Making predictions on the test set

```python
y_test_pred = res.predict(X_test_sm)
```

```python
y_test_pred[:10]
```

```python
# Converting y_pred to a dataframe which is an array
y_pred_1 = pd.DataFrame(y_test_pred)
```

```python
# Let's see the head
y_pred_1.head()
```

```python
# Converting y_test to dataframe
y_test_df = pd.DataFrame(y_test)
```

```python
# Putting CustID to index
y_test_df['CustID'] = y_test_df.index
```

```python
# Removing index for both dataframes to append them side by side 
y_pred_1.reset_index(drop=True, inplace=True)
y_test_df.reset_index(drop=True, inplace=True)
```

```python
# Appending y_test_df and y_pred_1
y_pred_final = pd.concat([y_test_df, y_pred_1],axis=1)
```

```python
y_pred_final.head()
```

```python
# Renaming the column 
y_pred_final= y_pred_final.rename(columns={ 0 : 'Churn_Prob'})
```

```python
# Rearranging the columns
# y_pred_final = y_pred_final.reindex_axis(['CustID','Churn','Churn_Prob'], axis=1)
```

```python
# Let's see the head of y_pred_final
y_pred_final.head()
```

```python
y_pred_final['final_predicted'] = y_pred_final.Churn_Prob.map(lambda x: 1 if x > 0.42 else 0)
```

```python
y_pred_final.head()
```

```python
# Let's check the overall accuracy.
metrics.accuracy_score(y_pred_final.Churn, y_pred_final.final_predicted)
```

```python
confusion2 = metrics.confusion_matrix(y_pred_final.Churn, y_pred_final.final_predicted )
confusion2
```

```python
TP = confusion2[1,1] # true positive 
TN = confusion2[0,0] # true negatives
FP = confusion2[0,1] # false positives
FN = confusion2[1,0] # false negatives
```

```python
# Let's see the sensitivity of our logistic regression model
TP / float(TP+FN)
```

```python
# Let us calculate specificity
TN / float(TN+FP)
```

```python
1050/1400
```

```python
1190/1400
```

```python
1050/1450
```

```python
precision - 4/6
sensitivity recall - 4/5
accuracy - 7/10
specificity - 3/5

```