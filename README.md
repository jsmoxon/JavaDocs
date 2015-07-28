#Getting Started with Java

Sauce Labs is a cloud platform for executing automated and manual mobile and web tests. Sauce Labs supports running automated tests with Selenium WebDriver (for web applications) and Appium (for native and mobile web applications).

In this tutorial we'll show you how to run a test with Selenium WebDriver and Java on Sauce Labs.

##Table of Contents
1. [Quickstart](https://github.com/jsmoxon/JavaDocs#quickstart)
1. [Prerequisites](https://github.com/jsmoxon/JavaDocs#prerequisites)
2. [Code Example](https://github.com/jsmoxon/JavaDocs#code-example)
3. [Running Tests on Sauce](https://github.com/jsmoxon/JavaDocs#running-tests-on-sauce)
4. [Running Against Local Applications](https://github.com/jsmoxon/JavaDocs#running-tests-against-local-applications)
5. [Running Tests in Parallel](https://github.com/jsmoxon/JavaDocs#running-tests-in-parallel)
6. [Reporting to the Sauce Labs Dashboard](https://github.com/jsmoxon/JavaDocs#reporting-to-the-sauce-labs-dashboard)

##Quickstart
Configuring Selenium tests to run on Sauce Labs is simple. The basic change is just to switch from using a local Selenium driver:

```java
WebDriver driver = new FirefoxDriver()
```

To using a remote driver pointed at ondemand.saucelabs.com, specifying your Sauce Labs account credentials and desired browser configuration:

```java
DesiredCapabilities caps = DesiredCapabilities.chrome();
caps.setCapability("platform", "Windows 8.1");
caps.setCapability("version", "43.0");
WebDriver driver = new RemoteWebDriver(new URL("http://USERNAME:ACCESS_KEY@ondemand.saucelabs.com:80/wd/hub"), caps);
```

To get things working really well, we recommend adding a number of features to your setup from here:
 - _test parallelization_
 - _metadata reporting via our API_
 - _local network access with Sauce Connect_

We'll walk you through setting those things up one at a time so that you can see how each piece works. If you'd rather just start from a complete example with all the bells and whistles, you can [jump straight to the end]().

##Prerequisites
Before running automated tests on Sauce Labs you must install JDK 1.6 (or higher) (not JRE).

__Note__: *To run tests in parallel or with a test framework like TestNG or Junit you will need to install a project management and comprehension tool like Maven or Ant, as we'll explain in the section on Running Tests in Parallel.*

##Code Example	

Now let’s take a look at some simple Java code that verifies the title of the Amazon.com home page. This example test doesn't have all the features we'd like, but it contains all the basics required to run an automated test on Sauce Labs:

```java
import org.openqa.selenium.By;
import org.openqa.selenium.Platform;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.openqa.selenium.remote.CapabilityType;

import java.net.URL;

public class SampleSauceTest {

/**
* Creates an authentication instance using the supplied user name/access key.
*/

  public static final String USERNAME = System.getenv("SAUCE_USERNAME");
  public static final String ACCESS_KEY = System.getenv("SAUCE_ACCESS_KEY");
  public static final String URL = "http://" + USERNAME + ":" + ACCESS_KEY + "@ondemand.saucelabs.com:80/wd/hub";

/**
* Represents the browser type, version, and operating system to be used as part * of the test run.
*/

  public static void main(String[] args) throws Exception {

    DesiredCapabilities caps = new DesiredCapabilities();
    caps.setCapability(CapabilityType.BROWSER_NAME, "internet explorer");
    caps.setCapability(CapabilityType.VERSION, "11");
    caps.setCapability(CapabilityType.PLATFORM, "Windows 7");
    caps.setCapability("name", "Sauce Sample Test");

/**
* Runs a simple test verifying the title of the amazon.com homepage.
*/

    WebDriver driver = new RemoteWebDriver(new URL(URL), caps);
    driver.get("http://www.amazon.com");
    WebElement element = driver.findElement(By.name("field-keywords"));

    element.sendKeys("Sauce Labs");
    element.submit();

    System.out.println(driver.getTitle());
    driver.quit();

  }
}

```
You can use these commands to compile and run a Java test class. The javac command is used to compile the Java test class and create a .class file. For instance, the SampleSauceTest.java references all of the dependent Jar files as shown in this code sample.

```
javac -cp ".:./selenium-2.46.0/selenium-java-2.46.0.jar:./selenium-2.46.0/selenium-java-2.46.0-srcs.jar:./selenium-2.46.0/libs/*" SampleSauceTest.java
```

__Note:__ *You need to specify the correct path for each Jar file in accordance to your system’s file structure.*

The java command is used to run the compiled .class file.

```
java -cp ".:./selenium-2.46.0/selenium-java-2.46.0.jar:./selenium-2.46.0/selenium-java-2.46.0-srcs.jar:./selenium-2.46.0/libs/*" SampleSauceTest
```

##Running Tests on Sauce
Now let’s take a closer look at the code so you can begin writing tests, or set your existing tests to run on Sauce Labs.

To run Selenium locally, you might initiate a driver instance for the browser that you want to test on as shown in the sample below:

```java
WebDriver driver = new FirefoxDriver()
```
To run on Sauce, you need to use the general RemoteWebDriver instance instead of the browser-specific FirefoxDriver instance. You must then pass two parameters: a URL that points your test to Sauce Labs and DesiredCapabilities.

__Note:__ *RemoteWebDriver is a standard Selenium interface. It allows you to perform all tests that you could do with a local Selenium test. The only difference is the URL that makes the test run using a browser on Sauce Labs' server.*

Here is a full look at the setup component of our test:

```java
public class SampleSauceTest {

/**
* Creates an authentication instance using the supplied user name/access key.
*/

  public static final String USERNAME = System.getenv("SAUCE_USERNAME");
  public static final String ACCESS_KEY = System.getenv("SAUCE_ACCESS_KEY");
  public static final String URL = "http://" + USERNAME + ":" + ACCESS_KEY + "@ondemand.saucelabs.com:80/wd/hub";

/**
* Represents the browser type, version, and operating system to be used as part * of the test run.
*/

  public static void main(String[] args) throws Exception {

    DesiredCapabilities caps = new DesiredCapabilities();
    caps.setCapability(CapabilityType.BROWSER_NAME, "internet explorer");
    caps.setCapability(CapabilityType.VERSION, "11");
    caps.setCapability(CapabilityType.PLATFORM, "Windows 7");
    caps.setCapability("name", "Sauce Sample Test");

/**
* Runs a simple test verifying the title of the amazon.com homepage.
*/

    WebDriver driver = new RemoteWebDriver(new URL(URL), caps);

```
####One: Pointing Tests to Run on Sauce

The first thing you need to send to RemoteWebdriver is a URL that authorizes your tests to be run on a browser in the Sauce Labs cloud. The URL contains your Sauce username and access key, which can be found on your account page.

####Two: Testing on Different Platforms
The next thing we need to do is provide the test with information about what kind of platform(s) we want to run tests on.  

Because we are not specifying FirefoxDriver() as we were in our local test, we must use DesiredCapabilities to specify what browser/OS combination(s) to spin up and execute against. DesiredCapabilities is a set of parameters and values that are sent to the Selenium server running in the Sauce Labs cloud. The parameters and values tell the Selenium server the specifications of the automated test that you plan to run. You can use the [Platforms Configurator](https://docs.saucelabs.com/reference/platforms-configurator/) to easily determine the correct DesiredCapabilities for your test.

You can  define the DesiredCapabilities as shown in this example:
```java
DesiredCapabilities caps = new DesiredCapabilities();
    caps.setCapability(CapabilityType.BROWSER_NAME, "Mozilla Firefox");
    caps.setCapability(CapabilityType.VERSION, " 38.0.5");
    caps.setCapability(CapabilityType.PLATFORM, "Windows 8.1");
    caps.setCapability("name", "Sauce Sample Test");

```

##Running Tests Against local Applications
If your test application is not publicly available, you will need to use Sauce Connect so that Sauce can reach it. 

Sauce Connect is a tunneling app that allows you to execute tests securely when testing behind firewalls or on localhost. For more detailed information, please see the [Sauce Connect docs](https://docs.saucelabs.com/reference/sauce-connect/). 

##Running Tests in Parallel
Now that you are running tests on Sauce, you may wonder how you can run your tests faster. One way to increase speed is by running tests in parallel across multiple virtual machines.

Most Java users use one of two popular third party testing frameworks: TestNG or Junit. These links are for two example projects written in each. They are designed to run in parallel. You can clone them and add your own test cases if you want:

1. https://github.com/ndmanvar/SeleniumJavaJunit
2. https://github.com/ndmanvar/SeleniumJavaTestNG

__Note:__ *Tests can be run in parallel at two levels, you can run your tests in parallel and you can run your tests in parallel across multiple browsers. For example, if you have 10 tests and run them serially on five browsers, this would be parallelism of five. You can also run tests across browsers and each test in parallel. Using our previous example, this would be 50 parallel tests (10 tests * 5 browsers). This requires that your tests are written in a way that they do not collide with one another. For more on this see Selenium WebDriver - [Running Your Tests in Parallel blog](https://saucelabs.com/selenium/selenium-webdriver).*

####Maven, Pom and Dependencies
The first thing we need to do regardless of which framework we have chosen is to install a project management tool like [Maven](https://maven.apache.org/download.cgi) or Ant. We'll use Maven in this tutorial. 

Once Maven is installed we need to update the POM (Project Object Model), an XML file used in Maven. The pom.xml stores information about a project like configuration details used during the build process.

The dependency list is the most important feature of a POM. Almost every project depends upon other pieces to build and run the project correctly. The dependency list is used to download and link the compiled dependencies.

Three dependencies are required to run tests in parallel on Sauce:

######1. Selenium Dependency 
The first thing you need to do is add the following common Selenium dependency to the pom.xml file regardless of the framework you are using to write your tests. This tells your project which version of Selenium to use.

```xml
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>2.45.0</version>
    <scope>test</scope>
</dependency>

```

######2. Framework Dependency
This is where you specify whether your are using Junit or TestNG as your framework. 

For Junit: 

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
</dependency>

```
For TestNG:
```xml
<dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>6.1.1</version>
    <scope>test</scope>
</dependency>
```

######3. Java Helper Library
The Java Helper Library handles reporting between your tests and Sauce Labs. It allows you to see useful information like the test name and pass/fail status in your Sauce Labs [Dashboard](https://saucelabs.com/beta/dashboard).

For Junit:
```xml
<dependency>
    <groupId>com.saucelabs</groupId>
    <artifactId>sauce_junit</artifactId>
    <version>2.1.18</version>
    <scope>test</scope>
</dependency>
```

For TestNG:
```xml
<dependency>
    <groupId>com.saucelabs</groupId>
    <artifactId>sauce_testng</artifactId>
    <version>2.1.18</version>
    <scope>test</scope>
</dependency>
```

####Parallelizing Junit
The Java helper library includes a Parallelized class that creates a dynamic thread pool that holds each thread that is running a test.

######1. Parallelizing the WebDriverTest Class

As mentioned previously, the SampleSauceTest class demonstrates how to run its tests in parallel. The test is parallelized by specifying the different parameters to test with, in this case the browser and platform. Behind the scenes, the test framework creates a different instance of the test class for each set of parameters and runs them in parallel. The parameters are passed to the constructor so each instance customizes its behavior using those parameters.

In this example, we're parallelizing tests across different browsers on different platforms. Since testing an app in Firefox on Linux is independent of testing it in Chrome on Windows, we can safely run both tests in parallel. The static method browsersStrings() is annotated with com.saucelabs.junit.ConcurrentParameterized.Parameters, indicating it should be used to determine the parameters for each instance of the test. The method returns a LinkedList of parameters to use for each test instance's constructor. The SampleSauceTest constructor captures these parameters and setUp() uses them to configure the DesiredCapabilities.

For a full example, see: https://github.com/ndmanvar/SeleniumJavaJunit/blob/master/src/test/java/com/yourcompany/SampleSauceTest.java

######2. Setting a Parallelism Limit

To stop tests from timing out when you're already using all your Sauce Labs parallel slots, we need to limit the number of threads.

The Sauce Labs Parallelized JUnit runner we used above uses the junit.parallel.threads System property to control how many threads it runs. Simply update your pom.xml like so:

```xml
 <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.0</version>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.12.4</version>
                <configuration>
                    <parallel>classes</parallel>
                    <threadCount>20</threadCount>
                    <redirectTestOutputToFile>true</redirectTestOutputToFile>
                </configuration>
            </plugin>
        </plugins>
</build>

```
####Parallelizing TestNG
TestNG has built in support for running tests in parallel. All you need to do is set a parallelism limit in your pom.xml like so: 

```xml
<build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.0</version>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.12.4</version>
                <configuration>
                    <parallel>classes</parallel>
                    <threadCount>20</threadCount>
                    <redirectTestOutputToFile>true</redirectTestOutputToFile>
                </configuration>
            </plugin>
        </plugins>
</build>

```

Don't forget to match your thread count to your concurrency limit. 

##Reporting to the Sauce Labs Dashboard
Follow these best practices to enhance your user experience while using Sauce Labs with Java:

######1. Reporting Test Results
If you have installed the Java Helper Library for your chosen framework, then your test information (like test name and pass/fail status) will automatically show up in your Sauce [Dashboard](https://saucelabs.com/beta/dashboard). 

Here are instructions for [installing the Java Helper Library for TestNG and Junit](https://github.com/jsmoxon/JavaDocs#3-java-helper-library).

######2. Tagging Tests

Sauce Labs allows you to “tag” your tests with different labels and variables like build ID, name, or custom. Sauce uses DesiredCapabilities to do this.

You can use assign build number/tag(s) to search or identify the test result with a particular build number or tag(s) once your test is complete. You can define the build number/tag(s) while defining the DesiredCapabilities as shown in the sample below:

```java
@BeforeMethod
    public void setUp() throws Exception {
        DesiredCapabilities capabilities = DesiredCapabilities.firefox();
        capabilities.setCapability("version", "17");
        capabilities.setCapability("platform", Platform.XP);
        capabilities.setCapability("name", "Web Driver demo Test");
        capabilities.setCapability("tags", "Tag1");
        capabilities.setCapability("build", "v1.0");
        this.driver = new RemoteWebDriver(
                new URL("http://" + authentication.getUsername() + ":" + authentication.getAccessKey() + "@ondemand.saucelabs.com:80/wd/hub"),
                capabilities);
        driver.get("http://tutorialapp.saucelabs.com");
    }
```

Assigning build numbers in your tests allows you to group your tests on the Sauce Labs Dashboard by builds as shown in the image below:

LINK TO PHOTO

Assigning tag(s) in your tests allows you to easily search for test results on the Sauce Labs Archives page as shown in the image below:

LINK TO PHOTO
