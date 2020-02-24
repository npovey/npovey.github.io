---

layout: post
title:  "Eclipse Project to Query PostgreSQL and Output Results on a Website"
date:   2020-2-23 14:34:33 -0700
categories: update
---


#### Eclipse Project to Query PostgreSQL and Output Results on a Website

The idea is to create a website as the one below.

![idea](/2020-02-23-eclipse_web_server/idea.png)

##### Step 1: Setting up Spark with Gradle

Follow this tutorial: <http://sparkjava.com/tutorials/gradle-setup>



![eclipse_main](/2020-02-23-eclipse_web_server/eclipse_main.png)



![eclipse_sql](/2020-02-23-eclipse_web_server/eclipse_sql.png)



##### Step 2: Install PostgreSQL

Follow the following link and install PostgreSQL<https://www.postgresql.org/download/>

##### Step 3: Add data to database

Manually added countries. (Future work do it programatically)

```txt
China:76943
Diamond Princess:691
South Korea:602
Italy:157
..
```

![postgre](/2020-02-23-eclipse_web_server/postgre.png)



![insert](/2020-02-23-eclipse_web_server/insert.png)



##### Step 4: Add end point /hello to the main function

This endpoint reads inputs to the form and gets the request

```java
        get("/hello", (req, res) -> "Enter country name"
        		+ " <form method = post>\n" +
        		"\n" +
        		"  First name:<br>\n" +
        		"  <input type=\"text\" name=\"country\">\n" +
        		"  <br>\n" +
        		"  <input type=\"submit\" value=\"Submit\">\n" +
        		"  \n" +
        		"</form>");
        post("/hello", (request, response) -> model.getCasesCount(request.queryParams("country"))+"");
```

##### Step 5: Create classes

The following 4 classes needed to create our webserver

```bash
Country.java                                
Sql2oModel.java                                 
Main.java                                      
Model.java                                     
```

###### Main.java

```java
package crv19;
import static spark.Spark.*;
import java.util.UUID;
import org.sql2o.Sql2o;
import org.sql2o.converters.UUIDConverter;
import org.sql2o.quirks.PostgresQuirks;
public class Main {
    public static void main(String[] args) {
//        Sql2o sql2o = new Sql2o("jdbc:postgresql://" + options.dbHost + ":" + options.dbPort + "/" + options.database,

        Sql2o sql2o = new Sql2o("jdbc:postgresql://" + "localhost" + ":" + "5432" + "/" + "crv19",
        		"blog_owner", "np", new PostgresQuirks() {
            {
                // make sure we use default UUID converter.
                converters.put(UUID.class, new UUIDConverter());
            }
        });
        Model model = new Sql2oModel(sql2o);
        get("/hello", (req, res) -> "Enter country name"
        		+ " <form method = post>\n" +
        		"\n" +
        		"  First name:<br>\n" +
        		"  <input type=\"text\" name=\"country\">\n" +
        		"  <br>\n" +
        		"  <input type=\"submit\" value=\"Submit\">\n" +
        		"  \n" +
        		"</form>");
        post("/hello", (request, response) -> model.getCasesCount(request.queryParams("country"))+"");    
    }
}
```

###### Country.java

```java
package crv19;
public class Country {
	private String name;
	private int count;
	public int getCount() {
		return count;
	}
	public void setCount(int cases) {
		this.count = cases;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
}
```

Model.java

```java

package crv19;
public interface Model {
	int getCasesCount(String countryName);
}
```

Sql2oModel.java

```java
package crv19;
import java.util.List;
import org.sql2o.Connection;
import org.sql2o.Sql2o;
public class Sql2oModel implements Model {
    private Sql2o sql2o;
    public Sql2oModel(Sql2o sql2o) {
        this.sql2o = sql2o;
    }

    @Override
    public int getCasesCount(String countryName)  {
        try (Connection conn = sql2o.open()) {
            List<Country> countries = conn.createQuery("select * from countries where name=:name ")
            		.addParameter("name", countryName)
                    .executeAndFetch(Country.class);
//            countries.forEach((post) -> post.setCategories(getCategoriesFor(conn, post.getPost_uuid())));
           if(countries.size()== 0) {
        	   return 0;
           } else {
        	   return countries.get(0).getCount();
           }
        }
    }
}
```

##### Step 6: Test the server

Connect to localhost with port number 4567. Port number is hardcoded.

Step1:

![idea](/2020-02-23-eclipse_web_server/idea.png)

