# [Django] MIDDLEWARE 와 MIDDLEWARE_CLASSES에 대한 정리

<!--more-->

## 개요

저는 django로 들어오는 request에서 필요한 정보들을 뽑아내고 해당 정보를 로그로 남길 필요성이 생겨 django middleware를 활용해보게 되었고, 이 과정을 통해 알게된 소소한 MIDDLEWARE에 대한 히스토리(?)를 이번 글을 통해 정리해보고자 합니다.



## MIDDLEWARE란?

MIDDLEWARE란 Django의 요청/응답 처리에 대한 후크 프레임워크입니다. 다시말해 Django로 들어오는 요청을 가로채 특정 작업을 하고 view로 보낸다거나 반대로 view를 거쳐 생성된 응답을 가로채 특정 작업을 한 뒤 클라이언트에게 보낼 수 있게 됩니다.



## MIDDLEWARE는 어떻게 사용할까?

Django의 MIDDLEWARE는 `settings.py`에서 사용할 미들웨어 클래스를 설정할 수 있습니다. 

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

위와 같이 설정된 미들웨어는 정의된 순서에 따라 요청 시에는 위에서부터 응답 시에는 아래에서부터 차례대로 진행됩니다. 레퍼런스 문서를 보면 이와 같은 특징으로 미들웨어를 양파 껍질에 비유를 하고있었습니다.



## Custom Middleware 구현

Django 프로젝트를 만들면 기본적으로 생성되는 미들웨어 이외의 커스텀한 클래스가 필요하다면 아래와 같이 미들웨어 클래스를 만들어 설정만 해준다면 얼마든지 사용이 가능합니다.

```python
class SimpleMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
				# view 또는 하위 미들웨어가 호출되기 전 실행이 되는 부분

        response = self.get_response(request)

        # view 또는 하위 미들웨어를 거친 뒤 실행이 되는 부분

        return response
```

`SimpleMiddleware` 클래스는 `__init__()`  메서드가 동작하는 과정에서 `self.get_response` 에 다음 단계의 미들웨어 혹은 실행될 view에 대한 정보가 담기게 됩니다. 또한, 실제로 `__call__()` 메서드의 동작을 통해 미들웨어가 동작하게 됩니다. 

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
  	'SimpleMiddleware',
]
```

예를들어 위와 같이 미들웨어가 설정되어있다면 `SimpleMiddleware` 의 동작을 간단히 살펴보면 다음과 같습니다. `django.middleware.clickjacking.XFrameOptionsMiddleware` 내부의 `get_response`를 통해 `SimpleMiddleware`가 실행되며, `SimpleMiddleware`의 `get_response`에는 실행될 view에 대한 정보가 담겨 실제로 view의 동작을 마친 뒤 생성된 response를 `django.middleware.clickjacking.XFrameOptionsMiddleware`로 반환하게 되며 `SimpleMiddleware`의 동작은 끝나게됩니다.

<div style="text-align:center">
<img width="600" alt="img" src="https://user-images.githubusercontent.com/78338337/132503139-1aea7129-c5b8-4267-a986-45b70cc4328e.png">
</div>



## MIDDLEWARE_CLASSES는 뭐고? MIDDLEWARE와의 차이점은?

MIDDLEWARE_CLASSES는 Django 1.10 이전에 MIDDLEWARE가 생겨나기 전에 사용되던 미들웨어를 지정하는 옵션으로 MIDDLEWARE와 동작에서 차이점이 존재하는데 이는 아래와 같습니다.
<br><br>
- MIDDLEWARE_CLASSES의 경우, `self.get_response`가 실행되기 전에 반환할 response가 존재하게 된다면 `get_response`가 실행되지 않고 해당 response를 반환해버리게 됩니다. 
- `process_exception()`이 MIDDLEWARE_CLASSES의 경우에는 `get_response()`이전에 실행되는 `process_request()`에서 발생하는 예외에 대해서만 적용이 되는 반면, MIDDLEWARE는 view에서 발생하는 예외에 대해서만 적용이 됩니다.
- MIDDLEWARE_CLASSES에서 `get_response()`가 실행된 후 동작하는 `process_response()`에서 예외가 발생하는 경우에는 이후 과정을 모두 생략하고 500 에러를 바로 뱉는 반면, MIDDLEWARE는 예외가 발생하는 즉시 적절한 HTTP Response로 변환한 뒤에 반환해서 이후 과정을 생략하지 않고 처리하게 됩니다.



## 호환성을 위해서라면? MiddlewareMixin을 사용

MIDDLEWARE_CLASSES와 MIDDLEWARE 모두 호환을 시키기 위해서는 MiddlewareMixin을 상속받아 `process_request()`, `process_response()`를 정의해주면 됩니다.

```python
class TestMiddleware(MiddlewareMixin):

  def process_request(self, request):
			# get_response 실행 이전에 실행되는 코드
      
  def process_response(self, request, response):
			# get_response 실행 이후에 실행되는 코드

      return response
```

MiddlewareMixin의 `__call__` 메서드를 살펴보면 아래와 같아 동작을 이해하는데 도움이 됩니다.

```python
def __call__(self, request):
  # Exit out to async mode, if needed
  if asyncio.iscoroutinefunction(self.get_response):
      return self.__acall__(request)
    
  response = None
  if hasattr(self, 'process_request'):
      response = self.process_request(request)
      
  # process_request()에서 response가 있다면 get_response()는 실행되지 않는다.
  response = response or self.get_response(request)		
  
  if hasattr(self, 'process_response'):
      response = self.process_response(request, response)
      
  return response
```



## 참고문서

- [Middleware | Django documentation | Django](https://docs.djangoproject.com/en/1.10/topics/http/middleware/#writing-your-own-middleware)

- [Settings | Django documentation | Django](https://docs.djangoproject.com/en/1.10/ref/settings/#std:setting-MIDDLEWARE_CLASSES)
