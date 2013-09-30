war-sec
=======

Sample WAR with Spring Security.

Versions
--------

* Spring `3.2.4.RELEASE`
* Spring Security `3.1.4.RELEASE`

Spring Logging
--------------

Spring's default is to use commons-logging which is a bit nasty.  The workaround, documented on Spring's own site, is to do the following.

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>3.2.4.RELEASE</version>
            <exclusions>
                <!-- We don't want commons-logging thanks -->
                <exclusion>
                    <groupId>commons-logging</groupId>
                    <artifactId>commons-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!-- This is what we have to do instead of commons-logging for Spring logging to work -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>1.7.0</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.0</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.0</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.14</version>
            <scope>runtime</scope>
        </dependency>

Jetty Plugin
------------

For simple development testing with `mvn jetty:run`

        <plugins>
            <!-- mvn jetty:run http://localhost:8080/spring-sec -->
            <plugin>
                <groupId>org.mortbay.jetty</groupId>
                <artifactId>maven-jetty-plugin</artifactId>
                <version>6.1.26</version>
            </plugin>
        </plugins>

WebLogic deployment
-------------------

For convenience of deployment to WebLogic, and to be on the safe side with class-loding policy.

		<!-- WebLogic shim.  Tested with WebLogic 11g (v 10.3.6) -->
		<weblogic-web-app xmlns="http://xmlns.oracle.com/weblogic/weblogic-web-app"
		    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		    xsi:schemaLocation="http://xmlns.oracle.com/weblogic/weblogic-web-app
		    http://xmlns.oracle.com/weblogic/weblogic-web-app/1.3/weblogic-web-app.xsd"
		    >
		    <description>Spring Security Sample</description>
		    <weblogic-version>10.3.6</weblogic-version>
		    <!-- Force policy of WAR class-loader first -->
		    <container-descriptor>
		        <prefer-web-inf-classes>true</prefer-web-inf-classes>
		    </container-descriptor>
		    <!-- Make WebLogic use a different session ID cookie name and path for this web application -->
		    <!-- Note, if this isn't done, bear in mind that the WebLogic console AND your app both use JSESSIONID by default -->
		    <!-- This causes confusion if debugging your app in a tab of the same browser as the WebLogic console -->
		    <!-- You apparently will be unable to log into your app because the cookie from the console is sent as well -->
		    <session-descriptor>
		        <cookie-name>spring-sec-session</cookie-name>
		        <cookie-path>spring-sec</cookie-path>
		    </session-descriptor>
		</weblogic-web-app>
		
Web App Descriptor
------------------

The web.xml provides the hooks necessary for Spring context loading, with a default config file of `WEB-INF/application.xml`.  It also sets up the 
Spring Security filter stack using the magic bean/filter name of `springSecurityFilterChain` which is defined by Spring Security at startup.

		<web-app>

		    <display-name>Spring Security Sample</display-name>

		    <!--
		       Spring security filter config
		         N.b. 'springSecurityFilterChain' is a magic bean name generated by Spring Security
		     -->
		    <filter>
		        <filter-name>springSecurityFilterChain</filter-name>
		        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
		    </filter>
		    <filter-mapping>
		        <filter-name>springSecurityFilterChain</filter-name>
		        <url-pattern>/*</url-pattern>
		    </filter-mapping>

		    <!-- Loads default Spring config file from WEB-INF/applicationContext.xml -->
		    <listener>
		        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
		    </listener>

		</web-app>
		
The Spring Context
------------------

The Spring Context, in `applicationContext.xml` defines the Spring Security beans using the Spring Security namespace.  It just uses a simple in-memory `userDetailsService` for password authentication.

		<beans:beans xmlns="http://www.springframework.org/schema/security"
		    xmlns:beans="http://www.springframework.org/schema/beans"
		    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		    xsi:schemaLocation="http://www.springframework.org/schema/beans
		           http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
		           http://www.springframework.org/schema/security
		           http://www.springframework.org/schema/security/spring-security-3.1.xsd">

		    <http auto-config='true'>
		        <intercept-url pattern="/**" access="ROLE_USER" />
		    </http>

		    <user-service id="userDetailsService">
		        <user name="admin" password="password" authorities="ROLE_USER, ROLE_ADMIN" />
		        <user name="user" password="password" authorities="ROLE_USER" />
		    </user-service>

		    <authentication-manager alias="authenticationManager">
		        <authentication-provider user-service-ref="userDetailsService"/>
		    </authentication-manager>

		</beans:beans>