Step2: Query the PosgreSQL

![query_entry](/2020-02-23-eclipse_web_server/query_entry.png)

Step3: Output is shown below. The data was entered for Feb 23, 2020,

![ans](/2020-02-23-eclipse_web_server/ans.png)

#### Publish project to AWS

##### Step 7: Copy projet to AWS

```bash
(base) nps-MacBook-Air-2:~ np$ scp -i ~/.ssh/2019.pem -r ~/eclipse-workspace/crv19_new ec2-user@ec2-52-41-181-26.us-west-2.compute.amazonaws.com:~/.
Library.class                                  100%  330    14.6KB/s   00:00    
Model.class                                    100%  145     7.8KB/s   00:00    
Main$1.class                                   100%  607    32.3KB/s   00:00    
Country.class                                  100%  718    34.3KB/s   00:00    
Sql2oModel.class                               100% 1589    74.0KB/s   00:00    
Main.class                                     100% 2373    99.5KB/s   00:00    
LibraryTest.class                              100%  630    24.9KB/s   00:00    
.classpath                                     100%  446    18.8KB/s   00:00    
gradle-wrapper.jar                             100%   53KB 246.8KB/s   00:00    
gradle-wrapper.properties                      100%  200     8.4KB/s   00:00    
gradlew                                        100% 5296   214.0KB/s   00:00    
.gitignore                                     100%   16     0.7KB/s   00:00    
org.eclipse.buildship.core.prefs               100%   54     2.4KB/s   00:00    
.project                                       100%  595    23.1KB/s   00:00    
build.gradle                                   100% 1289    67.9KB/s   00:00    
taskHistory.bin                                100%   74KB 273.2KB/s   00:00    
taskHistory.lock                               100%   17     0.7KB/s   00:00    
fileContent.lock                               100%   17     0.8KB/s   00:00    
annotation-processors.bin                      100%   18KB 301.5KB/s   00:00    
last-build.bin                                 100%    1     0.1KB/s   00:00    
fileHashes.lock                                100%   17     0.9KB/s   00:00    
fileHashes.bin                                 100%   21KB 277.4KB/s   00:00    
resourceHashesCache.bin                        100%   20KB 318.5KB/s   00:00    
cache.properties                               100%   49     2.6KB/s   00:00    
outputFiles.bin                                100%   18KB 287.1KB/s   00:00    
buildOutputCleanup.lock                        100%   17     0.8KB/s   00:00    
LibraryTest.class                              100%  630    29.2KB/s   00:00    
Library.class                                  100%  330    14.9KB/s   00:00    
Model.class                                    100%  145     7.7KB/s   00:00    
Main$1.class                                   100%  607    13.4KB/s   00:00    
Country.class                                  100% 1702    74.4KB/s   00:00    
Sql2oModel.class                               100% 1732    68.6KB/s   00:00    
Main.class                                     100% 1994    97.0KB/s   00:00    
crv19.jar                                      100% 4561   121.2KB/s   00:00    
results.bin                                    100%   58     2.5KB/s   00:00    
output.bin.idx                                 100%    1     0.1KB/s   00:00    
output.bin                                     100%    0     0.0KB/s   00:00    
TEST-LibraryTest.xml                           100%  385    20.1KB/s   00:00    
MANIFEST.MF                                    100%   25     1.0KB/s   00:00    
index.html                                     100% 2320    86.9KB/s   00:00    
base-style.css                                 100% 2645   115.2KB/s   00:00    
style.css                                      100% 1135    56.1KB/s   00:00    
LibraryTest.html                               100% 1915    74.6KB/s   00:00    
report.js                                      100% 5252   210.2KB/s   00:00    
default-package.html                           100% 1974    85.7KB/s   00:00    
gradlew.bat                                    100% 2260    82.2KB/s   00:00    
settings.gradle                                100%  578    26.5KB/s   00:00    
LibraryTest.java                               100%  360    19.4KB/s   00:00    
Library.java                                   100%  166     8.6KB/s   00:00    
Country.java                                   100%  293    12.8KB/s   00:00    
Sql2oModel.java                                100%  953    41.0KB/s   00:00    
Main.java                                      100% 1195    56.4KB/s   00:00    
Model.java                                     100%   83     3.8KB/s   00:00    
(base) nps-MacBook-Air-2:~ np$
```

##### Step 8: Install Gradle on AWS

```bash
future work
```

##### Step 9: Run project

```bash
future work
```

##### Step 10: Test

```bash
future work
```
