ReadMe file:

## Initial Setup:

Java downloaded
Maven downloaded

Setup pom.xml
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
Setup test runner
```java
    import io.cucumber.junit.Cucumber;
    import io.cucumber.junit.CucumberOptions;
    import org.junit.runner.RunWith;

    @RunWith(Cucumber.class)
    @CucumberOptions(
        features = {"src/test/resource"},
        plugin ="json:target/jsonReports/cucumber-report.json"
    )

    public class TestRunner {}
```
Run 

Add feature file get request 
```
    Scenario: Get Request
        Given I create a "get" request
        When I run a "get" request
        Then the status returns "200"
```
Add step one get request 
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
BaseUrl:
```java
    public static HttpRequest get;
    public static final String BASE_URL_NAME = "https://reqres.in/api/";
```
Add step two get request
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
BaseUrl:
```java
    public static HttpResponse<String> response;
    public static HttpClient httpClient = HttpClient.newBuilder().build();
```
Add step three get request 
```java
    @Then("^the status returns \"([^\"]*)\"$")
    public void the_status_returns(int statusCode) throws Exception {
        assertEquals(statusCode, actualCode);
    }
```
BaseUrl: 
```java    
    public static int actualCode;
```
run

Add step one Post Request with Datatable: 
```
    Scenario Outline: Post Request
        Given I create a "post" request with table
        | Name   | Job   |
        | <name> | <job> |
        When I run a "post" request
        Then the status returns "200"

        Examples:
        | name     | job    |
        | morpheus | leader |
```
Add step one Post request stepDefs 
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
BaseUrl:
```java  
    public static HttpRequest post;
    public static List<Map<String, String>> itemsToLoad;
```
Add step one put request with datatable - follows same route as put request 
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
Add step one put StepDefs 
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
BaseUrl: 
```java    
    public static HttpRequest put;
```
Add step one delete request feature file: 
```
    Scenario: Delete Request
        Given I create a "delete" request
        When I run a "delete" request
        Then the status returns "200"
```
Add step one delete request StepDefs:
```java
    else if(requestType.equalsIgnoreCase("DELETE")) {
        delete = HttpRequest.newBuilder(URI.create(BASE_URL_NAME + "users/2"))
            .DELETE()
            .setHeader("User-Agent", "Java 11 Http bot")
            .build();
    }
```
BaseUrl: 
```java
    public static HttpRequest delete;
```

JSON: 

Add in the feature file line
```
    And the "per_page" from the response is "6"
```
Add in the StepDef
```java
    @Then("^the \"([^\"]*)\" from the response is \"([^\"]*)\"$")
    public void i_print_the_int_from_response(String keyValue, String expectedValue) {
        bodyJson = response.body();
        JSONObject jsonObject = new JSONObject(bodyJson);

        System.out.println("Full response: " + jsonObject);

        assertEquals(getValueFor(jsonObject, keyValue).toString(), expectedValue);

        int topLevel = jsonObject.getInt(keyValue);
        System.out.println("Top level object which is " + keyValue + ": " + topLevel);

    }
```
BaseUrl:
```java
    public static String bodyJson;
```
Add in the common methods 
```java
    public static Object getValueFor(JSONObject jsonObject, String key) {
        return jsonObject.get(key);
    }
```
====

Add in the feature file: 
```
    And I print from the response using key "id"
```
Add in the StepDef: 
```java
    @Then("^I print from the response using key \"([^\"]*)\"$")
    public void i_print_the_response(String keyValue) {
        bodyJson = response.body();
        JSONObject jsonObject = new JSONObject(bodyJson);

        printJsonResponse(jsonObject, keyValue);
    }
```
Add in the Common Methods: 
```java
        public static void printJsonResponse(JSONObject jsonObject, String keyValue) {
        System.out.println("Full response: " + jsonObject);

        String topLevel = String.valueOf(jsonObject.getInt(keyValue));
        System.out.println("Top level object which is " + keyValue + ": " + topLevel);
    }
```
====

Add in feature file:
```
    And I print the response array
```
Add in StepDefs:
```java
    @Then("^I print the response array$")
    public void i_print_the_response_array() {
        bodyJson = response.body();
        JSONObject jsonObject = new JSONObject(bodyJson);

        printResponseArray(jsonObject);
    }
```
Add in Common Methods: 
```java
    public static void printResponseArray(JSONObject jsonObject) {
        JSONArray jsonArray = jsonObject.getJSONArray("data");
        JSONObject firstElement = jsonArray.getJSONObject(0);
        System.out.println("First element from List: " + firstElement);
        System.out.println();

        for (int counter = 0; counter < jsonArray.length(); counter++) {
            JSONObject jsonArrayNestedContents = jsonArray.getJSONObject(counter);

            System.out.println("Json array contents: " + jsonArrayNestedContents);

            String lastNameFromPosting = jsonArrayNestedContents.getString("last_name");
            String avatarFromPosting = jsonArrayNestedContents.getString("avatar");
            String emailFromPosting = jsonArrayNestedContents.getString("email");

            System.out.println();
            System.out.println("array list index: " + counter);
            System.out.println("lastName: " + lastNameFromPosting);
            System.out.println("Avatar: " + avatarFromPosting);
            System.out.println("Email: " + emailFromPosting);

        }
        System.out.println();

        JSONObject somethingFromArray1 = jsonArray.getJSONObject(1);
        System.out.println("Second item from Array: " + somethingFromArray1);

        JSONObject somethingFromArray2 = jsonArray.getJSONObject(2);
        System.out.println("Third item from Array: " + somethingFromArray2);
    }
```
And that's all folks