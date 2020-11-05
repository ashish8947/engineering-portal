---
title: "Automating Rest API's using Cucumber and Java"
date: "2020-11-05"
coverImage: "cucumber_rest_assured.png"
author: "Surendranath Reddy Birudala"
tags: ["Automation", "Cucumber", "Rest API", "Java"]
description: "This article is about basic overview of how to automate Rest API using Cucumber and JAVA."
---

## What is Rest?
Representational State Transfer in short-form as **REST** defines a set of constraints for creating Web Services.
Rest API is the most-used web service technology nowadays, and it's an almost meaningless description. A REST API is a way to communicate for two computer systems over HTTP, which is similar to web browsers and servers.

## What is BDD?
BDD stands for **Behavior Driven Development** (BDD). Nowadays, many Organizations, to get a better advantage of testing, are taking a step forward.  

 - BDD allows us to create test scripts from both the developer’s and the customer’s perspective.
 - BDD uses human-readable descriptions of software user requirements as the basis for software tests

## What is Cucumber?
A cucumber is an approach that supports BDD. It allows you to write tests that anyone can understand, irrespective of their technical knowledge. In BDD, users (business analysts, product owners, developers, etc..). We need to first write scenarios or acceptance tests that describe the system's behavior from the customer's point of view for review and sign-off by the product owners before developers start actual development.

## What are the Prerequisites to Test Rest API using Cucumber?
 - Java
 - Editor (Eclipse, IntelliJ, etc..)
 - Maven
 
### Steps to Install Java

    http://www.oracle.com/technetwork/java/javase/downloads/index.html  

 
Download JDK from the above link and install it
Set the Environment Variable on System Properties 

    Right Click on My PC -> Properties -> Advanced System Settings -> Environment Variables -> Create New -> provide path of JDK

### Steps to Install Eclipse

 - Make Sure Java is installed on your PC.
 - Use the below link to download Eclipse 
    https://eclipse.org/downloads
 - Install Eclipse on your PC 

**Steps to Install Maven**

 - Use the below link to download Maven
https://maven.apache.org/download.cgi
 - Set the environment variable.

### Configuring Cucumber with Maven

- Create a Maven Project. Use below navigation.

	`Open Eclipse -> File -> New -> Maven Project`
     
- Provide Group Id and Artifact Id and click on finish. 

  **Group Id:** This element indicates the organization's unique identifier or group that created the project. The groupId is one of the key identifiers of a project and is typically based on your organization's fully qualified domain name. For example, `com.loginradius`  is the designated groupId for all Maven plugins.

  **Artifact Id:**  it points to the unique base name of the primary artifact generated by this project. The main or primary artifact for a project is typically a JAR file. Secondary artifacts like source bundles also use the artifactId as part of their final name.

- Open pom.xml file to add necessary dependencies 
  
 	```
        <dependency>
		<groupId>io.cucumber</groupId>
		<artifactId>cucumber-java</artifactId>
		<version>6.6.0</version>
	</dependency>
	<dependency>
		<groupId>io.cucumber</groupId>
		<artifactId>cucumber-testng</artifactId>
		<version>6.6.0</version>
	</dependency>
	<dependency>
		<groupId>io.rest-assured</groupId>
		<artifactId>rest-assured</artifactId>
		<version>4.3.0</version>
		<scope>test</scope>
	</dependency>
	<dependency>
		<groupId>org.testng</groupId>
		<artifactId>testng</artifactId>
		<version>7.1.0</version>
		<scope>test</scope>
	</dependency>
	```
	
 Now we need three Important files.
 - Feature file
 - StepDefination file
 - Runner file
 
 **Feature File:**  It's a entry point to the cucumber. We use Gherkins to write the feature file. This is the file where you will describe your descriptive language(Gherkin uses simple English). A feature file can contains a Scenario or List of Scenario. A sample Feature file is below
 

 ```
 Feature: Login Functionality
 @validLogin
 Scenario: User Should Login With Valid Scredentials 
    Given Post Login API
    When Provide Valid Credential
    Then Status_code equals 200    
    And  response contains IsLogin equals "true"
@invalidLogin    
Scenario Outline: Email and Password Validation in Login API 
    Given Post Login API
    When Provide different combinations to "<email>""<password>"
    Then Status_code equals <statuscode>    
    And  response contains message equals "<message>"
    Examples:
    |email     		|password    | statuscode |  message
    |          		|            |   401      | Required email and password
    | abc	  	|            |   401      | Email format is incorrect
    | abc@mail7.io  	|	     |   401      | Required password
    | abc@mail7.io   	| password   |   401      | Email and Password combination Incorrect
   ```
  
 Save the above code as login.feature 

 - **Feature:** It indicates the name of the feature under the test.
 - **Description:** It indicates a meaningful description of the feature (Optional).
 - **scenario:** scenario indicates the steps and expected outcomes for a particular test case.
 - **Scenario Outline:** Single scenario can be executed for multiple data sets using scenario outlines. The data is provided by a tabular structure separated by (I I).
 - **Given:** Prerequisite before the test steps get executed.
 - **When:** Specific condition which should match to execute the next step.
 - **Then:** What should happen if the condition mentioned in WHEN is satisfied.
 - **Examples:** Its a tabular format input data to pass to scenario outline.
 - **<>:** anything if you write between is variable. 

