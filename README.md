# Chemical Reactor Process Machine Learning Project 

## Research Topic
The goal of this proposal was to determine if machine learning can be used to predict build up scenarios in a Tower Loop reactor based on process conditions and equipment setpoint.  By determining which features (variables) are most influential in reactor buildup scenarios, additional controls could be put in place to avoid the conditions that lead to production downtime. In addition to defining potential areas of concern it would be beneficial to provide the operators feedback on the highest areas of impact for changes they are planning to make on the system. 

## Background
Tower Loop Chemical reactors area a method to conduct gas liquid reactions because of the high mass transfer achieved in the gas-liquid interface.  It utilizes atomization, either through a high-speed wheel or via pressure nozzles, to quickly spray droplets as they pass through a hot medium.   Adjusting process conditions as mixing rate, residence time and temperatures to the required reaction rate, and removing product from the reaction zone will improve the yield and reduce the possibility of by-product formation or build of byproduct or product in the system.  It is actually a very complex process where the results can change with each product type being produced or the conditions of the reactor.  Small process changes can affect the characteristics of the product  In order to meet the required characteristics of individual formulas and at the same time maximize the reactor performance and capacity, the operating conditions such as airflows, temperatures, feed rates and atomization pressure are continuously monitored and controlled.   
While controlling the reactor and making setpoint changes to control the product characteristic and rates, a condition in the reactor can be created that causes the slurry to over agglomerate or become sticky.  This causes the typically free flowing product to be discharged as waste (not meeting specifications) or to build up in the system.  Both scenarios will cause product waste and if the buildup becomes too great it will cause extended downtime for the operation.  

## Implementation Strategy
The end objective of the machine learning project will be to define the features (variables) of the reactor that have the highest impact on the system product buildup and to predict if the reactor is in a state causing build up based on the current conditions.

## Data Collection:
The data was combined from 2 main sources, the operations control server database and the operator log files for each production run. 
The data from the server set is stored on private PI process servers.  Pi servers is a unique data storage system that captures timestamps and values for every PI point in the process.  A Pi point can be anything from valve position, flow rate, pump speed … etc.   The OSIsoft team refers to the database as a Microsoft SQL server in the traditional sense.  Only the relevant data points for the process were pulled into the dataset.  For this evaluation, information from over 135 PI points were pulled every 6 seconds during the time of actual batch production to allow evaluation.   This dataset contains information on over 1400 individual production runs.  This resulted in over 34 million pieces of data from around 237, 000 observations.    
The data in the original server dataset needed to be aligned prior to any assessments or combining with the operations logs.  The process data is created from multiple liquid feed trains that can be interchanged to multiple reactor units.  The data is collected on all points all the time, so alignment is necessary to determine the correct data is associated with the correct batch.    The image below (see Fig 1 below) shows an example of the complexity of the system and an example of the output alignment that was required is shown in red.   The alignment process reduced the number or columns from 135 to 26 features that are specific to the equipment used for the specific batches.  
  
Fig 1: Reactor Process Flow

The operator log files from the operation floor were in excel flat files.   The 1400 individual excel files were combined into a single DataFrame to allow processing.  These files contain some additional information not captured in the server information. The data logs include information about the nozzle parts, solids and comments from the operators that may indicate plugging.  This data is collected every hour as opposed to the 6min of the server data.  The data contained multiple data types, including numerical data, categorical data, time series data, and text data. Reviewing the data files indicates they are set up for easy viewing and not set up as database files. Formatting, hidden cells and merged cells made it more difficult to pull the information.  This data will then be merged with the production server data.


## Data Preprocessing and Analysis of Data
Quickly reviewing the data indicated that significant data preprocessing will be required.  Some of the issues quickly identified with the server data were lost data due to server issues, lack of feedback from the equipment (offline or broken) and downtime readings during batch.  In addition, there were issues with missing data from the log file information.   The data had to be converted into a usable numeric format to allow analysis and processing.   
The feature numerical data was reviewed for trends and outlier information.   A quick histogram of the features and analysis of the data showed outlier process information. Understanding of the process operations setpoints and limits allowed removal of the majority of the outliers (e.g. cannot have no flow and be producing product).  The size of the dataset allowed the removal of most of the outliers without impacting the model performance.  The pre and post cleaning of the data set is show below.   
 
