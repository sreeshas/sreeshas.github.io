---
layout: post
title: How to use Mongolab's mongodb with Jhipster
---

Jhipster has many subgenerators to deploy applications to various cloud 
providers such as Heroku, CloudFoundry, OpenShift etc.
However, if the apps use mongodb they cannot be deployed to cloud providers as mongeez bundled 
with Jhipster requires admin privileges as discussed [here] (https://github.com/jhipster/generator-jhipster/issues/733) 

After searching for sometime i found out a workaround [here] (http://stackoverflow.com/questions/31014405/deploying-a-jhipster-mongodb-application-to-heroku)
but it was too confusing to follow and it did not work for me. 

I stumbled upon [mongobee](https://github.com/mongobee/mongobee) and decided to give it a try.

The following are the steps to replace mongeez with mongobee in jhipster and i verified this works with mongolab.


* <strong>Step 1</strong>:  Add mongobee dependency to pom.xml

    ```
              <dependency>
                  <groupId>com.github.mongobee</groupId>
                  <artifactId>mongobee</artifactId>
                  <version>0.10</version>
              </dependency>
    ```
* <strong>Step 2</strong>:  Edit DatabaseConfiguration.java to instantiate mongobee instead of mongeez

``` java
//    @Bean
//    public Mongeez mongeez() {
//        log.debug("Configuring Mongeez");
//        Mongeez mongeez = new Mongeez();
//        mongeez.setFile(new ClassPathResource("/config/mongeez/master.xml"));
//        mongeez.setMongo(mongo);
//        mongeez.setDbName(propertyResolver.getProperty("databaseName"));
//        mongeez.process();
//        return mongeez;
//    }
    
      @Bean
      public Mongobee mongobee(){
    
         Mongobee runner = 
             new Mongobee("mongodb://username:password@ds011111.mongolab.com:11111/tablename");  
         runner.setChangeLogsScanPackage(
             "com.application.config.DatabaseConfiguration"); // package to scan for changesets
         runner.setEnabled(true);
         return runner;
      }
```
    
* <strong>Step 3</strong>: Create Java classes for mongeez xml config files. 

    Mongobee lets you create java config files instead of xml. 
    Create java config files for users.xml and authorities.xml.

``` java
@ChangeLog(order = "001")
public class MyMongoChangeSet {
     
   @ChangeSet(order = "001", id = "ChangeSet-1", author = "jhipster")
   public void someChange(DB db) {
      DBCollection mycollection = db.getCollection("T_AUTHORITY");
      BasicDBObject doc1 = new BasicDBObject().append("_id", "ROLE_ADMIN");
      BasicDBObject doc2 = new BasicDBObject().append("_id", "ROLE_USER");
      mycollection .insert(doc1);
      mycollection .insert(doc2);
   }
   ```
          
Thats's pretty much it.

There's just one small caveat: I see some exceptions thrown at WARN level during application startup 
which seems to be benign at the moment. 

``` java
[WARN] org.reflections.Reflections - could not create Vfs.Dir from url. ignoring the exception and continuing
org.reflections.ReflectionsException: could not create Vfs.Dir from url, no matching UrlType was found [file:/System/Library/Java/Extensions/libAppleScriptEngine.jnilib]
either use fromURL(final URL url, final List<UrlType> urlTypes) or use the static setDefaultURLTypes(final List<UrlType> urlTypes) or addDefaultURLTypes(UrlType urlType) with your specialized UrlType.
	at org.reflections.vfs.Vfs.fromURL(Vfs.java:108) ~[reflections-0.9.9.jar:na]
	at org.reflections.vfs.Vfs.fromURL(Vfs.java:90) ~[reflections-0.9.9.jar:na]
	at org.reflections.Reflections.scan(Reflections.java:236) [reflections-0.9.9.jar:na]
	at org.reflections.Reflections.scan(Reflections.java:203) [reflections-0.9.9.jar:na]
	at org.reflections.Reflections.<init>(Reflections.java:128) [reflections-0.9.9.jar:na]
	at org.reflections.Reflections.<init>(Reflections.java:169) [reflections-0.9.9.jar:na]
	at org.reflections.Reflections.<init>(Reflections.java:142) [reflections-0.9.9.jar:na]
	at com.github.mongobee.utils.ChangeService.fetchChangeLogs(ChangeService.java:43) [mongobee-0.10.jar:na]
	at com.github.mongobee.Mongobee.execute(Mongobee.java:142) [mongobee-0.10.jar:na]
	at com.github.mongobee.Mongobee.afterPropertiesSet(Mongobee.java:117) [mongobee-0.10.jar:na]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1613) [spring-beans-4.0.7.RELEASE.jar:4.0.7.RELEASE]
          
```            
          





