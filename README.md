DISCLAIMER: This application is used for demonstrative and illustrative purposes only and does not constitute an offering that has gone through regulatory review.

#  Create and deploy a scoring model to predict heart failure on IBM Cloud with Watson Studio

In this Code Pattern, we will use the Automatic Model Builder on Watson Studio to build a predictive model that demonstrates a potential health care use case.

When the reader has completed this Code Pattern, they will understand how to:

* Build a predictive model using the Watson Automatic Model Builder
* Deploy the model to IBM's Watson Machine Learning service
* Access the Machine Learning model via either APIs or a Node.js app

![](doc/source/images/architecture.png)

## Flow
1. The developer creates a Watson Studio Service
2. Use the Automatic Model Builder to build, train, and evaluate a model; part of the Watson Machine Learning Service
3. Watson Studio uses Cloud Object storage to manage your data and project files 
4. Watson Machine Learning Service depends on an Apache Spark service for model training
5. Import data on heart failure
6. Trained models are deployed into production using Watson Machine Learning Service
7. A Node.js web app is deployed on IBM Cloud calling the predictive model hosted in the Watson Machine Learning Service
8. A user visits the web app, enters their information, and the predictive model returns a response

## Included components
* [Watson Studio](https://www.ibm.com/cloud/watson-studio): Analyze data using RStudio, Jupyter, and Python in a configured, collaborative environment that includes IBM value-adds, such as managed Spark.

## Featured technologies
* [Artificial Intelligence](https://medium.com/ibm-data-science-experience): Artificial intelligence can be applied to disparate solution spaces to deliver disruptive technologies.
* [Data Science](https://medium.com/ibm-data-science-experience/): Systems and scientific methods to analyze structured and unstructured data in order to extract knowledge and insights.
* [Node.js](https://nodejs.org/): An open-source JavaScript run-time environment for executing server-side JavaScript code.


# Steps
1. [Create an instance of the Watson Studio Service](#1-create-an-instance-of-the-watson-studio-service)
2. [Welcome to Watson Studio](#2-welcome-to-watson-studio)
3. [Create a project in Watson Studio and upload training data](#3-create-a-project-in-watson-studio-and-upload-training-data)
4. [Create an instance of the Watson Machine Learning Service and associate it to project](#4-create-an-instance-of-the-watson-machine-learning-service-and-associate-it-to-project)
5. [Train a Machine Learning model](#5-train-a-machine-learning-model)



5. [Explore the data](#5-upload-and-explore-data)

5. [Save the credentials for your Watson Machine Learning Service](#5-save-the-credentials-for-your-watson-machine-learning-service)
8. [Deploy the saved predictive model as a scoring service](#8-deploy-the-saved-predictive-model-as-a-scoring-service)
9. [Deploy the testing application](#9-deploy-the-testing-application)


## Prerequisites

* An [IBM Cloud Account](https://console.bluemix.net)

* A space in IBM Cloud US South or United Kingdom regions.

As of 2/5/2018, the Machine Learning service on IBM Cloud is only available in the US South or United Kingdom regions.

## 1. Create an instance of the Watson Studio Service
Watson Studio is your IDE for Machine Learning and Data Science, combining opensource tools, and libraries into a unified Cloud based platform for discovering and sharing insights. For this lab we're using the Automatic Model Builder simplifying the data preparation, training, and evaluation steps of machine learning. 

1. In your browser go to the [IBM Cloud Dashboard](https://console.bluemix.net/dashboard/apps) and click `Catalog`. 

2. In the navigation menu at the left, select `AI` and then select `Watson Studio`.

  ![](doc/source/images/watson-studio-service.png?raw=true)

3. Verify this service is being created in the `Dallas region`, and you've selected the `lite/free` pricing plan.

Note the `lite/free` plan only allows you to add a single user to your project, and is limited in the compute capacity hours.  More details on limits and how to monitor usage is available in the [documentation](https://dataplatform.cloud.ibm.com/docs/content/admin/monitor-resources.html?context=analytics&linkInPage=true).

  ![](doc/source/images/watson-studio-create.png?raw=true)

4. Click `Create`

5. Launch your newly created Watson Studio Environment by clicking `Get Started`

  ![](doc/source/images/launch-watson-studio.png?raw=true)
 
 
  
## 2. Welcome to Watson Studio

IBM Watson Studio is a collaborative environment with AI tools that you and your team can use to collect and prepare training data, and to design, train, and deploy machine learning models.

Ranging from graphical tools you can use to build a model in minutes, to tools that automate running thousands of experiment training runs and hyperparameter optimization, Watson Studio AI tools support popular frameworks, including: TensorFlow, Caffe, PyTorch, and Keras.

You can think of Watson Studio AI tools in four categories:

* Visual recognition
* Natural language classification
* Machine learning
* Deep learning

Documentation is available [here](https://dataplatform.cloud.ibm.com/docs/content/getting-started/welcome-main.html?context=analytics)

#### Overview Landing Page

  ![](doc/source/images/watson-studio-overview.png?raw=true)
  
   1. **Projects** - Organize resources used when working with data; here you see your most recently updated projects
   2. **Toos** - Quick links to commonly used Data Science and ML Tools including RStudio, Data Refinery, Jupyter Notebooks, or a Visual Neturl Network Model Builder
   3. **Catalog** - Create and manage data policies for managed or connected data resources
   4. **Community** - Links to the best content found by IBM Data Scientists, including example notebooks, datasets, and tutorials
   5. **Services** - Create Watson, data, and compute services and connections. Such as Watson Visual Recognition, or Apache Spark 
   6. **Manage** - Account wide configuration, including Anaconda environments, security, catalogs, billing
   7. **Hamburger Menu** - Access to IBM Cloud dashboard and tools
   8. **IBM Studio Menu** - Quick link to the Watson Studio welcome page
   9. **Account Profile and Settings** - Personal account settings

#### Overview Project Page

  ![](doc/source/images/watson-studio-project-overview.png?raw=true)
  
  1. **Overview** - The page you're seeing now, shows who is collaborating on the projects, and number of assets associated
  2. **Assets** - Links to each asset found within the project, broken down into categories
  3. **Environments** - Track you capacity units used, and manage Anaconda environments. 
  4. **Access Control** - Manage collaborators for project
  5. **Readme** - Markdown documentation for projecct
  6. **Add to Project** - Create, connect, or import new assets to project

## 3. Create a project in Watson Studio and upload training data
A project is how you organize your resources to achieve a particular goal. Your project resources can include data, collaborators, and analytic assets like notebooks and models. Projects depends on a connection to object storage to store assets. Each project has a separate bucket to hold the project's assets. 


1. From the Watson Studio dashboard getting started display, click on `Create Project`, or `New Project` 

  ![](doc/source/images/new-project.png?raw=true)

2. Select project type. There are many different tools built into Watson Studio and multiple views are built to simplify the features shown to users.  Select the `Standard Project` where all features are available, and click `Create Project`.

  ![](doc/source/images/standard-project.png?raw=true)
  
3. Watson Studio projects depend on Object Storage for storing project assets such as notebooks, models, and data. These project assets are created in a project specific bucket within object storage.  If you don't already have Object Storage defined you can create a new instance of the service directly from the New Project dialog. Under `Define storage` select `Add`. In the Cloud Object Storage service creation menu, accept the default options, select `Lite` and then `Create`.  

**Note:**  You cannot define two Object Storage services under the free tier of IBM Cloud; if you already have object storage defined, choose `Existing` as highlighted in the screenshot below.

  ![](doc/source/images/create-services.png?raw=true)
  
  
  ![](doc/source/images/create-services-cos.png?raw=true)
  
  
4. With object storage created, or existing object storage linked, click `Refresh` allowing Watson Studio to discover the newly created service. Enter _Watson ML Demo_ as the project name and click `Create`.

5. From within the new project's `Overview` panel, click `Add to project` on the top right, selecting `Data asset`.

  ![](doc/source/images/add-to-project.png?raw=true)

  A panel on the right of the screen appears, select `Load` and click on `Browse` to upload the data file you'll use to create a predictive model.

  ![](doc/source/images/add-data-asset.png?raw=true)

6. On your machine, browse to the location of the file [**patientdataV6.csv**](https://raw.githubusercontent.com/justinmccoy/predictive-model-on-watson-ml/master/data/patientdataV6.csv) in this repository in the **data/** directory. Select the file and click on Open (or the equivalent action for your operating system).

Once successfully uploaded, the file should appear in the `Data Assets` section of `Assets`.

  ![](doc/source/images/data-assets.png?raw=true)

Congratulations, you've created a new Machine Learning Project and uploaded training data.

## 4. Create an instance of the Watson Machine Learning Service and associate it to project

Machine Learning is a service on IBM Cloud with features for training and deploying machine learning models and neural networks:

* Interfaces for building, training, and deploying models: [Python client library](https://wml-api-pyclient.mybluemix.net/), [Command line interface](https://dataplatform.cloud.ibm.com/docs/content/analyze-data/ml_dlaas_environment.html), [REST API](https://watson-ml-api.mybluemix.net/)
* Deployment infrastructure for hosting your trained models
* Hyperparameter optimization for training complex neural networks
* Distributed deep learning for distributing training runs across multiple servers
* GPUs for faster training

In this lab we're using the Automatic Model Builder, a feature of the Watson Machine Learning Service. The Automatic Model Builder guides you, step by step, through building a model that uses popular machine learning algorithms. Just upload your training data, and then let the model builder automatically prepare your data and recommend techniques that suit your data.

1. Click on `Settings` for the project, then `Add Service` under `Associate Services` and finally, select `Watson` to add a Watson service to the project.  Note there are several types of servcies we can assoicate with a project, including Dashboards, and connections to existing Apache Spark services.

  ![](doc/source/images/settings.png?raw=true)

2. Select `Machine Learning` from the list of available Watson Services.

  ![](doc/source/images/add-associated-service.png?raw=true)

**Note:** If you have an existing Machine Learning service select your `Existing` service instead of creating a new one.

  ![](doc/source/images/choose-ml-service.png?raw=true)
  
3. Verify this service is being created in the `Dallas region`.

  ![](doc/source/images/watson-ml-create.png?raw=true)

4. Click `Create`.

The Watson Machine Learning service is now listed as one of your `Associated Services`. 


## 5. Train a Machine Learning model
The Automatic Model Builder tool in Watson Studio, backed by the Watson Machine Learning Service simplifies two fundamental operations of machine learning: training and scoring.

*Training* is the process of refining an algorithm so that it can learn from a data set. The output of this operation is called a model. A model encompasses the learned coefficients of mathematical expressions.

*Scoring* is the operation of predicting an outcome by using a trained model. The output of the scoring operation is another data set containing predicted values.

Let's create a machine learning pipeline that leverages data transformations and machine learning algorithms to train several models and evaluate their accuracy; all without writing any code.


1. Starting from the `Assets` tab of our Watson Studio Project *Watson ML Demo*, select `Add to Project` and `Model`

  ![](doc/source/images/add-model-to-project.png?raw=true)
  
2. Within the *New Model* dialog, name the model *Heart Failure Prediction Model*, select the runtime that will be used for building a data pipeline, and training.  The `Default Spark Scala 2.11` environment should be used, and will consume 1.5 capacity units per hour of training. 

Select `Manual` to define the evaluator algorithms, they type of model to train, and how to split training and validation data.

  ![](doc/source/images/create-model.png?raw=true)

**Note:** Alternate environments or Apache Spark runtimes can be used for training.
  
3. Click `Create`, to begin training and evaluating.

4. Select your training dataset and click `Next`

  ![](doc/source/images/select-training-data.png?raw=true)

5. Configure how the Machine Learning Model is trained, and what data is used in the training. Before moving onto the next step there are several basic machine learning terms, best practices, and conventions to understand. First it's important to understand our data, what it represents, it's format, and what we're trying to predict.

  ![](doc/source/images/configure-amb.png?raw=true)
  
  
  1. **Selecting a Label Column** - The Label Column is what we would like to predict. In this usecase we are trying to predict if someone is at risk of heartrate failure or not. Looking at the data from previous steps we know there's a *HEARTRATEFAILURE* column and it's represented by a string value of *Y* or *N* for each sample.  Select `HEARTRATEFAILURE`, as it's what we're trying to predict.
  2. **Selecting Feature Columns** - The Feature Column(s), are what fields in each sample are used to make a prediction. From previous steps we identifyed several columns in our dataset that represent different *Features* of each sample that might influence if a patient is at risk of heart rate failure.  Here the Automatic Model Building tool will default to selecting all columns, excluding the Label Column of *HEARTRATEFAILURE* to use in making a prediction.
  3. **Selecting Model Type** - The Automatic Model Builder simplifies *Classification* and *Regression* tasks, where *classification* builds a model to predict a discreate class, and *Regression* builds a model to predict a continious value. Since we're trying to predict either *Y* someone is at risk of heartrate failure or *N* someone is not at risk of heartrate failure we're working on a `Binary Classification` task, where the prediction can either be 0 or 1. Based on the `Label Column` selected the Automatic Model Builder will have already selected `Binary Classification`.
  4. **Validation Split** - 
  5.
  

## 6. Score the newly trained, and deployed model

Although training is a critical step in the machine learning process, Watson Machine Learning enables you to streamline the functioning of your models by deploying them and getting actual business value from them over time and through all of their iterations.


## 5. Explore the Data


### 5. Save the credentials for your Watson Machine Learning Service

* In a different browser tab go to [http://console.bluemix.net](http://console.bluemix.net) and log in to the Dashboard.

* Click on your Watson Machine Learning instance under `Services`.

  ![](doc/source/images/services-overview.png?raw=true)

* Click on `Service credentials` and then on `View credentials` to see the credentials.

  ![](doc/source/images/ml-credentials.png?raw=true)

* Save the username, password and instance_id to a text file on your machine. You’ll need this information later in your Jupyter notebook.

### 6. Upload and explore data
stuff


### 7. Train a Machine Learning model

* If you don't have your newly created Project open, first click `Projects` -> `View All Projects`, and then select your newly created project from Step 4. Next, in the Data Science Experience browser tab click on `Overview` and then click `add notebooks`.

  ![](doc/source/images/add-notebook.png?raw=true)

* Click on `From URL` and name the notebook _Apache Spark integration with Watson ML_.

* Under `Notebook URL` provide the following url: https://github.com/IBM/predictive-model-on-watson-ml/blob/master/demo1.ipynb

  ![](doc/source/images/create-notebook.png?raw=true)

* Click `Create Notebook` to create the new notebook.

### 7. Run the notebook in IBM Data Science Experience

* Place your cursor in the first code block in the notebook.

  ![Insert Spark Data Frame Step 1](doc/source/images/Picture23.png)

* Click on the `Find and Add` data icon -- see step 1 in diagram below -- and then select `Insert to code` under the file **patientdataV6.csv**. This is step 2 in diagram below. Finally select `Insert Spark Data Frame` -- which is step 3 in diagram below.

  ![](doc/source/images/insert-credentials2.png?raw=true)

> Note:  Make sure to rename the variable to `df_data` and add `.option('inferSchema','True')\`.

  ![Insert Spark Data Frame Step 3](doc/source/images/Picture25.png)

* Click on the `Run` icon to run the code in the cell.

  ![](doc/source/images/run-notebook.png?raw=true)

* Move your cursor to each code cell and run the code in it. Read the comments for each cell to understand what the code is doing. **Important** when the code in a cell is still running, the label to the left changes to **In [\*]**:.
  Do **not** continue to the next cell until the code is finished running.

* When you get to the cell that says `Stop here !!!!` insert the username and password that you saved from your Watson Machine Learning instance into the code before running it.

  ![Run Notebook](doc/source/images/Picture27.png)

* Continue running each cell until you finish the entire notebook.

### 8. Deploy the saved predictive model as a scoring service

* In a different browser tab go to [http://console.bluemix.net](http://console.bluemix.net) and log in to your dashboard.

* Click on the entry for your Watson Machine Learning service under `Services`.

* Click `Manage` and then click on the `Launch Dashboard` button in the Watson Machine Learning tile.

![](doc/source/images/launch-dashboard.png?raw=true)

* Your saved model should appear. Under `Actions` select `Create Deployment`.

![](doc/source/images/create-deployment1.png?raw=true)

* Name the deployment _Heart Failure Prediction Model Deployment_ and click `Save`. Keep the type of deployment set to the default value 'Online'.

![](doc/source/images/create-deployment2.png?raw=true)

* Your model should now be deployed and visible as a Deployment with status 'ACTIVE'.

![](doc/source/images/create-deployment3.png?raw=true)

* Restart the Node.js Web App. For this, return to your IBM Cloud Dashboard and select the restart icon to restart the web application.

![](doc/source/images/restart-app.png?raw=true)


### 1. Deploy the testing application

Use Ctrl-click on the Deploy to `IBM Cloud` button below to open the deployment process in a separate tab.

  [![Deploy to IBM Cloud](https://bluemix.net/deploy/button.png)](https://bluemix.net/deploy?repository=https://github.com/IBM/predictive-model-on-watson-ml)

> Note:  Make sure to deploy the application to the same region and space as where the *Watson Studio*, *Apache Spark* and *Cloud Object Storage* services will be created. Please take note of this space as later in this lab the Watson Machine Learning service needs to be deployed into the same space.

* Click on `Deploy` to deploy the application.

  ![](doc/source/images/pipeline.png?raw=true
)

* A Toolchain and Delivery Pipeline will be created for you to pull the app out of Github and deploy it in to IBM Cloud. Click on the Delivery Pipeline tile to see the status of the deployment.

  ![](doc/source/images/toolchain.png?raw=true)

* Wait for the **Deploy Stage** to complete successfully.

  ![](doc/source/images/deploy-stage.png?raw=true)


# Sample Output

* Click on the application URL to open the application in a separate tab.

![](doc/source/images/open-app.png?raw=true)

* When the application appears click on `Score now` to test the scoring model with the default values.

* Verify that the model predicts that there is a risk of heart failure for the patient with these medical characteristics.

![](doc/source/images/failure-yes.png?raw=true)

* `Click Close`. Run the app again with the following parameters.

![Score](doc/source/images/Picture33.png)

* Verify that the model predicts that there is not a risk of heart failure for the patient with these medical characteristics.

![](doc/source/images/failure-no.png?raw=true)

# Troubleshooting


# Privacy Notice
If using the `Deploy to IBM Cloud` button some metrics are tracked, the following
information is sent to a [Deployment Tracker](https://github.com/IBM/cf-deployment-tracker-service) service
on each deployment:

* Node.js package version
* Node.js repository URL
* Application Name (`application_name`)
* Application GUID (`application_id`)
* Application instance index number (`instance_index`)
* Space ID (`space_id`)
* Application Version (`application_version`)
* Application URIs (`application_uris`)
* Labels of bound services
* Number of instances for each bound service and associated plan information

This data is collected from the `package.json` file in the sample application and the `VCAP_APPLICATION` and `VCAP_SERVICES` environment variables in IBM Cloud and other Cloud Foundry platforms. This data is used by IBM to track metrics around deployments of sample applications to IBM Cloud to measure the usefulness of our examples, so that we can continuously improve the content we offer to you. Only deployments of sample applications that include code to ping the Deployment Tracker service will be tracked.

## Disabling Deployment Tracking

To disable tracking, simply remove ``require("cf-deployment-tracker-client").track();`` from the ``app.js`` file in the top level directory.

# Links

* More data science journeys on IBM Code: [https://developer.ibm.com/code/journey/category/data-science/](https://developer.ibm.com/code/journey/category/data-science/)
* IBM Data Science Experience: [https://www.ibm.com/analytics/us/en/watson-data-platform/data-science-experience/](https://www.ibm.com/analytics/us/en/watson-data-platform/data-science-experience/)
* Watson Data Platform: [https://www.ibm.com/analytics/us/en/watson-data-platform/](https://www.ibm.com/analytics/us/en/watson-data-platform/)

# Learn more


* **Artificial Intelligence Code Patterns**: Enjoyed this Code Pattern? Check out our other [AI Code Patterns](https://developer.ibm.com/code/technologies/artificial-intelligence/).
* **Data Analytics Code Patterns**: Enjoyed this Code Pattern? Check out our other [Data Analytics Code Patterns](https://developer.ibm.com/code/technologies/data-science/)
* **AI and Data Code Pattern Playlist**: Bookmark our [playlist](https://www.youtube.com/playlist?list=PLzUbsvIyrNfknNewObx5N7uGZ5FKH0Fde) with all of our Code Pattern videos
* **With Watson**: Want to take your Watson app to the next level? Looking to utilize Watson Brand assets? [Join the With Watson program](https://www.ibm.com/watson/with-watson/) to leverage exclusive brand, marketing, and tech resources to amplify and accelerate your Watson embedded commercial solution.
* **Data Science Experience**: Master the art of data science with IBM's [Data Science Experience](https://datascience.ibm.com/)
* **Spark on IBM Cloud**: Need a Spark cluster? Create up to 30 Spark executors on IBM Cloud with our [Spark service](https://console.bluemix.net/catalog/services/apache-spark)

# License
[Apache 2.0](LICENSE)