Fig 1:  Feature Histogram Raw
 
Fig 2: Feature Histogram Cleaned
The histograms identified a potential issue with scales of the different features.  In order to remove the potential for unjust significance given to the larger scale features, the features scales were all standardized in the machine learning pipeline.   This makes sure that each feature is given the same importance in the evaluation.  
A quick review of the target information showed a class imbalance in the dataset.   This was expected as the majority of the time the reactors are running under good conditions.   Figure 4 below shows the extent of the imbalance.  The imbalance was addressed in the machine leaning pipeline with random oversampling.  Random oversampling was chosen over undersampling and smote in order to keep the majority of the data to train the model and not introduce random data (smote) that may not align with true operating conditions.  



 
Fig 4: Class Imbalance 

An analysis was conducted to determine if there were relationships between the target value and some of the categorical values.    Fig 5 below show the distribution of build-up cases against the different orifice sizes.  It shows that the build up cases are evenly distributed across all sets of orifices and is more dependent on the number of batches.  Fig 6 below shows the relationship of buildup and formulations.  It shows that the buildup is not consistent to any single formulation and that the number of assurances is also correlated to the number of times the formulation is run (i.e.  more occurrence on formulations that are produced more often).  
 
Fig 5:  Distribution per orifice size
 
Fig 6:  Distribution per Formula

Further review of the data was completed to establish any associations or tends in the data with respect to the target value (build-up).     From Fig 7, Operator controllable Variables Trend, and Fig 8, System or Feedback Variables Trends, there was no obvious indictor or combination of variables that would indicate build-up.  This can be seen from the random True (positive) indicators.  There is no significant grouping or trend area.  This led to the inclusion of all variables (features) in the initial model.   



 
Fig 7:  Operator controllable Variables Trend 

 
Fig 8:  System or Feedback Variables Trends

## Model selection and analysis review

The model was set up as a supervised classification model to determine if the conditions are conducive to buildup in the system.  This was chosen because the build-up conditions are known on past data and it was believed that precise conditions must occur to trigger the build-up.  Supervised learning models are better at precision with limited data and high feature numbers.   The final model chosen was a random forest classifier.  This provided the best performance metrics when evaluating using a combination of confusion matrix, precision, recall, f1 scores and visualized with ROC/AUC curve.  The random forest model was compared to multiple other models including (XGBoost, Decision Tree and KNN) and provided the best results.  The focus was on decision trees due to the fact that decision trees perform better at potential interactions between features. On reactors, multiple variables may have impact on other features. This was confirmed when KNN results provided the lowest scores.    
A pipeline was built to handle the numerical and categorical features.   Due to the multiple features with different units, multiple rows will have to be scaled to limit the adverse effect on the model. While some classifiers can handle the data without scaling there is some evidence that RFC is still impacted by scale.  Categorical encoding included one hot encoding to allow the inclusion of all variables.   
The Grid search and multiple trials were run on random forest model to determine the optimum parameters.   Parameters such as n_estimators, max_depth, min_sample_split, min_sample_leaves and criterion were all evaluated.  The parameters that had little impact on the results were removed in order to minimize computation time.   The model was geared to favor higher recall over precision in the preference. The impact of the false positive is lower than having a false negative. A false positive means slowing the system down unnecessarily while a false negative could lead to buildup and shutdown.  

Results: 
The model performed better than expected.  The accuracy was above 97.5%.  However this is not a good performance indicator due to the imbalanced dataset.   The f1 score was 87.9% and both the recall and precision were above 97%.  This indicates that the both the false negatives and false positive predictions are low as compared to the true positive.   This can be seen in the confusion matrix below Fig 9.  
 
 Fig 9:  Build Model Confusion Matrix


The ROC curve result of 0.97 indicates that the model accuracy is high.  
 
A review of the feature importance figure (Fig 11, showed that there was not one or two overriding important features.  It shows 20 features that have similar importance.  This was expected from the correlation in process data previously discussed.   All features were kept in the model to make it as accurate as possible since the target( build-up) indicator potential was so low in the imbalanced dataset.  Even the smallest change due to the less important features may push it over the edge into a true(build-up) scenario.   

 

Fig 11:   Feature importance










