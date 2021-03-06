Moksaya :
======================================================================

Creating a restful interface to moksaya and the website/front end would be a client which consumes the RESTful Api itself 


###Setup:
    $virtualenv ../ENV
    $source ../ENV/bin/activate
    (ENV)$pip install -r requirements.txt
    (ENV)$fab setup start

   

###TODO:
	1: Implement Proper authorization Class to Apis  	   



###Authentication:
I am using the ApiKeyAuthentication class offered by the django-tastypie, so in order to set this up once you have setup the project login to 127.0.0.1:<port>/admin and  login with the super user you created while setting up the project.You will find Tastypie app with Api keys model.Select it and add api key for desired user.

       # Pass it as GET params	     
       curl http://127.0.0.1:8000/api/v1/profile/?username=<yourusername>\&api_key=<yourapikey>

Each user is supposed to get their own api key , which could be autogenerated and send in the request ,currently I have manually set the api_key for super user and I will use it to perform various operations.


Reffer Tastypie documentation to know more about it:
http://django-tastypie.readthedocs.org/en/latest/authentication.html#apikeyauthentication


###Authorization:
Tastypie seperates the authentication and authorization pretty neatly ,I am using DjangoAuthorization class from tastypie.authorization.This class provides the authenticated user to access the areas of the application for which they are authorized in django user permission.
 

###Test:
After setting up the project for the first time , you would need to register an user and get the api_key to access the resources.
You can also setup the superuser while setting up the project with ./manage.py syncdb or fab setup 	 

To Register a new user make the following curl request:
   
     $curl --dump-header - -H "Content-Type: application/json" -X POST --data '{"username":"spock","password":"notebook"}' http://127.0.0.1:8000/api/v1/register/ 

To get your api_key :

     $ curl -k --user "username:password" http://127.0.0.1:8000/api/v1/token/auth/

This should return a json resoponse like this :

     {
    "key": "600ca8c91d70376c7427854687d979ad97bb92ef"
     }       


Next thing you need to do is to edit thhe fabfile
     
     user = "spock"
     #Provide the username you used while setting up the project.
     key =  "600ca8c91d70376c7427854687d979ad97bb92ef"
     #Add the api_key that you createdd in the admin 

Now , this fabric script would test basic use cases for the project using curl.Fabric is included in the dependencies so just open a new terminal to the root of project:

     $source ../ENV/bin/activate
     (ENV)$fab test


Apart from fab test , you may try fab fabfile.py to get list of commands.
This test cases covers primariliy these Basic Operations:


     * POST request to populate the data base < fab PostTest >
     * GET  request to retrive all the accessible json data feed < fab GetTest >
     * PATCH requst  ,we can use POST and PATCH interchangeably they and I find PATCH more comfortable thant PUT < fab PatchTest >
     * DELETE request to simliy remove the entries from the database < fab clean > , this function performs the DELETE operation on all the Resources ie : Users,Profiles,Projects,Comments,Likes,Connections but leaving the admin or super user intact.



Uppon successful completation , you woul find a project forked to super users profile.
   


###Interacting with the APIs :

Okay , lets say first thing we want to do is to create a new user say 'Spock':  

       $curl --dump-header - -H "Content-Type: application/json" -X POST --data '{"username":"spock","password":"notebook"}' http://127.0.0.1:8000/api/v1/register/ 
		
Lets check the created user :


      	$ curl --dump-header - -H "Content-Type:application/json"-X http://127.0.0.1:8000/api/v1/user/spock/?username=spock\&api_key=<key>
	
	HTTP/1.0 200 OK
	Date: Thu, 11 Jul 2013 02:31:21 GMT
	Server: WSGIServer/0.1 Python/2.7.3
	Vary: Accept, Accept-Language, Cookie
	X-Frame-Options: SAMEORIGIN
	Content-Type: application/json
	Content-Language: en-us
	Cache-Control: no-cache

	{
	"date_joined": "2013-07-10T21:30:26.927720", 
	"first_name": "", 
	"id": 3, 
	"last_login": "2013-07-10T21:30:26.927712", 
	"last_name": "", 
	"resource_uri": "/api/v1/user/3/", 
	"username": "spock"
	}


