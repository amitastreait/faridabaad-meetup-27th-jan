## MuleSoft CI/CD

[![My First MuleSoft CI/CD](https://github.com/amitastreait/faridabaad-meetup-27th-jan/actions/workflows/master.yml/badge.svg)](https://github.com/amitastreait/faridabaad-meetup-27th-jan/actions/workflows/master.yml)

# Index

1. [Prerequisites](#prerequisites)
2. [Configure the CloudHub Deployment Strategy (pom.xml)](#configure-the-cloudhub-deployment-strategy-pomxml)
3. [Add MuleSoft public repository](#add-mulesoft-public-repository-optional)
4. [Deploy to Cloudhub](#deploy-to-cloudhub)
5. [Pass the dynamic parameters in Maven Command](#pass-deployment-parameters)
6. [Create Settings.xml file](#create-settingsxml-file-optional)
7. [Create GitHub Actions Workflows](#create-github-actions-workflows)
8. [Commit & Push the change to GitHub Remote](#commit--push-the-change-to-github-remote)
9. [Create Secrets in GitHub Actions](#create-secrets-in-github-actions)
10. [Refer to secrets in the steps](#refer-to-secrets-in-the-steps)
11. [Add Environments in GitHub Actions](#add-environments-in-github-actions)
    - [Steps to create an environment in GitHub Actions](#steps-to-create-an-environment-in-github-actions)

## Prerequisites
- If you are using the HTTP Listener as source for your flow, you need to set its host to `0.0.0.0` and its port to `${http.port}`
- [Click Here for more information](https://docs.mulesoft.com/mule-runtime/4.4/deploy-to-cloudhub)

## Sneak-Peak to Deployment Error

If you get the below error that means you are getting because of the incorrect maven plugin version

![image](https://github.com/amitastreait/faridabaad-meetup-27th-jan/assets/14299807/cd9d623b-4736-439b-ba61-aafb89d535d9)

Below is the Syntax where you need to change the Version. To make the deplopyment work in the POM.xml file.

Change 

```<mule.maven.plugin.version>3.8.0</mule.maven.plugin.version>```

To

```<mule.maven.plugin.version>3.4.0</mule.maven.plugin.version>```

![image](https://github.com/amitastreait/faridabaad-meetup-27th-jan/assets/14299807/9841567b-0bbd-4bb7-88ef-f186681abdcd)

After making the changes, you will be able to see that the deployment is sucess.

![image](https://github.com/amitastreait/faridabaad-meetup-27th-jan/assets/14299807/6b01251b-3903-42aa-9814-b3580fe9b1fc)

![image](https://github.com/amitastreait/faridabaad-meetup-27th-jan/assets/14299807/1388b12f-b563-406d-b8e8-e8d57467f92f)

## Configure the CloudHub Deployment Strategy (pom.xml)

As we waned to automate the Mule project deployment using Maven commands, the first step is to modify our `pom.xml` file and add the Cloudhub deployment strategy. Open your `pom.xml` file and inside the `plugin` element with the `groupId` equal to `org.mule.tools.maven` add the configuration tag like below

```
<configuration>
	<!--  Adding the below tag for CI-CD Purpose -->
	<cloudHubDeployment>
		<uri>https://anypoint.mulesoft.com</uri>
		<muleVersion>${mule.version}</muleVersion>
		<username>${anypoint.username}</username>
		<password>${anypoint.password}</password>
		<environment>${env}</environment>
		<applicationName>${app.name}</applicationName>
		<businessGroup>${businessGroup}</businessGroup>
		<workerType>${wokerType}</workerType>
		<workers>${workerSize}</workers>
		<objectStoreV2>true</objectStoreV2>
	</cloudHubDeployment>
	<!--  Adding the below tag for CI-CD Purpose -->
</configuration>
```

## Add MuleSoft public repository (Optional)

While you are still in `pom.xml` file add the below code to refer the `MuleSoft public Repository` within `pluginRepositories` tag

```
<pluginRepository>
	<id>mulesoft-public</id>
	<name>MuleSoft Public Repository</name>
	<layout>default</layout>
	<url>https://repository.mulesoft.org/nexus/content/repositories/public/</url>
	<snapshots>
		<enabled>false</enabled>
	</snapshots>
</pluginRepository>
```

## Deploy to CloudHub
As we have done the basic setup, you can run the below commands within the project directory from your local machine to see if the command is working properly or not.

Run the below command to test the configuration -
```
mvn clean deploy -DmuleDeploy
```
You will see that a lot more is going on your command prompt like below
![image](https://user-images.githubusercontent.com/14299807/210102764-e7cd5ced-e199-4ae5-86e0-d0f19aca730b.png)

And you will also get the deployment error like below and that is expected because we have to provide more details about the cloud hub deployment like mule version, anypoint username, password and many more details

![image](https://user-images.githubusercontent.com/14299807/210102877-7adf2dce-6f14-4ee6-8cd5-ae1d05b7d6be.png)

## Pass deployment parameters
If you look at the code snippet that we have [configure-the-cloudhub-deployment-strategy-pom.xml](#configure-the-cloudhub-deployment-strategy-pomxml) section, you will notice that we are using the dynamic parameters like `${mule.version}` and similary the values for others as well. To pass the value dynamically, we need to modify our deploy command and pass the values using command only.

To pass the value in any parameter that we have defined in a dynamic way in `pom.xml` we need to use `-D` followed by the parameter name and then `=` and then the parameter value.

For Example, to pass the value for `mule.version` variable, we can use the command like 

```
mvn clean deploy -DmuleDeploy -Dmule.version=4.4.0
```

> One important thing that we need to remember is that there should not be any space after `-D` and after `=`.

Prepare the final command by passing all the parameters and your command will look like below

```
mvn clean deploy -DmuleDeploy -DskipTests -Dmule.version=4.4.0 -Danypoint.username=youranypointplatform-username -Danypoint.password=anypointplatform-password -Denv=Sandbox -Dapp.name=mule-learning-amit-123 -DbusinessGroup=PantherSchools -DworkerSize=1 -DwokerType=MICRO
```

> The value for businessGroup must be the absolute path for your business group that you wanted to deploy the application

Now, again run the above command from the command promot and see the outcome

You will see the outcome like below syaing your application is being deployed - 
![image](https://user-images.githubusercontent.com/14299807/210104532-5be62e79-69a2-4fa4-97f9-f3842cb68b21.png)

## Create `settings.xml` file (optional)
To configure your Enterprise Maven Repository credentials with embedded Maven to allow you to reference and use dependencies only available from the Enterprise Repository. Please be advised that this is only applicable for customers with an Enterprise license. This also reminds you to update your pom.xml after updating your `settings.xml` file.

<!-- So far, we did the setup for CloudHub deployment and tested that it is working fine from our local machine and the application got deployed. Now, as our goal is to setup the CI-CD using GitHub Actions, we need to create the `settings.xml` file to store the credentials for our anypoint platform so that the plugins and dependencies can be downloaded that are required for the project. -->

1. Open your project in the System Explorer and create a `.mavens` folder at the top level
2. Now, create the `settings.xml` file within the `.mavens` folder
3. Use the below sample code for the xml file

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<settings>
	<servers>
		<server>
			<id>my.anypoint.credentials</id>
			<username>my.anypoint.username</username>
			<password>my.anypoint.username</password>
		</server>
	</servers>
	<plugin>
		<configuration>
			<cloudHubDeployment>
				<server>my.anypoint.credentials</server>
			</cloudHubDeployment>
		</configuration>
	</plugin>
	<pluginGroups>
		<pluginGroup>org.mule.tools</pluginGroup>
	</pluginGroups>
	<profiles>
		<profile>
			<id>mule-extra-repos</id>
			<activation>
				<activeByDefault>true</activeByDefault>
			</activation>
			<repositories>
				<repository>
					<id>mule-public</id>
					<url>https://repository.mulesoft.org/nexus/content/repositories/public</url>
				</repository>
			</repositories>
			<pluginRepositories>
				<pluginRepository>
					<id>mule-public</id>
					<url>https://repository.mulesoft.org/nexus/content/repositories/public</url>
				</pluginRepository>
			</pluginRepositories>
		</profile>
		<profile>
			<id>Mule</id>
			<activation>
				<activeByDefault>true</activeByDefault>
			</activation>
			<repositories>
				<repository>
					<id>my.anypoint.credentials</id>
					<name>MuleRepository</name>
					<url>https://repository.mulesoft.org/nexus-ee/content/repositories/releases-ee/</url>
					<layout>default</layout>
					<releases>
						<enabled>true</enabled>
					</releases>
					<snapshots>
						<enabled>true</enabled>
					</snapshots>
				</repository>
			</repositories>
		</profile>
	</profiles>
</settings>

```

## Create GitHub Actions Workflows
Now, we are all set to setup the CI-CD using GitHub Actions, and the first step is to create couple of folders. See the below instructions for the same

- Navigate to Project folder in your local machine
- Create `.github` folder at the top level of project structure
- Open `.github` folder and create `workflows` folder within `.github` folder
- Open `workflows` folder and create `main.yml` file within `workflows` folder and use below starter code

```
name: GitHub Actions Demo
run-name: ${{ github.actor }} is testing out GitHub Actions 🚀
on: [push]
jobs:
  Explore-GitHub-Actions:
    runs-on: ubuntu-latest
    steps:
      - run: echo "🎉 The job was automatically triggered by a ${{ github.event_name }} event."
      - run: echo "🐧 This job is now running on a ${{ runner.os }} server hosted by GitHub!"
      - run: echo "🔎 The name of your branch is ${{ github.ref }} and your repository is ${{ github.repository }}."
      - name: Check out repository code
        uses: actions/checkout@v3
      - run: echo "💡 The ${{ github.repository }} repository has been cloned to the runner."
      - run: echo "🖥️ The workflow is now ready to test your code on the runner."
      - name: List files in the repository
        run: |
          ls ${{ github.workspace }}
      - run: echo "🍏 This job's status is ${{ job.status }}."
```

## Commit & Push the change to GitHub Remote
Finally after we have prepared all the configuration, folders and file. Commit all the files and push it to remote repo.
[For Git Commands Click Here](https://people.irisa.fr/Anthony.Baire/git/git-for-beginners-handout.pdf)

## Create Secrets in GitHub Actions
After you have pushed the changed to GitHub remote your CI-CD actions pipeline will automatically run ( need not to worry ). Before we start making any changes into the `yml` file we need to do some ground work so that we can develop a secure pipeline.

If you remember, in our maven deploy command we have passed some dynamic parameters and one of the parameters was password for mulesoft anypoint platorm and the username for the platform. So if we pass the value in plain text then anyone can read it easily and then the security threat will be there.

So to mitigate the security risk, we will create Secrets in GitHub Actions and will store the sensitive information in the secrets (these secrets are encrypted at rest and will never be exposed even if you try to print)

To Create the Secrets follow the below steps

- Open the Github repo
- Click on the Settings tab on the tab ( if you do not see settings tab, then make sure you are the admin of the repo )
- Expand `Secrets` on the left side and click `Actions`
![image](https://user-images.githubusercontent.com/14299807/210107307-7b295c1a-b13a-4aaa-823e-f4eaa7c31cf0.png)

- Click on the `New Repository Secret`, to create the secrets
![image](https://user-images.githubusercontent.com/14299807/210107374-bfa34401-6a86-442c-bdb5-f922cb6d5f2f.png)
![image](https://user-images.githubusercontent.com/14299807/210107394-e00ea1c8-3e8d-43cc-80f1-343b155da37c.png)

- For the provide the name of the secret for example, `ANYPOINT_USERNAME` & for value enter the correct value

Repeat the same process to store the other information like `ANYPOINT_PASSWORD` and other information
![image](https://user-images.githubusercontent.com/14299807/210107530-e523d425-91a6-450b-877a-cf008f5e6b61.png)

## Refer to secrets in the steps
- To access the secrets within the GitHub Action pipeline we need to first use $ followed by double opening flower brackets( {{ ) & double closing flower brackets( }} ). Example ` ${{ }} `.
- Within the flower brackets use the `secrets` keyword followed by the period `.` statement followed by the name of the secrets. For Example - `${{ secrets.DECRYPTION_KEY }}`
- Replace all the hardcoding values with the secrets that you have just created.
- Below is the modified code for the `build` job

## Add Environments in GitHub Actions
Adding the environment in Github Action is very important because whenever you are making the changes to the codebase and pushing the changes the validation runs against the org. What if you wanted to deploy the code to different environments like Integration, QA, Staging, SIT, &, etc and this will be an obvious use case? You must be deploying the code to a different environment.

### Steps to create an environment in GitHub Actions
1. Open the Repo where you wanted to create an environment
2. Click on **Setting** tab to open the settings
3. Locate **Environment** item from the left side
4. Click on **New Environment** to create a New Environment
5. Give a name & click on **Configure Environment**

![image](https://user-images.githubusercontent.com/14299807/202894602-689f2a2d-9e35-408b-8c68-12c8eb59902e.png)

![image](https://user-images.githubusercontent.com/14299807/202894631-7475c2bb-a357-4594-a4a9-a19a94921995.png)

![image](https://user-images.githubusercontent.com/14299807/202894723-0fe21564-1c95-4e1e-b48b-77b0822e4773.png)

> Congratulation :tada: you have created the environment. If you wanted to create more environment then follow the same steps

## Configure CI-CD Pipelne 
So far we did the ground work and now we are ready with all the ingrediants that are needed to develop a secure and scussfull pipeline. Let's follow the step by step approach to complete the Pipeline.

### YML File Structure
It is very important for us to Understand the YML file structure so that we can develop the script without any issue and trust me it is very easy to YML script in the pipeline.

Below is the sample structure

```
name: Build and Deploy to Sandbox in Cloudhub

on:
  push:
    branches:
      - feature/*
      - developer
    paths:
      - 'src/**'
  pull_request:
    branches:
      - master
      - main
    types:
      - closed
    paths:
      - 'src/**'
  workflow_dispatch:
    
jobs:
  build:
    runs-on: ubuntu-latest
    env:
    steps:
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:    
```

Every YML file (CI-CD workflow) will have the following details

- name :- The name of the job which will be displayed when the pipeline will run
- on :- Here we specify the events on which the pipeline will be triggered. For example, push, pull_request, tags, & etc
- jobs :- Every workflow can have multiple job and each job can have below steps
	1. runs-on:- We need to specify the machine where we wanted to run the pipeline for example ubnuntu-latest, windows-latets and many more
	2. env:- We can specify the variables that can be accessible across all the steps within this job
	3. steps:- multiple steps like build, test, deploy and others 

### Add the Job to prepare the Build of latest code
The very first job is to build the code that we have developed using MuleSoft anypoint Studio. To build the job we will use maven commands and to use maven we need couple of things 

- JDK must be installed
- We must checkout the latest code so that our command build the latest code.
- See the below samle Job for build

```
build:
    runs-on: ubuntu-latest
    
    env:
      NEXUS_USERNAME: ${{ secrets.NEXUS_USERNAME }}
      NEXUS_PASSWORD: ${{ secrets.NEXUS_PASSWORD }}
      ANYPOINT_USERNAME: ${{ secrets.ANYPOINT_USERNAME }}
      ANYPOINT_PASSWORD: ${{ secrets.ANYPOINT_PASSWORD }}
      SANDBOX_CLIENT_ID: ${{ secrets.SANDBOX_CLIENT_ID }}
      SANDBOX_CLIENT_SECRET: ${{ secrets.SANDBOX_CLIENT_SECRET }} 
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
      SECURE_KEY: ${{ secrets.SECURE_KEY}}
      SETTINGS_FILE: ${{ secrets.SETTINGS_FILE}}
    
    steps:

    - uses: actions/checkout@v3
      id: checkout-code
      with:
        fetch-depth: 0
    
    - uses: actions/cache@v3.2.1
      id: prepare-chache
      with:
        path: ~/.m2/repository
        key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
        restore-keys: |
          ${{ runner.os }}-maven-
    
    - name: Set up Java JDK
      id: setup-jdk
      uses: actions/setup-java@v3
      with:
        distribution: 'zulu'
        java-version: 8
    
    - name: Build with Maven
      id: maven-build
      run: mvn clean -B package -s .maven/settings.xml
      # run: mvn -B package --file pom.xml
          
    - name: Stamp artifact file name with commit hash
      id: prepare-artifacts
      run: |
        artifactName1=$(ls target/*.jar | head -1)
        commitHash=$(git rev-parse --short "$GITHUB_SHA")
        artifactName2=$(ls target/*.jar | head -1 | sed "s/.jar/-$commitHash.jar/g")
        mv $artifactName1 $artifactName2
    
    - name: Upload artifact 
      id: upload-artifacts
      uses: actions/upload-artifact@master
      with:
          name: artifacts
          path: target/*.jar
          
```

Every job can have multiple steps and if you see the build job we have mutiple steps

1. checkout-code :- This step will checkout the latest code and we will be on the directory where we will have latest code
2. prepare-chache :- This step is very important because we will have multiple dependencies in our job, so while first run the pipleline it will take as it will download all the dependency repositories and then we are caching them so that from the next time onwards we will reuse the existing cache to improve the performance of our CI-CD Job
3. setup-code :- In this step we are installing the JDK version 8 however you can change it based on the maven plugin supported JDK
4. maven-build :- This is the heart of this job where we are running build using our pom.xml file
5. prepare-artifacts :- preparing the artifacts to be uploaded and we are using commit hash to prepare the artifacts so that if there is any changes in pom.xml we can easily identify that there should be a new artifact.
6. upload-artifacte :- Finally upload the artifacts so that we can utilize the same in next step for deploying the code to cloud hub
