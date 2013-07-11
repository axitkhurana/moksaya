Moksaya :
======================================================================

Creating a restful interface to moksaya and the website/front end would be a client which consumes the RESTful Api itself 


###Setup:
    $virtualenv ../ENV
    $source ../ENV/bin/activate
    $pip install -r requirements.txt
    $python manage.py syncdb
    $python manage.py migrate	
    $python manage.py runserver

###TODO:
	1: Write Test Cases 
	2: CRUD operation for likes 	   
	3: CRUD functionality for Friends
	4: Write Fabric Script to automate setup and deployment of the project 
	5: Hook up auto create api keys fucntionality 

###Authentication:
I am using the ApiKeyAuthentication class offered by the django-tastypie, so in order to set this up once you have setup the project login to 127.0.0.1:<port>/admin and  login with the super user you created while setting up the project.You will find Tastypie app with Api keys model.Select it and add api key for desired user.

       # Pass it as GET params	     
       curl http://127.0.0.1:8000/api/v1/profile/?username=<yourusername>\&api_key=<yourapikey>


Reffer Tastypie documentation to know more about it http://django-tastypie.readthedocs.org/en/latest/authentication.html#apikeyauthentication

###Interacting with the APIs :

Okay , lets say first thing we want to do is to create a new user say 'Spock'  
        $ curl --dump-header - -H "Content-Type: application/json" -X POST --data '{"username":"spock","password":"notebook"}' http://127.0.0.1:8000/api/v1/user/?username=aregee\&api_key=notebook
	
Lets check the created user :
      	$ curl --dump-header - -H "Content-Type:application/json"-X http://127.0.0.1:8000/api/v1/user/3/?username=aregee\&api_key=notebook
	

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


Now we have a new user created , now lets create a user profile for the Spock 

       $ curl --dump-header - -H "Content-Type:application/json" -X POST --data '{"user":"/api/v1/user/3/" , "about_me":"Hello I am Spock" }' http://127.0.0.1:8000/api/v1/profile/?username=aregee\&api_key=notebook

       HTTP/1.0 201 CREATED
       Date: Thu, 11 Jul 2013 02:35:17 GMT
       Server: WSGIServer/0.1 Python/2.7.3
       Vary: Accept, Accept-Language, Cookie
       X-Frame-Options: SAMEORIGIN
       Content-Type: text/html; charset=utf-8
       Location: http://127.0.0.1:8000/api/v1/profile/3/
       Content-Language: en-us

Now simply we can access the spock's profile and prjects at the above generated url 

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
       $ curl --dump-header - -H "Content-Type:application/json" -X POST --data '{"user":"/api/v1/profile/3/" ,"title":"Spocks first Project" , "desc":"Spock created objects with javascript" , "src":"projects/objects.js","screenshot":"/home/aregee/img_screen.jpg" }' http://127.0.0.1:8000/api/v1/projects/?username=aregee\&api_key=notebook

       HTTP/1.0 201 CREATED
       Date: Thu, 11 Jul 2013 02:45:34 GMT
       Server: WSGIServer/0.1 Python/2.7.3
       Vary: Accept, Accept-Language, Cookie
       X-Frame-Options: SAMEORIGIN
       Content-Type: text/html; charset=utf-8
       Location: http://127.0.0.1:8000/api/v1/projects/6/
       Content-Language: en-us

      
Now lets check spocks profile 
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
       "desc": "Spock created objects with javascript", 
       "history": "", 
       "id": 6, 
       "owner": "spock", 
       "resource_uri": "/api/v1/projects/6/", 
       "screenshot": "/home/aregee/img_screen.jpg", 
       "shared_date": "2013-07-10T21:45:34.469755", 
       "src": "/media/projects/objects.js", 
       "title": "Spocks first Project", 
       "user": "/api/v1/profile/3/"
       }
       ], 
       "resource_uri": "/api/v1/profile/3/", 
       "user": "spock", 
       "website": ""
       }
       
We can also access the Projects directly , here for Spock 
       $ curl http://127.0.0.1:8000/api/v1/projects/6/?username=aregee\&api_key=notebook
       {
       "Likes": 0, 
       "comment": [], 
       "desc": "Spock created objects with javascript", 
       "history": "", 
       "id": 6, 
       "owner": "spock", 
       "resource_uri": "/api/v1/projects/6/", 
       "screenshot": "/home/aregee/img_screen.jpg", 
       "shared_date": "2013-07-10T21:45:34.469755", 
       "src": "/media/projects/objects.js", 
       "title": "Spocks first Project", 
       "user": "/api/v1/profile/3/"
       } 

I am logged in as aregee , and I would like to fork Spock's work ;)

  
	$ curl http://127.0.0.1:8000/api/v1/forking/6/?username=aregee\&api_key=notebook

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



Lets delete Spock's only project 
     $  curl --dump-header - -H "Content-Type:application/json" -X DELETE http://127.0.0.1:8000/api/v1/projects/6/?username=aregee\&api_key=notebookHTTP/1.0 204 NO CONTENT


     Date: Thu, 11 Jul 2013 03:38:22 GMT
     Server: WSGIServer/0.1 Python/2.7.3
     Vary: Accept, Accept-Language, Cookie
     X-Frame-Options: SAMEORIGIN
     Content-Type: text/html; charset=utf-8
     Content-Length: 0
     Content-Language: en-us


Now lets check spocks profile 

       $  curl http://127.0.0.1:8000/api/v1/profile/3/?username=aregee\&api_key=notebook
 
 
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

Lets delete Spock's Profile 
     $  curl --dump-header - -H "Content-Type:application/json" -X DELETE http://127.0.0.1:8000/api/v1/profile/3/?username=aregee\&api_key=notebook ; curl http://127.0.0.1:8000/api/v1/profile/?username=aregee\&api_key=notebook


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
       


Lets delete the Spock from registered user as well 
     $ curl --dump-header - -H "Content-Type:application/json" -X DELETE http://127.0.0.1:8000/api/v1/user/3/?username=aregee\&api_key=notebook;curl --dump-header - -H "Content-Type:application/json" http://127.0.0.1:8000/api/v1/user/
     


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