**StepDefinition:**
Now you have your feature file ready with test scenarios defined. However, our job is not done yet. Cucumber doesn't know when should execute which part of code. So StepDefinition acts as an intermediate to your runner and feature file. It stores the mapping between each step of the scenario in the Feature file. So when you run the scenario, it will scan the stepDefination file to check matched glue.

**How to write step definition:**
We have a chrome extension(Tidy Gherkin) to convert your feature into a step definition.  
Copy your scenario from the feature file and paste it in Tidy Gherkin and click on Java Steps; copy the Java Steps.
Create a new file name a StepDefinition.java and paste the Java Steps

```
package com.loginradius.login
import cucumber.api.java.en.Given;
import cucumber.api.java.en.When;
import cucumber.api.java.en.Then;
import cucumber.api.java.en.And;
import cucumber.api.junit.Cucumber;
import org.junit.runner.RunWith;
import static io.restassured.RestAssured.given;
import static org.testng.Assert.assertTrue;
import io.restassured.http.ContentType;
import io.restassured.response.Response;
import io.restassured.specification.RequestSpecification;
import io.restassured.RestAssured;

public class StepDefinition {
	private  static  final  String  BASE_URL  =  "https://www.loginradius.com";
	String email = "abcxyz@mail7.io;
	String password = "password";
	RequestSpecification request;
	private  static  Response response;
	private  static  String  jsonString;

    @Given("Post Login API")
    public void post_login_api() {
	    RestAssured.baseURI  =  BASE_URL;
		request  =  RestAssured.given();
		request.header("Content-Type",  "application/json");    
    }
    
   @When("Provide Valid Credential")
    public void provide_valid_credential() {
      response  =  request.body("{ \"userName\":\""  +  email  +  "\", \"password\":\""  +  password  +  "\"}")
						  .post("/user/login");	
    }

   @Then("Status_code equals {int}")
    public void statuscode_equals_(int agr) {
	    Assert.assertEquals(arg, response.getStatusCode());
    }
	
	@And("response contains IsPosted equals {string}")
    public void response_contains_IsPosted_equals_(String message) {
	    Assert.assertEquals(message, getJsonPath(response, "IsPosted"));
    }	

   @And("response contains message equals {string}")
    public void response_contains_equals_(String message) {
	    Assert.assertEquals(message, getJsonPath(response, "message"));
    }
    
	@When("Provide different combinations to {string}, {string}")
    public void provide_different_combinations_to_somethingsomething(String email, String password){
	    response = request.body("{ \"userName\":\"" + email + "\", \"password\":\"" + password + "\"}") .post("/user/login");
    }

	public String getJsonPath(Response response, String key) {
		String resp = response.asString();
		JsonPath js = new JsonPath(resp);
		return js.get(key).toString();
	}
	
}
```

Now we have Our Feature file and StepDefinition Ready, we need a runner file to run our Test Scenario's

**Runner File:**
A runner will help us to run the feature file and acts as an interlink between the feature file and StepDefinition Class
Below is the code which will help you to run the tests

```
package runner;
import io.cucumber.testng.CucumberOptions;

@CucumberOptions(
		features= {"feature_files/login.feature"},
        glue= {"step_definations/StepDefinitation.java"},
        tags= "@validLogin",    // based on tags scenarios will run
        monochrome=true, dryRun=false,
		)	
public  class  TestRunner  {

}
```
Now We have Feature, StepDefinition, and runner files ready, We can run the runner file to see the results.

## Conclusion

Cucumber is very useful in understanding the overall testing process without having coding knowledge. Since it uses simple English to write the feature file and makes developer, QA and other stakeholders on same page before starting the actual development.



