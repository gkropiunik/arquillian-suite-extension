The Arquillian Suite Extension
==================================

[![Build Status](https://travis-ci.org/ingwarsw/arquillian-suite-extension.png?branch=master)](https://travis-ci.org/ingwarsw/arquillian-suite-extension)
[![Coverage Status](https://coveralls.io/repos/ingwarsw/arquillian-suite-extension/badge.png?branch=master)](https://coveralls.io/r/ingwarsw/arquillian-suite-extension?branch=master)
[![Maven Central](https://img.shields.io/maven-central/v/org.eu.ingwar.tools/arquillian-suite-extension.svg)](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.eu.ingwar.tools%22%20AND%20a%3A%22arquillian-suite-extension%22)

The Extension will force all Classes in a Module into a TestSuite running from the same DeploymentScenario.

Class marked with @ArquillianSuiteDeployment will be used as a 'template' for all other TestClass scenarios.

Deploy will occur only once on first test witch require Arquillian.

So far tested on:
- Jboss 7.1, Jboss 7.2
- EAP 6.1, EAP 6.2
- Wildfly 8.0, Wildfly 10.0
- Glassfish 3.2.2
- Should work on other servers too

From version 1.1.0 working with domain mode (see extended usage).

# Basic Usage

Add module to test classpath.

    <dependency>
        <groupId>org.eu.ingwar.tools</groupId>
        <artifactId>arquillian-suite-extension</artifactId>
        <version>1.1.3</version>
        <scope>test</scope>
    </dependency>

Mark one of your test classes with annotation @ArquillianSuiteDeployment along with usual @Deployment annotation on method.

    @ArquillianSuiteDeployment
    public class Deployments {

        @Deployment
        public static WebArchive deploy() {
            return ShrinkWrap.create(WebArchive.class)
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml")
                .addPackage(Deployments.class.getPackage());
        }
    }

Run test either from IDE or maven.

**Make sure you use servlet protocol!**

To do that add following to arquillian.xml:

    <defaultProtocol type="Servlet 3.0"/>

Or mark your deployments with annotation @OverProtocol("Servlet 3.0")

# Extended usage

## Getting SNAPSHOT versions of the plugin
If you really want to use **snapshots**, here's the repository they are deployed to.

```xml
<pluginRepositories>
    <pluginRepository>
        <id>sonatype-snapshots</id>
        <name>Sonatype Snapshots</name>
        <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
    </pluginRepository>
</pluginRepositories>
```

## Generic builder

That project contains generic builder with should work for most simple J2EE cases.
You dont need to define deployment yourself but you can use generic builder to do work for you.

Basic usage for EJB module looks like.

        @Deployment
        public static EnterpriseArchive generateAutogeneratedDeployment() {
          EnterpriseArchive ear = EarGenericBuilder.getModuleDeployment(ModuleType.EJB);
            return ear;
        }

## Multi deployment

As with normal arquillian you can define more than one deployment and then force tests to run on one of them.
You can run them in order you like adding order attribute to @Dployment annotation.

    @ArquillianSuiteDeployment
    public class Deployments {

        @Deployment(name = "normal", order = 2)
        public static Archive<?> generateDefaultDeployment() {
            return ShrinkWrap.create(WebArchive.class, "normal.war")
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml")
                .addClass(Deployments.class)
                .addClass(InjectedObject.class)
                .addPackage(Extension1Test.class.getPackage());
        }
    
        @Deployment(name = "extra", order = 1)
        public static Archive<?> generateExtraDeployment() {
            Archive<?> ejb = ShrinkWrap.create(WebArchive.class, "extra_ejb.war")
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml")
                .addClass(Deployments.class)
                .addClass(InjectedObject.class)
                .addPackage(ExtensionExtra1Test.class.getPackage());
            return ejb;
        }
    }
    
Then at test methods you need to add annotation @OperateOnDeployment("name")

    @Test
    @OperateOnDeployment("normal")
    public void testNormal() {
        // Test on normal deployment
    }
    
    @Test
    @OperateOnDeployment("extra")
    public void testExtra() {
        // Test on extra deployment
    }

## Domain deployment

To use domain to test classes you need to do few extra steps.
- Mark your deployment with @TargetsContainer("container-name")
- Change arquillian container to domain one.
- And add group for container in arquillian.xml

    <group qualifier="domain">
        <container qualifier="DomainController">
        </container>
    </group>

All of that + extra switching betweeen standalone/domain is done in that project itself.

# Easiest way to move to suite:

- Move all deployments to one class and make all your tests extend that class..
- If you have more than one deployment name them and add everywhere in your tests @OperateOnDeployment
- Check if everything is working
- Dont go further unless your tests are not working at that moment (probably will work much slower then normally)
- Add arquillian-suite as dependency
- Check if its working (should work slow as before)
- Add @ArquillianSuiteDeployment annotation on your deployment class
- Now check if your deployments deploy only once.

**That way if you will ever wonder if suite is failing your build only thing you will need to do is comment out @ArquillianSuiteDeployment and checks if its working then. If not its not suite related problem.**

# Extra info

## Credits

Most work was done by Aslak Knutsen
- https://gist.github.com/aslakknutsen/3975179
- Part was added by ItCrowd team.

I just mixed things up..

## Continuous integration

Travis CI builds the plugin with Oracle and Open JDK 7. All successfully built snapshots are deployed to
Sonatype OSS repository. Jacoco is used to gather coverage metrics and the report is submitted
to Coveralls.


## License

The project is licensed under the Apache License, Version 2.0.
