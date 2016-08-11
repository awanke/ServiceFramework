![logo](http://allwefantasy.com/service_framework_logo_big.jpg)

## Welcome To ServiceFramework

ServiceFramework is  a web-application framework that includes some powerfull Persistence Components 
like [ActiveORM](https://github.com/allwefantasy/active_orm) and [MongoMongo](https://github.com/allwefantasy/mongomongo)
written by  java language according to the Model-View-Controller(MVC) pattern.


### How to use ServiceFramework in Maven

add Dependency in your pom.xml:

        <dependency>
            <groupId>net.csdn</groupId>
            <artifactId>ServiceFramework</artifactId>
            <version>1.0</version>
        </dependency>

however,we realy recommend you clone source and using 'maven deploy -DskipTests' command to upload jar to your private maven repository.

make sure config/application.yml,config/logging.yml are present in your project root.


1.  ActiveORM in ServiceFramework like ActiveRecord in Rails,Awesome.
  
  
		    List<Tag> tags = Tag.where(map("name","java")).fetch;
   
2. Controller is total redesigned and some usefull functions are provied.

3. Most Object are managed by IOC.
  
			  @inject
			  Service service;
   
4. Easy to Test without Servlet container
  
	     @Test
	     public void search() throws Exception {
	         RestResponse response = get("/doc/blog/search", map(
	                 "tagNames", "_10,_9"
	         ));
	         Assert.assertTrue(response.status() == 200);
	         Page page = (Page) response.originContent();
	         Assert.assertTrue(page.getResult().size() > 0);
	     }

5. just a little configuration and Thrift & RESTFul are all supported
    
			 
		###############http config##################
		http:
		    port: 7700
		    disable: false

		thrift:
		    disable: false
		    services:
		        net_csdn_controller_thrift_impl_CLoadServiceImpl:
		           port: 7701
		           min_threads: 100
		           max_threads: 1000		        

		    servers:
		        load: ["127.0.0.1:7701"]

	  
6. Template Engine using Velocity. Instance Variables and Helper Class method will be automatically imported in page

	 
			    @At(path = "/hello", types = GET)
			    public void hello() {
			        render(200, map(
			                "name", "ServiceFramework"
			        ), ViewType.html);
			    }  


7. Dubbo is suppored now. You can do everything dubbo can in Serviceframework. RestProtocol as a new feature is also been  added to make invoking HTTP API like RPC.

			
			@At(path = "/say/hello", types = {RestRequest.Method.GET})
			    public void sayHello() {
			        render(200, "hello" + param("kitty"));
			}
			
			you can just type 'http://127.0.0.1/say/hello?kitty=wow' in Chrome ， 'hellokitty' will be retured.
			
		   However,maby you wanna invoke this HTTP API in your program. Just declare a interface like this:

					public interface TagController {
					    @At(path = "/say/hello", types = {RestRequest.Method.GET, RestRequest.Method.POST})
					    public HttpTransportService.SResponse sayHello(RestRequest.Method method, Map<String, String> params);
					
					    @At(path = "/say/hello", types = {RestRequest.Method.GET})
					    public HttpTransportService.SResponse sayHello3(@Param("kitty") String kitty);
					
					now,you can code like this:
					
					tagController.sayHello(RestRequest.Method.GET, WowCollections.map("kitty", "你好，太脑残")).getContent()
					
					or
					
					tagController.sayHello3("哇塞，天才呀").getContent()
					

8. Without Dubbo,you can also invoke HTTP API like RPC.

    * Suppose we have a Searcher, the rest API is something like '/v2/~/~/_search'。Create a new Interface in your project(Scala Example):
    
			   trait SearcherClient {
			  @At(path = Array("/v2/~/~/_search"), types = Array(GET, POST))
			  @BasicInfo(
			    desc = "索引服务",
			    state = State.alpha,
			    testParams = "",
			    testResult = "",
			    author = "WilliamZhu",
			    email = "allwefantasy@gmail.com"
			  )
			  def search(params: Map[String, String], content: String, method: net.csdn.modules.http.RestRequest.Method): java.util.List[HttpTransportService.SResponse]
			
			}


    *  Now you can create SearchClient instance like this (Please make sure you only create once.)(Scala Example):

			       val _searchClient = AggregateRestClient.buildClient[SearcherClient](hostAndPorts, new SearchEngineStrategy(), httpRequest)
			       
			       //SearchEngineStrategy is the Strategy how you invoke multi backend.

    * Now ,invoking like this(Scala Example)：			       			       
    
			
			val res = _searchClient.search(
			    url._2.toMap ++ Map("index" -> index, "type" -> ctype),
			    query,
			    RestRequest.Method.POST).searchResult




## QuickStart

Step 1 >   clone
 
 

	git clone https://github.com/allwefantasy/ServiceFramework
 
 
Step 2 >   import to your IDE
 
Step 3 >   modify config/application.yaml according to your DB Connection infomation. Notice that if you only use mysql you should disable mongodb.
  				
    datasources:
        mysql:
           host: 127.0.0.1
           port: 3306
           database: wow
           username: root
           password: root
           disable: false
        mongodb:
           host: 127.0.0.1
           port: 27017
           database: wow
           disable: false
        redis:
            host: 127.0.0.1
            port: 6379
            disable: true 		          
 
Step4 >   import sql/wow.sql into MySQL
 
Step5 >   create com.example.model.Tag class.

			public class Tag extends Model 
			{
			
			}

Step6 >   create com.example.controller.http.TagController 

          public class TagController extends ApplicationController 
			{
			   @At(path = "/hello", types = RestRequest.Method.GET)
			    public void hello() {
			        Tag tag = Tag.create(map("name","java"));
			        tag.save();
			        render(200, map(
			                "tag", tag
			        ), ViewType.html);
			    }
			}
			
Step7 >  create template/tag/hello.vm


			Hello $tag.name!  Hello  world!		

Step8 >   create startup class

    public class ExampleApplication {

    public static void main(String[] args) {
        ServiceFramwork.scanService.setLoader(ExampleApplication.class);
        Application.main(args);
    }
    }
    
Step9 >   run  ExampleApplication in your IDE

Step10 >  visit  http://127.0.0.1:9002/hello . Check your database, table tag will have one item.


Step11 >  Prepare for Test Unit . modify runner.DynamicSuite and add follow in  first line of initEnv method.

      ServiceFramwork.scanService.setLoader(ExampleApplication.class);

Step12 > create test class test.com.example.TagControllerTest

    public class TagControllerTest extends BaseControllerTest {
	    @Test
	    public void testHello() throws Exception {
	        Tag.deleteAll();
	        RestResponse response = get("/hello", map());
	        Assert.assertTrue(response.status() == 200);
	        String result = response.content();
	        Assert.assertEquals("Hello java!  Hello  world!", result);
	    }
    }

Step13 >  run DynamicSuiteRunner 

Step14 >   However,DynamicSuteRunner is not required。 You can also add following lines
to you test class and then you can use your IDE to test the class directly.

    static {
        initEnv(ExampleApplication.class);
    }

initEnv method promise that your container will be started propertly and right ClassLoader be
used.   




## Tutorial Usefull


* [Walk through for Creating Your first Application with ServiceFramework](https://github.com/allwefantasy/service_framework_example/blob/master/README.md)


## Step by Step tutorial
Step-by-Step-tutorial-for-ServiceFramework(continue...)

* [Step-by-Step-tutorial-for-ServiceFramework(1)](https://github.com/allwefantasy/service_framework_example/blob/master/README.md)
* [Step-by-Step-tutorial-for-ServiceFramework(2)](https://github.com/allwefantasy/service_framework_example/blob/master/doc/Step-by-Step-tutorial-for-ServiceFramework\(2\).md)
* [Step-by-Step-tutorial-for-ServiceFramework(3)](https://github.com/allwefantasy/service_framework_example/blob/master/doc/Step-by-Step-tutorial-for-ServiceFramework\(3\).md)
* [Step-by-Step-tutorial-for-ServiceFramework(4)](https://github.com/allwefantasy/service_framework_example/blob/master/doc/Step-by-Step-tutorial-for-ServiceFramework\(4\).md)


## Doc Links

* [Summary](https://github.com/allwefantasy/ServiceFramework/tree/master/doc/ServiceFrameworkWiki-start.md)
* [Model](https://github.com/allwefantasy/ServiceFramework/tree/master/doc/ServiceFrameworkWiki-model.md)
* [Controller](https://github.com/allwefantasy/ServiceFramework/tree/master/doc/ServiceFrameworkWiki-controller.md)
* [Test](https://github.com/allwefantasy/ServiceFramework/tree/master/doc/ServiceFrameworkWiki-test.md)


##  Some projects based on ServiceFramework

* [QuickSand](https://github.com/allwefantasy/QuickSand)


