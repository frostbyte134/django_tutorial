# django_tutorial
code of https://www.django-rest-framework.org/tutorial


# pythonenv
in current local PC (nailbrainz-PC), restart the django container, and type
```
root@nailbrainz-ROG-STRIX-Z390-F-GAMING:/ext/ssd2/codes/django_tutorial# source ../Django/djangoenv/bin/activate
```

이후
```
python manage.py runserver
``` 
으로 실행

************************************

### Tutorial 1 Serialization
<a href="https://www.django-rest-framework.org/tutorial/1-serialization/" target="_blank">https://www.django-rest-framework.org/tutorial/1-serialization/<.a>

- what happens when i type `python manage.py migrate`?
- made a model
- made a file `snippets/setializers.py`, and made serialization class for the model (support partial CRUD)
    - model의 각 field와의 serialization은 자동으로 되지 않음. update 함수에서 일일히 지정
        -  The `create()` and `update()` methods define how fully fledged instances are created or modified when calling serializer.save()
    - A serializer class is very similar to a Django `Form` class, and includes similar validation flags on the various fields, such as required, max_length and default
    - The field flags can also control how the serializer should be displayed in certain circumstances
- django serialization : python native datatypes  ->  model instance (in JSON, DB, in bytes) 
- We can also serialize querysets instead of model instances : ?



model과 serializer가 같은 필드에 대해 비슷한 선언 작업을 중복으로 함 -> `ModelSerializer`를 통해 중복 제거
- `class Meta` 에서 대상 모델과 필드들 지정

> It's important to remember that ModelSerializer classes don't do anything particularly magical, they are simply a shortcut for creating serializer classes:  
    An automatically determined set of fields.  
    Simple default implementations for the create() and update() methods.  


#### Writing regular Django views using our Serializer
<a href="https://www.django-rest-framework.org/tutorial/1-serialization/#writing-regular-django-views-using-our-serializer" target="_blank">https://www.django-rest-framework.org/tutorial/1-serialization/#writing-regular-django-views-using-our-serializer</a>

- writing some API views using (new) Serializer class
- We'll also need a view which corresponds to an individual snippet, and can be used to retrieve, update or delete the snippet.
- 각 모델을 건드릴 수 있는 view가 있어서, 데이터 검색/수정 등을 해 주는 듯 (view당 endpoint 1개 할당? 아직은 잘 모름)
- `view` = `request`s 인 듯?
    - view하나가 get, post 등의 기본 리퀘스트들을 처리
- 생성한 view는 snippet model을 여러 방식 (listing, read details) 으로 접근가능함
    - endpoint는 `snippets/urls.py`, `tutorial/urls.py`를 직접 만들어 내용을 채워넣어야 했음

************************************

### VScode + container 포트 고갈 이슈
- 컨테이너를 VSCode의 터미널로 사용중인데, `python manage.py runserver`로 장고를 실행하면 자기가 알아서 컨테이너의 포트 - 밖 운영체제의 포트로 매핑을 해 줌.
- 문제는, __VSCode를 종료해도 이 포트 포워딩 설정을 유지하고 있다는 것__
    - 재시작하고 `python manage.py runserver` 명령어를 치면 포트가 점유되고 있다고 나옴
- 옆 Remote Explorer에서 맨 밑 forwarded port에서 포트포워딩 제거하면 됨



************************************

### Tutorial 2 Requests and Responses

#### Requet / Response object
- `Django request` extends the regular `HttpRequest`
    - `request.POST` : Only handles form data.  Only works for 'POST' method.
    - `request.data` : Handles arbitrary data.  Works for 'POST', 'PUT' and 'PATCH' methods.

#### Wrapping API views

뷰의 구현
- 함수기반 뷰 : `@api_view` decorator 사용
- 클래스기반 뷰 : `APIView` 클래스 상속받아 구현
- Django View에서는
    1. view 함수 내에서 Request instances 를 받도록 보장해주며 
    2. adding `context` to Response objects so that `content negotiation` can be performed.
    3. 근데 임포트는 `from rest_framework.response import Response` 인데...? rest_framework는 다른 프레임워크 아닌가?

#### 함수기반 뷰 (with decorator)
 - 에전에 만들었던 `def snippet_list(request):` 위에 `@api_view(['GET', 'POST'])` decorator를 더함 (parameterized decorator? 기억이 잘안나네)
 - response도 이전에는 지정해 줬었는데 (JsonResponse) context에 따라 알아서 처리해 주는 듯
 - `status=201` 보다는 `status=status.HTTP_201_CREATED`
 - view안에서 `serializer.save()`로 response data 를 채웠으면, 안에 json이 있음. 이 경우 django response는 JsonResponse를 지정해 줄 필요가 없음 (그냥 Response로 감싸서 return)

#### format suffix 처리하기
- endpoint 설정하는 곳(`urls.py`)에서 `urlpatterns = format_suffix_patterns(urlpatterns)` 추가
- 함수기반 뷰의 파라미터에 format=None 추가, `def snippet_list(request, format=None):`

이러면 request 전달 시 format suffix가 지정된 경우
- ex)`http://127.0.0.1:8000/snippets.json` 나 `http://127.0.0.1:8000/snippets.api`
- django (rest_framework?) __response는__ 상황에 맞는 type를 반환해 줌
- `.api` : `Browsable API`?
    - <a href="https://www.django-rest-framework.org/topics/browsable-api/" target="_blank">https://www.django-rest-framework.org/topics/browsable-api/</a>
    - 일단 request에 대해, 특히 browser가 request한 경우 browsable한 html문서를 반환해 준다는 것 같음

또는, accept header에 response type이 명시되어 있는 경우도 자동으로 처리 가능
```
http http://127.0.0.1:8000/snippets/ Accept:application/json  # Request JSON
http http://127.0.0.1:8000/snippets/ Accept:text/html         # Request HTML
```

### 3 - Class based Views

class based views helps us keep our code <a href="https://en.wikipedia.org/wiki/Don't_repeat_yourself" target="_blank">DRY.</a>

```
@api_view(['GET', 'POST'])
def snippet_list(request, format=None):
    if request.method == 'GET':
        ...
    if request.method == 'POST':
        ...
```

를 아래와 같이 바꿀 수 있음

```
class SnippetList(APIView):
    def get(self, request, format=None):
        ...
    def post(self, request, format=None):
        ...
```

__url pattern__ 도 다음과 같이 바꿈

```
path('snippets/', views.snippet_list),
```
에서
```
urlpatterns = [
    path('snippets/', views.SnippetList.as_view()),
]
```

으로


#### mixins
view들의 get, post 등은 보통 하는 일이 엄청나게 다양하지는 않음 (ex - get은 보통 아이템 하나를 가져오거나 listing하는 경우가 많음).  
이를 일일이 구현할 필요 없이 (MVC 구조여서 가능) 공통연산을 미리 mixins으로 구현해서 이를 다중상속받아 쓰면 편함

```
def get(self, request, format=None):
    snippets = Snippet.objects.all()
    serializer = SnippetSerializer(snippets, many=True)
    return Response(serializer.data)
```
을


```
class SnippetDetail(mixins.RetrieveModelMixin,
                    ...
                    generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)
```

으로 바꿀 수 있음

심지어 많은 view들이 (다른 model에 대해) 비슷한 일을 하기 때문에, 이를 `generics.SomeViewName`으로 상속받아 코드 길이를 더 줄일 수 있음

```
class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```