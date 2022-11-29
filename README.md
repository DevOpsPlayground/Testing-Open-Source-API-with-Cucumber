# ReadMe file:

---
## Summary:

Many modern systems use APIs to interact with a source database or application and port data into a frontend application, which powers business process needs to meet the demands of clients. API testing is an important aspect of the overall testing suite.

This template will introduce you to API testing with Java, Cucumber and Maven. 
It will also teach you how to write a suite of tests in Java for an API.
In my example I have used the open source hosted REST populated API called 'Reqres'.

---
## Outcomes:

- Understand the structure of a Java Maven project 
- Understand how to test APIs with Http requests & responses
- Understand how to format and parse JSON
- Understand how to write API tests in Java and Cucumber

---
## Pre-requisites

Knowledge of basic Java programming
Knowledge of APIs and Http methods 

---
## Support:

If you have any questions about the topics in this pathway or need a helping hand with a concept or completing the validation task, reach out to us on the playground slack.

---
## Framework Setup

we'll want to add Cucumber, Junit and Json dependencies via the pom.xml file. The Project Object Model or POM is the fundamental unit of work in Maven. It is an XML file that contains information about the project and configuration details used by Maven to build the project.

To do this you can copy the lines below, but if we were doing this from the beginning we would have to go onto the [Maven repository](https://mvnrepository.com/)
and find the three cucumber plugins required:

```bash
    cucumber-java
    cucumber-junit
    json
```
But for our example we can copy and add the XML for the dependencies to your pom.xml file below:

```xml
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-java</artifactId>
            <version>7.8.1</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-junit</artifactId>
            <version>7.8.1</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.json</groupId>
            <artifactId>json</artifactId>
            <version>20171018</version>
            <scope>test</scope>
        </dependency>
```

Once that is done we can setup the test runner. This will allow us to run the tests using the command ```mvn test``` 

Please copy the below lines into the TestRunner.java file located in the src -> test -> java folders:

```java
    @RunWith(Cucumber.class)
    @CucumberOptions(
        features = {"src/test/resource"},
        plugin ="json:target/jsonReports/cucumber-report.json"
    )

    public class TestRunner {}
```

After that has been inputted and saved we can run the project using ```mvn test``` in the terminal. If it builds successfully, then we know the project is setup correctly.

---
## Adding in HTTP requests:

To start creating Http requests and parsing json responses we'll want to start with a GET request. 
We will use a generic API that conforms to REST principles and accepts a content type of JSON.

https://reqres.in/  already has data and gives free access to
 GET, POST, PUT and DELETE requests.

The primary or most-commonly-used HTTP verbs (or methods, as they are properly called) are POST, GET, PUT, PATCH, and DELETE. These correspond to create, read, update, and delete (or CRUD) operations, respectively. There are a number of other verbs, too, but are utilized less frequently.

We've prebuilt in the abstraction for this codebase, where we put commonly repeating variables into a separate file to be called upon. These files are Base_Url and then Common_Methods. The actual requests themselves will occur in the HttpRequestSteps and HttpRequestsFeatures
The request examples follow the same basic pattern but with slight variations. 

We'll be using HttpRequest for GET, POST, PUT and DELETE requests.

We'll also be writing our feature files in the Gherkin language with Cucumber.

Cucumber is a software framework that supports behavior-driven development. Central to the Cucumber BDD approach is its ordinary language parser called Gherkin. It allows expected software behaviors to be specified in a logical language that customers can understand.

---

### Get Request:

First we'll add the feature file get request 
```
    Scenario: Get Request
        Given I create a "get" request
        When I run a "get" request
        Then the status returns "200"
```

Then we'll run with ```mvn test```. You can see the application will generate steps for you, which is a fantastic feature of cucumber. Particularly since variations in the feature file step and the step definition can cause errors. 

After we've run we can add the step definition for our first feature file line. 
```java
    @Given("^I create a \"([^\"]*)\" request$")
    public void i_create_a_request(String requestType) throws Exception {
    if(requestType.equalsIgnoreCase("GET")) {
        get = HttpRequest.newBuilder(URI.create(BASE_URL_NAME + "users?page=2"))
            .GET()
            .setHeader("User-Agent", "Java 11 Http bot demo")
            .build();
        }
    }
```

Currently the get and BASE_URL_NAME variables are undefined, we'll be defining those in our Base_Url.java file:

```java
    public static HttpRequest get;
    public static final String BASE_URL_NAME = "https://reqres.in/api/";
```

After this we can define the second line in the step definition, which will run the request.

```java
    @When("^I run a \"([^\"]*)\" request$")
    public void i_run_a_request(String requestType) throws Exception {
    if(requestType.equalsIgnoreCase("GET")) {
        response = httpClient.send(get, HttpResponse.BodyHandlers.ofString());
        actualCode = response.statusCode();
        System.out.println("Request: " + response);
        } 
    }
```
And the response and httpClient are both undefined, so we will define these in our Base_Url file:
```java
    public static HttpResponse<String> response;
    public static HttpClient httpClient = HttpClient.newBuilder().build();
```
Finally we can add the final step, where we are comparing the actual status we're receiving with the expected status we're declaring in the feature file.
```java
    @Then("^the status returns \"([^\"]*)\"$")
    public void the_status_returns(int statusCode) throws Exception {
        assertEquals(statusCode, actualCode);
    }
```
Then we have the actualCode which isn't defined in our Step definition, we can put this into our Base_Url file:

```java    
    public static int actualCode;
```

Then we can run again in the terminal using ```mvn test```. We should have a successful build.


---

### Post Request:

First we'll want to add the Post request with a Datatable into our feature file. Once again, Cucumber is a Behavioral Driven Development (BDD) framework that allows developers to create text-based test scenarios using the Gherkin language.

In many cases, these scenarios require mock data to exercise a feature, which can be cumbersome to inject — especially with complex or multiple entries. We can mock this data using data tables to pass data from the feature file into the step definition and here we'll see how to do this. Please add the following code to the feature file:


```
    Scenario Outline: Post Request
        Given I create a "post" request with table
        | Name   | Job   |
        | <name> | <job> |
        When I run a "post" request
        Then the status returns "201"

        Examples:
        | name     | job    |
        | morpheus | leader |
```

Then we'll want to define that step with the creation of a post request: 

```java
    @Given("^I create a \"([^\"]*)\" request with table$")
    public void i_create_a_request_with_table(String requestType, DataTable dataTable) {
        if(requestType.equalsIgnoreCase("POST")) {
            itemsToLoad = dataTable.asMaps(String.class, String.class);
            for (int i = 0; i < itemsToLoad.size(); i++) {
                post = HttpRequest.newBuilder(URI.create(BASE_URL_NAME + "users"))
                        .POST(HttpRequest.BodyPublishers.ofString("{\n" +
                                "    \"name\": \"" + itemsToLoad + "\",\n" +
                                "    \"job\": \"" + itemsToLoad + "\"\n" +
                                "}"))
                        .setHeader("User-Agent", "Java 11 Http bot")
                        .build();
            }
        } 
    }
```
And we'll want to add the ability to send our post request as well under the ```@When("^I run a \"([^\"]*)\" request$")``` as an else if statement. This will be in the step definition file.

```java
    else if(requestType.equalsIgnoreCase("POST")) {
        response = httpClient.send(post, HttpResponse.BodyHandlers.ofString());
        actualCode = response.statusCode();
        System.out.println("Request: " + response);
    } 
```
You'll see we haven't defined the post variable and the itemsToLoad variable, so we'll once again be going back to the Base_Url file to declare these. 
```java  
    public static HttpRequest post;
    public static List<Map<String, String>> itemsToLoad;
```
Then we can run this once more with ```mvn test``` and see the result:

---

### Put Request:

Once again we'll start with the feature file, since a Put is updating then it'll follow the same style as the earlier Post request, which was creating. 
```
     Scenario Outline: Put Request
        Given I create a "put" request with table
        | Name   | Job   |
        | <name> | <job> |
        When I run a "put" request
        Then the status returns "200"

        Examples:
        | name     | job    |
        | morpheus | leader |
```
Once the above is done we'll want to fill in the step definition to define what we've put into the feature file:
```java
    else if (requestType.equalsIgnoreCase("PUT")) {
            itemsToLoad = dataTable.asMaps(String.class, String.class);
            for (int i = 0; i < itemsToLoad.size(); i++) {
                put = HttpRequest.newBuilder(URI.create(BASE_URL_NAME + "users/2"))
                        .PUT(HttpRequest.BodyPublishers.ofString("{\n" +
                                "    \"name\": \"" + itemsToLoad + "\",\n" +
                                "    \"job\": \"" + itemsToLoad + "\"\n" +
                                "}"))
                        .setHeader("User-Agent", "Java 11 Http bot")
                        .build();
            }
        }
```
And once again we'll want to send the request, to do this we'll want to put another else if under the ```@When("^I run a \"([^\"]*)\" request$")``` as an else if statement. Once again this will be in the step definition file.
```java
        else if(requestType.equalsIgnoreCase("PUT")) {
        response = httpClient.send(put, HttpResponse.BodyHandlers.ofString());
        actualCode = response.statusCode();
        System.out.println("Request: " + response);
        }

```
Finally we want to declare our put variable in the Base_Url file:
```java    
    public static HttpRequest put;
```

---

### Delete Request:

We'll begin with the feature file, the delete request will be straight forward and follow the same style as the get request we did originally. 
```
    Scenario: Delete Request
        Given I create a "delete" request
        When I run a "delete" request
        Then the status returns "204"
```
Then we can define the step we created in the feature file. We'll add this under our get code since the creation are so similar. 
```java
    else if(requestType.equalsIgnoreCase("DELETE")) {
        delete = HttpRequest.newBuilder(URI.create(BASE_URL_NAME + "users/2"))
            .DELETE()
            .setHeader("User-Agent", "Java 11 Http bot")
            .build();
    }
```

And we'll want to send this request off as well, so under the ```@When("^I run a \"([^\"]*)\" request$")``` in the step definitions file.

```java
    else if(requestType.equalsIgnoreCase("DELETE")) {
        response = httpClient.send(delete, HttpResponse.BodyHandlers.ofString());
        actualCode = response.statusCode();
        System.out.println("Request: " + response);
    }

```
And we can declare the delete variable in our Base_Url file:
```java
    public static HttpRequest delete;
```

---

## JSON Manipulation:

JSON is an open standard file format and data interchange format that uses human-readable text to store and transmit data objects consisting of attribute–value pairs and arrays. 

---

### Top level Json manipulation:

First we add in the feature file line for grabbing a value from the top level and comparing it to what we expect it to be. We'll be doing this at the end of the get request:

```
    And the "per_page" from the response is "6"
```

Then we furnish the step for this feature file line inside the step definition file.

```java
    @Then("^the \"([^\"]*)\" from the response is \"([^\"]*)\"$")
    public void i_print_the_int_from_response(String key, String expectedValue) {
        bodyJson = response.body();
        JSONObject jsonObject = new JSONObject(bodyJson);

        System.out.println("Full response: " + jsonObject);

        assertEquals(getValueFor(jsonObject, key).toString(), expectedValue);

        int topLevel = jsonObject.getInt(key);
        System.out.println("Top level object which is " + key + ": " + topLevel);

    }
```
And as we've declared a new variable called bodyJson we can add this into the Base_Url file.

```java
    public static String bodyJson;
```

Finally, we're going to start filling in the Common_Methods file, the first line we'll add will be the below, which will allow us to grab the value:

```java
    public static Object getValueFor(JSONObject jsonObject, String key) {
        return jsonObject.get(key);
    }
```
---

### Top level Json manipulation:

Again we'll add the line into the feature file under the post request:

```
    And I print from the response using key "id"
```

Then we once again add in the underlying code in the step definition file. 

```java
    @Then("^I print from the response using key \"([^\"]*)\"$")
    public void i_print_the_response(String key) {
        bodyJson = response.body();
        JSONObject jsonObject = new JSONObject(bodyJson);

        printJsonResponse(jsonObject, key);
    }
```

And then in common methods we'll add the below method which we are calling in the step definition:

```java
        public static void printJsonResponse(JSONObject jsonObject, String key) {
        System.out.println("Full response: " + jsonObject);

        String topLevel = String.valueOf(jsonObject.getInt(key));
        System.out.println("Top level object which is " + key + ": " + topLevel);
    }
```
---

### Nested level Json manipulation:

Finally, we have nested level Json arrays. First we'll be adding in the below line into the feature file at the end of the Get request:

```
    And I print the response array
```

Then we'll be furnishing the step definition with the below code: 

```java
    @Then("^I print the response array$")
    public void i_print_the_response_array() {
        bodyJson = response.body();
        JSONObject jsonObject = new JSONObject(bodyJson);

        printResponseArray(jsonObject);
    }
```

And finally, we'll add in the below method to the Common_Methods file.

```java
        public static void printResponseArray(JSONObject jsonObject) {
        JSONArray jsonArray = jsonObject.getJSONArray("data");
        JSONObject firstElement = jsonArray.getJSONObject(0);
        System.out.println("First element from List: " + firstElement);
        System.out.println();

        for (int counter = 0; counter < jsonArray.length(); counter++) {
            JSONObject jsonArrayNestedContents = jsonArray.getJSONObject(counter);

            String lastNameFromPosting = jsonArrayNestedContents.getString("last_name");
            String avatarFromPosting = jsonArrayNestedContents.getString("avatar");
            String emailFromPosting = jsonArrayNestedContents.getString("email");

            System.out.println();
            System.out.println("array list index: " + counter);
            System.out.println("Json array contents: " + jsonArrayNestedContents);
            System.out.println("lastName: " + lastNameFromPosting);
            System.out.println("Avatar: " + avatarFromPosting);
            System.out.println("Email: " + emailFromPosting);

        }
        System.out.println();

        JSONObject somethingFromArray1 = jsonArray.getJSONObject(1);
        System.out.println("Second item from Array: " + somethingFromArray1);

        System.out.println();

        JSONObject somethingFromArray2 = jsonArray.getJSONObject(2);
        System.out.println("Third item from Array: " + somethingFromArray2);
        System.out.println();
    }
```

And that's all folks