Now we have a new user created , now lets create a user profile for the Spock: 

       $ curl --dump-header - -H "Content-Type:application/json" -X POST --data '{"user":"/api/v1/user/3/" , "about_me":"Hello I am Spock" }' http://127.0.0.1:8000/api/v1/profile/?username=aregee\&api_key=notebook

       HTTP/1.0 201 CREATED
       Date: Thu, 11 Jul 2013 02:35:17 GMT
       Server: WSGIServer/0.1 Python/2.7.3
       Vary: Accept, Accept-Language, Cookie
       X-Frame-Options: SAMEORIGIN
       Content-Type: text/html; charset=utf-8
       Location: http://127.0.0.1:8000/api/v1/profile/3/
       Content-Language: en-us

Now simply we can access the spock's profile and prjects at the above generated url: 

      $ curl http://127.0.0.1:8000/api/v1/profile/3/?username=aregee\&api_key=notebook
 	   

      {
      "about_me": "Hello I am Spock", 
      "friends": [], 
      "id": 3, 
      "language": "en", 
      "location": "", 
      "mugshot": null, 
      "privacy": "registered", 
      "projects": [], 
      "resource_uri": "/api/v1/profile/3/", 
      "user": "spock", 
      "website": ""
      }
 


Spock has no projects , lets add a new project to his profile 

       $ curl --dump-header curl -F "user=/api/v1/profile/2/" -F "title=Fiddle with JS" -F "desc=this file documents my PROGRESS with learning JavaScript" -F "src=@projects/objects.js" -F "screenshot=@projects/img_screen.png" http://127.0.0.1:8000/api/v1/projects/?username=aregee\&api_key=notebook

       HTTP/1.0 201 CREATED
       Date: Thu, 11 Jul 2013 02:45:34 GMT
       Server: WSGIServer/0.1 Python/2.7.3
       Vary: Accept, Accept-Language, Cookie
       X-Frame-Options: SAMEORIGIN
       Content-Type: text/html; charset=utf-8
       Location: http://127.0.0.1:8000/api/v1/projects/6/
       Content-Language: en-us

      
Now lets check spocks profile: 

       $  curl http://127.0.0.1:8000/api/v1/profile/3/?username=aregee\&api_key=notebook

       {
       "about_me": "Hello I am Spock", 
       "friends": [], 
       "id": 3, 
       "language": "en", 
       "location": "", 
       "mugshot": null, 
       "privacy": "registered", 
       "projects": [
      {
            "Likes": 0,
            "comment": [],
            "desc": "this file documents my PROGRESS with learning JavaScript",
            "history": "",
            "id": 7,
            "resource_uri": "/api/v1/projects/7/",
            "screenshot": "/media/projects/img_screen_10.png",
            "shared_date": "2013-07-26T19:25:09.538939",
            "src": "/media/projects/objects_4.js",
            "title": "Fiddle with JS",
            "user": "spock"
        }
       ], 
       "resource_uri": "/api/v1/profile/3/", 
       "user": "spock", 
       "website": ""
       }
       
We can also access the Projects directly , here for Spock:
   
       $ curl http://127.0.0.1:8000/api/v1/projects/6/?username=aregee\&api_key=notebook
       {
            "Likes": 0,
            "comment": [],
            "desc": "this file documents my PROGRESS with learning JavaScript",
            "history": "",
            "id": 7,
            "resource_uri": "/api/v1/projects/7/",
            "screenshot": "/media/projects/img_screen_10.png",
            "shared_date": "2013-07-26T19:25:09.538939",
            "src": "/media/projects/objects_4.js",
            "title": "Fiddle with JS",
            "user": "spock"
        }

