# Python DRF file upload test


Summary
----------


Environment
----------
> build all and tested on GNU/Linux

    GNU/Linux: Ubuntu 20.04_x64 LTS
    Python: v3.8.10


Run
----------
```sh
$ pip3 install djangorestframework
$ pip3 install requests

$ mkdir test_file_upload
$ cd test_file_upload
$ django-admin startproject test_file_upload .
$ cd test_file_upload
$ django-admin startapp test_file_upload
$ cd ..
$ python3 manage.py migrate

$ python3 manage.py runserver localhost:8888
or
$ python3 manage.py runserver 0.0.0.0:8888


// test
// - upload path: test_file_upload/upload
$ curl -X POST -F "file=@./img.jpg" -F "file=@./img1.jpg" 127.0.0.1:8888/v1/file_upload
```



```sh
#
# settings.py
#

import os # for os.path.join(): 'NAME': BASE_DIR / 'db.sqlite3',

(REPLACE)
...
'NAME': BASE_DIR / 'db.sqlite3',
-> 'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),


ALLOWED_HOSTS = ["*"] # allow clients, "*" or "x.x.x.x", ...


// CORS
$ pip3 install django-cors-headers

(ADD)
INSTALLED_APPS = [
    ...,

    'corsheaders',
    'rest_framework',
]

(ADD)
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': (
        # No Web UI, JSON only
        'rest_framework.renderers.JSONRenderer',
    )
}

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',

    ...
]

# $ pip install django-cors-headers
#CORS_ORIGIN_ALLOW_ALL = True
CORS_ALLOWED_ORIGINS = [
    "http://localhost:8888",
    "http://127.0.0.1:8888",

    # Browser (127.0.0.1:9010) <-> Request 127.0.0.1:8888 (VirtualBox Network Port Forward) <-> 10.244.0.4:8888
    "http://127.0.0.1:9010",
]



#
# urls.py
#

...

from test_file_upload.test_file_upload import views

# file upload
urlpatterns = [
    ...,

    path( 'v1/file_upload', views.CUploadFile.as_view() ),
]



#
# views.py
#

from rest_framework.decorators import api_view
from rest_framework.response import Response
from django.http import HttpResponse
from django.http import JsonResponse

# for index page (/)
from django.views.decorators.csrf import csrf_exempt

import traceback
import datetime
import time
import math
import os
import json

# file upload
from rest_framework.views import APIView
#from rest_framework.parsers import FileUploadParser
from rest_framework import parsers
# SEE: https://stackoverflow.com/questions/59484753/how-to-read-or-save-a-file-from-multivaluedict-in-django
from django.core.files.storage import default_storage
from django.core.files.base import ContentFile


# test
# curl -X POST -F "file=@./img.jpg" -F "file=@./img1.jpg" 127.0.0.1:8888/v1/file_upload
#
# settings.py
#
# ADD:
#INSTALLED_APPS = [
#    ...,
#    'rest_framework',
#]
#
#REST_FRAMEWORK = {
#    'DEFAULT_RENDERER_CLASSES': (
#        # No Web UI, JSON only
#        'rest_framework.renderers.JSONRenderer',
#    )
#}
#
class CUploadFile(APIView):
    parser_classes = (parsers.FormParser, parsers.MultiPartParser)

    def post(self, request):
        try:
            _date = datetime.datetime.now()
            DATE = "%s-%s-%s" % (_date.year, _date.month, _date.day)
            FILE_UPLOAD_ROOT_PATH = "./upload/" + DATE + "/"

            if not os.path.exists(FILE_UPLOAD_ROOT_PATH):
                os.makedirs( FILE_UPLOAD_ROOT_PATH )

            #print( "files = ", request.FILES )
            #print( "files = ", request.data.getlist("file") )
            #for index, value in enumerate(request.data.getlist("file")):
            #    print( index, value ) # index, filename
            for _fp in request.data.getlist("file"):
                _filename = str(_fp)
                data = ContentFile(_fp.read())
                #print( _filename )
                #print( data )

                # UUID will be added if duplicate filename
                path = default_storage.save( FILE_UPLOAD_ROOT_PATH + _filename, data )
                print( "file path = ", path )


                #timestamp = str( math.trunc(time.time()) )
                #filename = _filename[:len(_filename)-len(".zip")]
                #ext = _filename[len(_filename)-len(".zip"):]
                #if _fp != None and filename != None and len(filename) > 0 and ext == ".zip":
                #    new_filename = filename + str("-") + timestamp + ext
                #    path = default_storage.save( FILE_UPLOAD_ROOT_PATH + new_filename, data )

            result = { "res": "true" }
            #res = json.loads(str(result))
            res = json.loads(json.dumps(result).encode("utf8"))
            return JsonResponse(res)
        except Exception as e:
            traceback.print_exc()
            result = { "res": "false" }
            #res = json.loads(str(result))
            res = json.loads(json.dumps(result).encode("utf8"))
            return JsonResponse(res)
```

