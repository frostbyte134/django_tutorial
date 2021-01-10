# django_tutorial
code of https://www.django-rest-framework.org/tutorial


# pythonenv
in current local PC (nailbrainz-PC), restart the django container, and type
```
root@nailbrainz-ROG-STRIX-Z390-F-GAMING:/ext/ssd2/codes/django_tutorial# source ../Django/djangoenv/bin/activate
```


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


### VScode + container 포트 고갈 이슈
- 컨테이너를 VSCode의 터미널로 사용중인데, `python manage.py runserver`로 장고를 실행하면 자기가 알아서 컨테이너의 포트 - 밖 운영체제의 포트로 매핑을 해 줌.
- 문제는, __VSCode를 종료해도 이 포트 포워딩 설정을 유지하고 있다는 것__
    - 재시작하고 `python manage.py runserver` 명령어를 치면 포트가 점유되고 있다고 나옴
- 옆 Remote Explorer에서 맨 밑 forwarded port에서 포트포워딩 제거하면 됨