I am logged in as aregee , and I would like to fork Spock's work ;)

  
	$ curl http://127.0.0.1:8000/api/v1/forking/7/?username=aregee\&api_key=notebook

	{
	"desc": "Spock created objects with javascript", 
	"history": "project Spocks first Project  created by spock forked by aregee ", 
	"resource_uri": "/api/v1/forking/6/", 
	"screenshot": "/home/aregee/img_screen.jpg", 
	"shared_date": "2013-07-10T21:45:34.469755", 
	"src": "/media/projects/objects.js", 
	"title": "Spocks first Project"
	}
	

	$ curl http://127.0.0.1:8000/api/v1/profile/1/?username=aregee\&api_key=notebook

	{
	"about_me": "", 
	"friends": [], 
	"id": 1, 
	"language": "en", 
	"location": "", 
	"mugshot": null, 
	"privacy": "registered", 
	"projects": [
        {
        "Likes": 0, 
        "comment": [
        {
        "entry": "ProjectAPIS", 
        "resource_uri": "", 
        "text": "concky"
        }
        ], 
        "desc": "Posting from RESTfulapi", 
        "history": "", 
        "id": 1, 
        "owner": "aregee", 
        "resource_uri": "/api/v1/projects/1/", 
        "screenshot": null, 
        "shared_date": "2013-07-10T19:27:52.005761", 
        "src": "/media/%40/home/aregee/file_rem.py", 
        "title": "ProjectAPIS", 
        "user": "/api/v1/profile/1/"
        }, 
        {
        "Likes": 0, 
        "comment": [], 
        "desc": "Spock created objects with javascript", 
        "history": "project Spocks first Project  created by spock forked by aregee ", 
        "id": 7, 
        "owner": "aregee", 
        "resource_uri": "/api/v1/projects/7/", 
        "screenshot": "/home/aregee/img_screen.jpg", 
        "shared_date": "2013-07-10T21:54:16.320153", 
        "src": "/media/projects/objects.js", 
        "title": "Spocks first Project", 
        "user": "/api/v1/profile/1/"
        }
	], 
	"resource_uri": "/api/v1/profile/1/", 
	"user": "aregee", 
	"website": ""
	}



Lets delete Spock's only project:
     
     $ curl --dump-header - -H "Content-Type:application/json" -X DELETE http://127.0.0.1:8000/api/v1/projects/7/?username=aregee\&api_key=notebook 
     HTTP/1.0 204 NO CONTENT
     Date: Thu, 11 Jul 2013 03:38:22 GMT
     Server: WSGIServer/0.1 Python/2.7.3
     Vary: Accept, Accept-Language, Cookie
     X-Frame-Options: SAMEORIGIN
     Content-Type: text/html; charset=utf-8
     Content-Length: 0
     Content-Language: en-us


Now lets check spocks profile: 


       $ curl http://127.0.0.1:8000/api/v1/profile/3/?username=aregee\&api_key=notebook
 
 
       {
       "about_me": "Hello I am Spock", 
       "friends": [], 
       "id": 3, 
       "language": "en", 
       "location": "", 
       "mugshot": null, 
       "privacy": "registered", 
       "projects": [], 
       "resource_uri": "/api/v1/profile/3/", 
       "user": "spock", 
       "website": ""
       }


##How to add Likes to a Project : 
    
       $curl --dump-header - -H "Content-Type:application/json" -X POST --data '{"user":"/api/v1/profile/2/" ,"liked_content_type":"/api/v1/projects/2/" }' http://127.0.0.1:8000/api/v1/liking/?username=aregee\&api_key=notebook

       $ curl --dump-header - -H "Content-Type:application/json" -X GET http://127.0.0.1:8000/api/v1/liking/?username=aregee\&api_key=notebook

       {
       "meta": {
       "limit": 20,
       "next": null,
       "offset": 0,
       "previous": null,
       "total_count": 2
       },
       "objects": [
       {
       "Liked": "Looping for a While ",
       "id": 1,
       "liked_content_type": "/api/v1/projects/2/",
       "resource_uri": "/api/v1/liking/1/",
       "shared_date": "2013-07-26T19:33:48.322368",
       "user": "spock"
       },
       {
       "Liked": "Looping for a While ",
       "id": 2,
       "liked_content_type": "/api/v1/projects/2/",
       "resource_uri": "/api/v1/liking/2/",
       "shared_date": "2013-07-26T19:33:48.785608",
       "user": "aregee"
       }
       ]
       }

We can also use the Resource_id endpoint in the APIs to access the liked content and LikeResource allows POST,DELETE and GET options.
 

