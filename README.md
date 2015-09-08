# Feature Toggle in Java
In our project, we choose [togglz](http://www.togglz.org/) as a framework for feature toggle.
## import togglz in project
We need to add these dependencies in project file `build.gradle`
```shell
  compile 'org.togglz:togglz-core:2.2.0.Final'
  compile 'org.togglz:togglz-servlet:2.2.0.Final'
  compile 'org.togglz:togglz-jsp:2.2.0.Final' //support jsp
  compile 'org.togglz:togglz-console:2.2.0.Final'  //support togglz console
  compile 'org.togglz:togglz-spring-security:2.2.0.Final' //support user authentication

  testCompile 'org.togglz:togglz-testing:2.2.0.Final' //support togglz test
  testCompile 'org.togglz:togglz-junit:2.2.0.Final' //support togglz test
```
## config togglz in project
### Add a new class `ToggledFeature` implements Feature
```java
public enum ToggledFeature implements Feature {
    @Label("CONFIRM_PASSWORD")
    CONFIRM_PASSWORD,

    @Label("ADDRESS")
    ADDRESS,

    @Label("AGREE_TERMS")
    AGREE_TERMS,

    @Label("EMAIL_CONFIRMATION")
    EMAIL_CONFIRMATION;

    public boolean isActive() {
        return FeatureContext.getFeatureManager().isActive(this);
    }
}
```
### Add a new class `ToggledFeatureConfiguration`  implements TogglzConfig
```java
public class ToggledFeatureConfiguration implements TogglzConfig{
    public Class<? extends Feature> getFeatureClass() {
        return ToggledFeature.class;
    }

    public StateRepository getStateRepository() {
        ClassLoader classLoader = getClass().getClassLoader();
        File file = new File(classLoader.getResource("features.properties").getFile());
        return new FileBasedStateRepository(file);
    }

    @Override
    public UserProvider getUserProvider() {
        return new SpringSecurityUserProvider("ROLE_ADMIN") {
        };
    }

}
```
### Add a `features.peoperties` file to set togglz status
```shell
CONFIRM_PASSWORD=true
ADDRESS=true
AGREE_TERMS=true
EMAIL_CONFIRMATION=true
```
### Then config togglz in `web.xml`
```xml
<context-param>
       <param-name>org.togglz.core.manager.TogglzConfig</param-name>
       <param-value>com.trailblazers.freewheelers.utils.ToggledFeature.ToggledFeatureConfiguration</param-value>
   </context-param>

   <filter>
       <filter-name>TogglzFilter</filter-name>
       <filter-class>org.togglz.servlet.TogglzFilter</filter-class>
   </filter>
   <filter-mapping>
       <filter-name>TogglzFilter</filter-name>
       <url-pattern>/*</url-pattern>
   </filter-mapping>
```
### Now we can use `<togglz:feature>` in JSP file
* First, we need to add togglz tags to jsp file
```html
<%@ taglib prefix="togglz" uri="http://togglz.org/taglib"%>
```

* Then we can use togglz to hide elements
```html
    <togglz:feature name="AGREE_TERMS">
        <div>

        </div>
    </togglz:feature>
```

## How to use togglz in Unit Test
```java
public class ToggledFeaturesTest {

    @Rule
    public TogglzRule togglzRule = TogglzRule.allEnabled(ToggledFeature.class);

    @Test
    public void testToggleFeature() {

        assertTrue(ToggledFeature.CONFIRM_PASSWORD.isActive());

        togglzRule.disable(ToggledFeature.CONFIRM_PASSWORD);
        assertFalse(ToggledFeature.CONFIRM_PASSWORD.isActive());
    }

    @After
    public void tearDown() throws Exception {
        togglzRule.disableAll();
    }
}
```

## How to show toggle console in browser
* First, we need to add toggle servlet in `web.xml`
```xml
    <servlet>
        <servlet-name>TogglzConsoleServlet</servlet-name>
        <servlet-class>org.togglz.console.TogglzConsoleServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>TogglzConsoleServlet</servlet-name>
        <url-pattern>/togglz/*</url-pattern>
    </servlet-mapping>
```
* Then login with admin role and open (http://localhost:8080/togglz/
* Now we can use this page to toggle feature
  ![Image of Togglz console](https://github.com/sighWang/feature-toggle-in-java/blob/master/togglz-console.png)
