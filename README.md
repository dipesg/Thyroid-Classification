# Thyroid-Classification 🇳🇵
- In this project we have develop a classification methodology to predict the type of thyroid based on the given training data.

## Table of Content
  * [Demo](#demo)
  * [Overview](#overview)
  * [Architechture](#architechture)
  * [Motivation](#motivation)
  * [Technical Aspect](#technical-aspect)
  * [Installation](#installation)
  * [Run](#run)
  * [Deployment on Heroku](#deployment-on-heroku)
  * [Directory Tree](#directory-tree)
  * [Technologies Used](#technologies-used)
  * [Credits](#credits)

## Demo
Link: [https://thyroiddetection1234.herokuapp.com/](https://thyroiddetection1234.herokuapp.com/)


## Overview
This is a Thyroid-Classification Flask app trained on **RandomForest and KNN**. Here in this app data is given by the user in the form of **csv format.** 
- Code is organized in a **Modular Format.**
- First user input data is stored in a **Sqlite database** where given csv file is separeted as a good file or a bad file according to specified **Schema file** thus separeted good file is given for a model training. 
- After preprocessing the data we select sample by the means of **Clustering** technique and perform a  hyperparameter tuning then we train a model according to that cluster and select a best model according to their AUC score.


## Architechture
![git_read](https://user-images.githubusercontent.com/75604769/154045380-3ee6d67e-4eaf-4dc1-9227-5387bf3693f8.jpg)

## Motivation
My mother recently suffered from hypothyroid and waiting for result is tedius and time consuming task so its a critical issue to get a result fast. So this is the motivation for this project.

## Technical aspect
#### Data preprocessing and Model Training
  - **Dataset**
      - The client will send data in multiple sets of files in batches at a given location. Data will contain different classes of thyroid and 30 columns of different values.
"Class" column will have four unique values “negative, compensated_hypothyroid,
primary_hypothyroid, secondary_hypothyroid”.

      - Apart from training files, we also require a "schema" file from the client, which contains all the relevant information about the training files such as:
Name of the files, Length of Date value in FileName, Length of Time value in FileName, Number of Columns, Name of the Columns, and their datatype.

  - **Data Validation**
      - In this step, we perform different sets of validation on the given set of training files.  
      - Name Validation- We validate the name of the files based on the given name in the schema file. We have created a regex pattern as per the name given in the schema file to use for validation. After validating the pattern in the name, we check for the length of date in the file name as well as the length of time in the file name. If all the values are as per requirement, we move such files to "Good_Data_Folder" else we move such files to "Bad_Data_Folder."

      - Number of Columns - We validate the number of columns present in the files, and if it doesn't match with the value given in the schema file, then the file is moved to "Bad_Data_Folder."


      - Name of Columns - The name of the columns is validated and should be the same as given in the schema file. If not, then the file is moved to "Bad_Data_Folder".

      - The datatype of columns - The datatype of columns is given in the schema file. This is validated when we insert the files into Database. If the datatype is wrong, then the file is moved to "Bad_Data_Folder".


      - Null values in columns - If any of the columns in a file have all the values as NULL or missing, we discard such a file and move it to "Bad_Data_Folder".

 - **Data Insertion in Database**
    - **Steps**
      - Database Creation and connection - Create a database with the given name passed. If the database is already created, open the connection to the database. 
      - Table creation in the database - Table with name - "Good_Data", is created in the database for inserting the files in the "Good_Data_Folder" based on given column names and datatype in the schema file. If the table is already present, then the new table is not created and new files are inserted in the already present table as we want training to be done on new as well as old training files.     
      - Insertion of files in the table - All the files in the "Good_Data_Folder" are inserted in the above-created table. If any file has invalid data type in any of the columns, the file is not loaded in the table and is moved to "Bad_Data_Folder".

 - **Model Training**
    - **Steps**
      - Data Export from Db - The data in a stored database is exported as a CSV file to be used for model training.
      - Data Preprocessing   
       - Drop columns not useful for training the model. Such columns were selected while doing the EDA.
       - Replace the invalid values with numpy “nan” so we can use imputer on such values.
       - Encode the categorical values
       - Check for null values in the columns. If present, impute the null values using the KNN imputer.
       - After imputing, handle the imbalanced dataset by using RandomOverSampler.
       - Clustering - KMeans algorithm is used to create clusters in the preprocessed data. The optimum number of clusters is selected by plotting the elbow plot, and for the dynamic selection of the number of clusters, we are using "KneeLocator" function. The idea behind clustering is to implement different algorithms To train data in different clusters. The Kmeans model is trained over preprocessed data and the model is saved for further use in prediction.
      - Model Selection - After clusters are created, we find the best model for each cluster. We are using two algorithms, "Random Forest" and "KNN". For each cluster, both the algorithms are passed with the best parameters derived from GridSearch. We calculate the AUC scores for both models and select the model with the best score. Similarly, the model is selected for each cluster. All the models for every cluster are saved for use in prediction. 


- **Prediction**
- **Steps**
  - Data Export from Db - The data in the stored database is exported as a CSV file to be used for prediction.
  - Data Preprocessing   
      - Drop columns not useful for training the model. Such columns were selected while doing the EDA.
      - Replace the invalid values with numpy “nan” so we can use imputer on such values.
      - Encode the categorical values
      - Check for null values in the columns. If present, impute the null values using the KNN imputer.
  - Clustering - KMeans model created during training is loaded, and clusters for the preprocessed prediction data is predicted.
  - Prediction - Based on the cluster number, the respective model is loaded and is used to predict the data for that cluster.
  - Once the prediction is made for all the clusters, the predictions along with the original names before label encoder are saved in a CSV file at a given location and the location is returned to the client.


## Installation
The Code is written in Python 3.7. If you don't have Python installed you can find it [here](https://www.python.org/downloads/). If you are using a lower version of Python you can upgrade using the pip package, ensuring you have the latest version of pip. To install the required packages and libraries, run this command in the project directory after [cloning](https://www.howtogeek.com/451360/how-to-clone-a-github-repository/) the repository:
```bash
pip install -r requirements.txt
```

## Run
> **Step 1: Clone the repository.**
```bash
git clone https://github.com/dipesg/Thyroid-Detection.git
```
> **Step 2: Create virtual Environment, either by using conda, pyenv or venv.**
```bash
conda create -n fraud python=3.7 -y
```
> **Step 3: Install dependensies.**
```bash
pip install -r requirements.txt
```
>**Step 4: Run app from postman.**
 - **1.Install postman.**
 - Link for detail description and guide: [Postman Guide](https://www.guru99.com/postman-tutorial.html)


## Deployment on Heroku
#### **Step 1: Create Required file**
- **Create requirements.txt**
```bash
pip freeze > requirements.txt
```
- **Create Procfile and include:**
```bash
web: gunicorn main:app
```
- **Create runtime .txt and include:**
```bash
python-3.7.12
```

#### **Step 2**
- **Go to Heroku.com and create an account and login.**
- **Click on new(inside the red box) to create a new app.**

![dipu_github2](https://user-images.githubusercontent.com/75604769/156884168-cebe9ce5-591c-4ea6-a9c6-e378f8c1f0c7.png)

- **Give the name of the app and click 'create app'.**
- **For pushing it on heroku follow [[Reference](https://devcenter.heroku.com/articles/config-vars)]**

## Directory Tree
![dir1](https://user-images.githubusercontent.com/75604769/156930494-393f2afb-fa9a-4c88-b5dc-f3a08a9aaa2f.png)

![thy2](https://user-images.githubusercontent.com/75604769/156930563-ae069eb1-cc22-46ad-a4a9-e8944f4794e1.png)

![thy3](https://user-images.githubusercontent.com/75604769/156930574-6d1b75c6-4731-4213-bd16-c29665ea8eee.png)

![thy4](https://user-images.githubusercontent.com/75604769/156930599-dbb0e5fc-44b7-4c3d-8d81-cf55ebef6d6a.png)

![thy5](https://user-images.githubusercontent.com/75604769/156930648-3113dfe2-dca1-44a9-a34d-e08c74daf5af.png)


## Bug / Feature Request
If you find a bug (the website couldn't handle the query and / or gave undesired results), kindly open an issue [here](https://github.com/dipesg/Insurance_Fraud/issues/new) by including your search query and the expected result.

If you'd like to request a new function, feel free to do so by opening an issue [here](https://github.com/dipesg/Insurance_Fraud/issues/new). Please include sample queries and their corresponding results.

## Technologies Used
![](https://forthebadge.com/images/badges/made-with-python.svg)

[<img target="_blank" src="https://scikit-learn.org/stable/_static/scikit-learn-logo-small.png" width=200>](https://scikit-learn.org/stable/) [<img target="_blank" src="https://flask.palletsprojects.com/en/1.1.x/_images/flask-logo.png" width=170>](https://flask.palletsprojects.com/en/1.1.x/) [<img target="_blank" src="https://number1.co.za/wp-content/uploads/2017/10/gunicorn_logo-300x85.png" width=280>](https://gunicorn.org) [<img target="_blank" src="https://i0.wp.com/learn.onemonth.com/wp-content/uploads/2019/07/image2-1.png?w=600&ssl=1" width=200>](https://learn.onemonth.com/the-best-way-to-learn-sql/) 

[<img target="_blank" src="https://numpy.org/images/logo.svg" width=270>](https://numpy.org/)  [<img target="_blank" src="https://pandas.pydata.org/static/img/pandas.svg" width=270>](https://pandas.pydata.org/about/citing.html)

## Credits
- [Ineuron](https://ineuron.ai/) - This project wouldn't have been possible without this website. It saved my enormous amount of time . A huge shout-out to its creator [krish Naik](https://github.com/krishnaik06) and [Sudhanshu Sir]().