Lets GET the project that is being liked:

       $ curl --dump-header - -H "Content-Type:application/json" -X GET http://127.0.0.1:8000/api/v1/projects/2/?username=aregee\&api_key=notebook      

       {
            "Likes": 2,
            "comment": [],
            "desc": "It been a while and I know how to iterate",
            "history": "",
            "id": 2,
            "resource_uri": "/api/v1/projects/2/",
            "screenshot": "/media/projects/img_screen_3.png",
            "shared_date": "2013-07-26T19:32:56.988147",
            "src": "/media/projects/while_1.js",
            "title": "Looping for a While ",
            "user": "uhura"
        }




##User followership functionality 

To follow a particular user , we can make POST request like the one shown below where we pass the URI's of the follower and the followee:
   
	  $curl --dump-header - -H "Content-Type: application/json" -X POST --data '{"follower":"/api/v1/profile/2/","followee":"/api/v1/profile/1/"}'  http://127.0.0.1:8000/api/v1/relations/?username=aregee\&api_key=notebook

      	  $curl -X GET  http://127.0.0.1:8000/api/v1/relations/1/?username=aregee\&api_key=notebook
    
             {
            "created": "2013-07-26T19:50:06.934249",
            "followee": "/api/v1/profile/1/",
            "follower": "/api/v1/profile/2/",
            "followers": [
                "uhura"
            ],
            "following": [
                "aregee",
                "kirk"
            ],
            "id": 1,
            "user": "spock"
            }     



To unfollow a user , simply delete the above relation 

      $curl -X DELTE  http://127.0.0.1:8000/api/v1/relations/1/?username=aregee\&api_key=notebook


Here is how its reflected in Profiles  : 
 
      $curl -X GET http://127.0.0.1:8000/api/v1/profile/2/?username=aregee\&api_key=notebook

           {
            "about_me": "Hello I am Mister Spock",
            "followers": [
                "uhura"
            ],
            "following": [
                "aregee",
                "kirk"
            ],
            "id": 2,
            "language": "en",
            "location": "",
            "mugshot": null,
            "privacy": "registered",
            "projects": [
                {
                    "Likes": 0,
                    "comment": [],
                    "desc": "this file documents my PROGRESS with learning JavaScript",
                    "history": "",
                    "id": 1,
                    "resource_uri": "/api/v1/projects/1/",
                    "screenshot": "/media/projects/img_screen_2.png",
                    "shared_date": "2013-07-26T19:32:55.829584",
                    "src": "/media/projects/objects_1.js",
                    "title": "Fiddle with JS",
                    "user": "spock"
                }
            ],
            "resource_uri": "/api/v1/profile/2/",
            "user": "spock"
        },




#Comments :
 
How To POST : 

      $ curl --dump-header - -H "Content-Type:application/json" -X POST --data '{"user":"/api/v1/profile/1/","entry":"/api/v1/projects/2/" , "text":"Comment posted with REST" }' http://127.0.0.1:8000/api/v1/comment/?username=aregee\&api_key=notebook
     {
	    "id":1,
            "entry": "Looping for a While ",
            "resource_uri": "/api/v1/comment/1/",
            "text": "Comment posted with REST",
            "user": "aregee"
     }

     $curl --dump-header - -H "Content-Type:application/json" -X PATCH --data '{"user":"/api/v1/profile/1/","entry":"/api/v1/projects/2/" , "text":"Editing the Patched field and Comment posted with REST" }' http://127.0.0.1:8000/api/v1/comment/<id>/?username=aregee\&api_key=notebook

    $curl --dump-header - -H "Content-Type:application/json" -X DELETE ' http://127.0.0.1:8000/api/v1/comment/<id>/?username=aregee\&api_key=notebook





Lets delete Spock's Profile:

     $  curl --dump-header - -H "Content-Type:application/json" -X DELETE http://127.0.0.1:8000/api/v1/profile/2/?username=aregee\&api_key=notebook     ; curl http://127.0.0.1:8000/api/v1/profile/?username=aregee\&api_key=notebook


     {
     "meta": {
     "limit": 20, 
     "next": null, 
     "offset": 0, 
     "previous": null, 
     "total_count": 2
     }, 
     "objects": [
     {
     "about_me": "", 
     "friends": [], 
     "id": 1, 
     "language": "en", 
     "location": "", 
     "mugshot": null, 
     "privacy": "registered", 
     "projects": [.... ], 
     "resource_uri": "/api/v1/profile/1/", 
     "user": "aregee", 
     "website": ""
     }, 
     {
     "about_me": "I am sweet", 
     "friends": [], 
     "id": 2, 
     "language": "en", 
     "location": "", 
     "mugshot": null, 
     "privacy": "registered", 
     "projects": [...], 
     "resource_uri": "/api/v1/profile/2/", 
     "user": "vrinda", 
     "website": ""
     }
     ]
     }
       


Lets delete the Spock from registered user as well:

     $ curl --dump-header - -H "Content-Type:application/json" -X DELETE http://127.0.0.1:8000/api/v1/user/3/?username=aregee\&api_key=notebook;
     curl --dump-header - -H "Content-Type:application/json" http://127.0.0.1:8000/api/v1/user/
     


     {
     "meta": {
     "limit": 20, 
     "next": null, 
     "offset": 0, 
     "previous": null, 
     "total_count": 3
     }, 
     "objects": [
     {
     "date_joined": "2013-07-10T19:18:50.714252", 
     "first_name": "", 
     "id": -1, 
     "last_login": "2013-07-10T19:18:50.714234", 
     "last_name": "", 
     "resource_uri": "", 
     "username": "AnonymousUser"
     }, 
     {
     "date_joined": "2013-07-10T19:18:33.733717", 
     "first_name": "", 
     "id": 1, 
     "last_login": "2013-07-10T19:19:42.532263", 
     "last_name": "", 
     "resource_uri": "/api/v1/user/1/", 
     "username": "aregee"
     }, 
     {
     "date_joined": "2013-07-10T19:30:32.104176", 
     "first_name": "", 
     "id": 2, 
     "last_login": "2013-07-10T19:30:32.104167", 
     "last_name": "", 
     "resource_uri": "/api/v1/user/2/", 
     "username": "vrinda"
     }
     ]
     }




### Elasticsearch Backend :
   
This is an experimental functionality  I have been playing around for a while, If you would like to setup the elastic search get the server here <http://www.elasticsearch.org/download/> 
Furthermore I am using Django-haystack to integerate the elasticsearch with this project.You would have to specify Haystack Connection in the settings.py file : 

	    HAYSTACK_CONNECTIONS = {
	    'default': {
	    'ENGINE': 'haystack.backends.elasticsearch_backend.ElasticsearchSearchEngine',
	    'URL': 'http://127.0.0.1:9200/',# url to elasticsearch webserver 
	    'INDEX_NAME': 'haystack',
	    },
	    }
 
Usage:

	$(ENV)fab update_search # this maps to ./manage.py update_index command offered by haystack to push data into elasticsearch server

	$curl --dump-header - -H "Authorization: ApiKey aregee:notebook" -X GET http://127.0.0.1:8000/api/v1/projects/search/?q=python
	
	
	{
	"objects": [
        {
        "Likes": 0,
        "comment": [],
        "desc": "This python script changes extension of all the media files in the directory so they are not skipped in a media scan",
        "history": "project Gallery Lock   created by kirk forked by aregee ",
        "id": 6,
        "resource_uri": "/api/v1/projects/6/",
        "screenshot": "/media/projects/img_screen_7_1.png",
        "shared_date": "2013-07-26T20:26:20.275120",
        "src": "/media/projects/file_rem_1_1.py",
        "title": "Gallery Lock ",
        "user": "aregee"
        },
        {
        "Likes": 0,
        "comment": [],
        "desc": "This python script changes extension of all the media files in the directory so they are not skipped in a media scan",
        "history": "",
        "id": 3,
        "resource_uri": "/api/v1/projects/3/",
        "screenshot": "/media/projects/img_screen_7.png",
        "shared_date": "2013-07-26T20:26:22.787545",
        "src": "/media/projects/file_rem_1.py",
        "title": "Gallery Lock  ++ ",
        "user": "kirk"
        }
	]
	}
	
