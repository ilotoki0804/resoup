> [!CAUTION]
> <big>**This library is unmaintained and replaced with [hxsoup](https://github.com/ilotoki0804/hxsoup).**</big>

# resoup

**Various convenient features related to requests and BeautifulSoup.** (<span style="color:blue">**_re_**</span>quests + Beautiful<span style="color:blue">**_Soup_**</span>)

1. `requests`라이브러리와 BeatifulSoup를 합쳐 몇 줄의 코드를 하나에 합칠 수 있으며,
1. 간단하게 async, cache를 불러와 사용할 수 있습니다.
1. 웹 스크래핑 시 편리한 기본값도 준비되어 있고,
1. `no_empty_result`, `attempts`, `avoid_sslerror` 등 다양하고 소소한 기능도 준비되어 있습니다.

소소하지만 유용하며, 서너 줄의 코드 작성량을 줄여주는 라이브러리입니다.

## 시작하기

1. 파이썬을 설치합니다.
1. 터미널에서 다음과 같은 명령어를 실행합니다.

   ```console
   pip install -U resoup
   ```

requests와 bs4는 같이 설치되지만 BeatifulSoup의 추가적인 parser인 lxml와 html5lib는 기본으로 제공하지 않습니다.

따라서 lxml, html5lib 등은 스스로 설치하셔야 오류가 나지 않을 수 있습니다.
만약 설치되지 않은 상태로 해당 parser를 이용한다면 `NoParserError`가 납니다.

## 사용법

참고: 예시들의 경우 많은 경우 `get` 요청을 위주로 설명하지만, 다른 모든 메소드(options/head/post/put/patch/delete)에서도 동일하게 작동합니다.

### `resoup.requests` 모듈

`resoup.requests` 모듈은 다음과 같이 import해 사용할 수 있습니다.

```python
from resoup import requests  # `import requests`와 호환됨.
```

이 라이브러리는 requests 라이브러리와 99% 호환되며 (심지어 타입 힌트도 requests 라이브러리와 똑같이 잘 작동합니다!), 그 위에 편리한 기능을 얹은 형태입니다. 즉, 기존 `import requests`를 위의 코드로 교체하면 기존의 코드를 망가뜨리지 않으면서도 잘 통합할 수 있습니다.

requests의 Session도 비슷하게 사용할 수 있습니다.

```python
from resoup import requests

with requests.Session() as session:
    ...  # cget, attempts 등 모든 기능 사용 가능
```

#### 기본값

기본값들은 각각 적당한 값으로 설정되어 있습니다.

기본값들은 다음과 같고 request.get/options/head/post/put/patch/delete에서 적용됩니다.

```python
timeout 기본값: 120
headers 기본값: {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
    "Accept-Encoding": "gzip, deflate, br",
    "Accept-Language": "ko-KR,ko;q=0.9",
    "Sec-Ch-Ua": '"Chromium";v="116", "Not)A;Brand";v="24", "Google Chrome";v="116"',
    "Sec-Ch-Ua-Mobile": "?0",
    "Sec-Ch-Ua-Platform": '"Windows"',
    "Sec-Fetch-Dest": "document",
    "Sec-Fetch-Mode": "navigate",
    "Sec-Fetch-Site": "none",
    "Sec-Fetch-User": "?1",
    "Upgrade-Insecure-Requests": "1",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36",
}
attempts 기본값: 1
avoid_sslerror 기본값: False
```

```python
>>> from resoup import requests
>>>
>>> from resoup import requests
>>> res = requests.get("https://httpbin.org/headers")
>>> res.json()['headers']
{'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7',
 'Accept-Encoding': 'gzip, deflate, br',
 'Accept-Language': 'ko-KR,ko;q=0.9',
 'Host': 'httpbin.org',
 'Sec-Ch-Ua': '"Chromium";v="116", "Not)A;Brand";v="24", "Google Chrome";v="116"',
 'Sec-Ch-Ua-Mobile': '?0',
 'Sec-Ch-Ua-Platform': '"Windows"',
 'Sec-Fetch-Dest': 'document',
 'Sec-Fetch-Mode': 'navigate',
 'Sec-Fetch-Site': 'none',
 'Sec-Fetch-User': '?1',
 'Upgrade-Insecure-Requests': '1',
 'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36',
 'X-Amzn-Trace-Id': ...}
```

#### 응답

`resoup.requests` 모듈의 get/options/head/post/put/patch/delete 함수는 모두 ResponseProxy를 리턴합니다.

ResponseProxy는 기존 Response와 100% 호환되는 Response의 subclass입니다. 자세한 내용은 `ResponseProxy` 항목을 참고하세요.

기능을 잘 이해하지 못했다면 기존에 Response를 사용하던 방식대로 사용하시면 문제 없이 작동합니다.

#### attempts

`attempts`는 파라미터로, 모종의 이유로 `ConnectionError`가 발생했을 때 같은 requests를 몇 번 더 반복할 것인지 설정하는 파라미터입니다.

만약 10번을 실행하고도 실패했다면 가장 최근에 실패한 연결의 이유를 보여줍니다.

```python
>>> from resoup import requests
>>>
>>> requests.get('https://some-not-working-website.com', attempts=10)
WARNING:root:Retring...
WARNING:root:Retring...
WARNING:root:Retring...
WARNING:root:Retring...
WARNING:root:Retring...
WARNING:root:Retring...
WARNING:root:Retring...
WARNING:root:Retring...
WARNING:root:Retring...
WARNING:root:Retring...
Traceback (most recent call last):
...
socket.gaierror: [Errno 11001] getaddrinfo failed

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
...
urllib3.exceptions.NameResolutionError: <urllib3.connection.HTTPSConnection object at ...>: Failed to resolve 'some-not-working-website.com' ([Errno 11001] getaddrinfo failed)

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
...
urllib3.exceptions.MaxRetryError: HTTPSConnectionPool(host='some-not-working-website.com', port=443): Max retries exceeded with url: / (Caused by NameResolutionError("<urllib3.connection.HTTPSConnection object at ...>: Failed to resolve 'some-not-working-website.com' ([Errno 11001] getaddrinfo failed)"))

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
...
requests.exceptions.ConnectionError: HTTPSConnectionPool(host='some-not-working-website.com', port=443): Max retries exceeded with url: / (Caused by NameResolutionError("<urllib3.connection.HTTPSConnection object at ...>: Failed to resolve 'some-not-working-website.com' ([Errno 11001] getaddrinfo failed)"))

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
...
ConnectionError: Trying 10 times but failed to get data.
URL: https://some-not-working-website.com
```

### avoid_sslerror

`avoid_sslerror`는 `UNSAFE_LEGACY_RENEGOTIATION_DISABLED`으로 인해 오류가 나타나는 사이트에서 사용할 수 있습니다.

예를 들어 다음의 사이트는 `avoid_sslerror` 없이는 다음과 같은 오류를 일으킵니다.

```python
>>> from resoup import requests
>>> requests.get('https://bufftoon.plaync.com')
---------------------------------------------------------------------------
SSLError                                  Traceback (most recent call last)
...
SSLError: HTTPSConnectionPool(host='bufftoon.plaync.com', port=443): Max retries exceeded with url: / (Caused by SSLError(SSLError(1, '[SSL: UNSAFE_LEGACY_RENEGOTIATION_DISABLED] unsafe legacy renegotiation disabled (_ssl.c:1000)')))
```

`avoid_sslerror`를 `True`로 하면 해당 오류를 피할 수 있습니다.

```python
<Response [200]>
```

#### 일반 요청 함수

일반 requests.get/options/head/post/put/patch/delete를 `requests`에서 사용하던 방식 그대로 사용할 수 있습니다.

다음은 requests.get과 post의 예시입니다. `requests`모듈과 똑같이 작동합니다.

```python
>>> from resoup import requests
>>>
>>> requests.get('https://jsonplaceholder.typicode.com/todos/1').json()  # API that can send request in order to test. Don't execute this command unless you trust this API.
{'userId': 1, 'id': 1, 'title': 'delectus aut autem', 'completed': False}
>>> requests.post('https://jsonplaceholder.typicode.com/todos', json={
...     'title': 'foo',
...     'body': 'bar',
...     'userId': 1,
... }).json()
{'title': 'foo', 'body': 'bar', 'userId': 1, 'id': 201}  # Same with original requests library
```

#### 캐시된 요청 함수

일반 requests.get/../delete 요청과 동일하지만 캐시됩니다. 이때 캐시는 후술할 `비동기적이며 캐시된 요청 함수`와 공유됩니다. 하지만 각 메소드들끼리 공유되지는 않습니다. 앞에 `c`를 붙여 requests.cget/coptions/chead/cpost/cput/cpatch/cdelete로 함수를 작성해 사용할 수 있습니다.

같은 URL을 보내도 다른 결과를 응답할 수 있는 동적인 서비스를 사용하거나(시간에 따른 응답의 변화를 반영하지 않음) 응답의 크기가 클 경우(메모리가 낭비될 수 있음) 사용하지 않는 것이 좋습니다.

```python
>>> # 기기 사양과 인터넷 연결 품질에 따라 결과는 다를 수 있음
>>> import timeit
>>>
>>> timeit.timeit('requests.get("https://python.org")', number=10, setup='from resoup import requests')
1.1833231999917189 # 기기 사양과 인터넷 연결 품질에 따라 다름: 10번의 연결 모두 request를 보냄
>>> timeit.timeit('requests.cget("https://python.org")', number=10, setup='from resoup import requests')
0.10267569999268744 # : 처음 한 번만 request를 보내고 그 뒤는 캐시에서 값을 불러옴
```

#### 비동기적인 요청 함수

비동기적인 요청을 보냅니다. 앞에 `a`를 붙여 requests.aget/aoptions/ahead/apost/aput/apatch/adelete로 함수를 작성합니다.

`run_in_executer`는 기본적으로 켜져 있습니다. 자세한 내용은 아래의 `run_in_executer 사용`을 참고하세요.

```python
>>> import asyncio
>>> 
>>> from resoup import requests
>>>
>>> res = asyncio.run(requests.aget('https://python.org'))
>>> res
<response [200]>
```

#### 비동기적이며 캐시된 요청 함수

비동기적이며 캐시되는 요청입니다. 이때 캐시는 같은 메소드라면 `캐시된 요청 함수`와 공유됩니다. 앞에 `ac`를 붙여 requests.acget/acoptions/achead/acpost/acput/acpatch/acdelete로 함수를 작성합니다.

같은 URL을 보내도 다른 결과를 응답할 수 있는 동적인 서비스를 사용하거나(시간에 따른 응답의 변화를 반영하지 않음) 응답의 크기가 클 경우(메모리가 낭비될 수 있음) 사용하지 않는 것이 좋습니다.

`run_in_executer`는 기본적으로 켜져 있습니다. 자세한 내용은 아래의 `run_in_executer 사용`을 참고하세요.

```python
>>> import asyncio
>>> import timeit
>>>
>>> timeit.timeit('asyncio.run(requests.aget("https://python.org"))', number=10, setup='from resoup import requests; import asyncio')
0.8676127000362612 # 기기 사양과 인터넷 연결 품질에 따라 다름: 10번의 연결 모두 request를 보냄
>>> timeit.timeit('asyncio.run(requests.acget("https://python.org"))', number=10, setup='from resoup import requests; import asyncio')
0.11984489997848868 # 처음 한 번만 request를 보내고 그 뒤는 캐시를 불러옴
```

#### `run_in_executer` 사용

비동기적인 요청(aget, acget 등 a가 붙은 메소드)에서는 `run_in_executer` parameter를 사용할 수 있습니다. 이 parameter는 함수가 다른 쓰레드에서 돌게 합니다. 순차적으로 프로그램이 동작할 때에는 큰 차이가 없지만 병렬적으로 프로그램을 돌릴 때 큰 속도 향상을 기대할 수 있습니다.

아래와 같이 `asyncio.gather`를 이용하면 큰 성능 향상을 보일 수 있습니다.

```python
import asyncio
import time

from resoup import requests

async def masure_coroutine_time(coroutine):
    start = time.perf_counter()
    await coroutine
    end = time.perf_counter()

    print(end - start)

async def main():
    # 단일 request를 보낼 때(큰 차이 없음)

    req = requests.aget('https://python.org', run_in_executor=False)
    await masure_coroutine_time(req)  # 0.07465070000034757

    req = requests.aget('https://python.org')
    await masure_coroutine_time(req)  # 0.05844969999452587

    # 여러 request를 보낼 때(큰 속도 향상을 보임)

    reqs = (requests.aget(f'https://python.org/{i}', run_in_executor=False) for i in range(10))  # 더미 url을 만듦
    await masure_coroutine_time(asyncio.gather(*reqs))  # run_in_executor를 사용하지 않을 때: 느림(3.7874760999984574)

    reqs = (requests.aget(f'https://python.org/{i}') for i in range(10))  # 더미 url을 만듦
    await masure_coroutine_time(asyncio.gather(*reqs))  # run_in_executor를 사용할 때(기본값): 빠름(0.11582900000212248)

if __name__ == '__main__':
    asyncio.run(main())
```

#### requests 모듈과 호환되지 않는 부분

이 모듈은 `requests` 라이브러리와 거의 모든 부분에서 호환되지만 호환되지 않는 부분이 몇 가지 있습니다.

##### dunder method(`__dunder__`)

잠정적 버그의 이유가 될 수 있다는 이유 혹은 기술적인 이유로 일부 dunder method는 불러와지지 않거나 호환되지 않습니다.

사용할 수 없거나 requests 라이브러리와 일치하지 않는 dunder method: `__builtins__`, `__cached__`, `__doc__`, `__file__`, `__loader__`, `__name__`, `__package__`, `__spec__`

사용 가능하고 requests 라이브러리와 일치하는 dunder method: `__author__`, `__author_email__`, `__build__`, `__cake__`, `__copyright__`, `__description__`, `__license__`, `__title__`, `__url__`, `__version__`

```python
>>> import requests
>>> requests.__name__
'requests'
>>> requests.__path__
['some path']
>>> requests.__cake__
'✨ 🍰 ✨'
>>>
>>> from resoup import requests
>>> requests.__name__  # 호환되지 않는 dunder method
'resoup.requests_proxy'  # requests와 값이 다름
>>> requests.__path__ # 사용할 수 없고 호환되지 않는 dunder method
AttributeError: module 'resoup.requests_' has no attribute '__path__'
>>> requests.__cake__  # 호환되는 dunder method
'✨ 🍰 ✨'
```

##### import

`resoup.requests`는 거의 모든 경우에서 import 관련 호환성이 유지됩니다. 하지만 import와 관련해서는 몇 가지 규칙이 존재합니다.

`resoup.requests`는 `from resoup import requests`의 형태로만 사용할 수 있습니다.

```python
# 각 라인에서 윗줄과 아랫줄은 각각 requests를 import 할 때와 `resoup.requests`를 import할 때를 나타냅니다.

# requests 모듈 import
import requests
from resoup import requests  # 가능
```

따라서 다음과 같은 경우는 `resoup.requests`에서 import가 불가능합니다.

```python
# requests의 하위 모듈 import
import requests.models  # 가능
import resoup.requests.models  # 불가능!

# requests의 하위 모듈 import (w/ from .. import ...)
from request import models  # 가능
from resoup.requests import models  # 불가능!

# requests의 하위 모듈의 하위 구성 요소 import
from request.models import Response  # 가능
from resoup.requests.models import Response  # 불가능!
```

이런 경우엔 모듈 import를 이용하면 해결됩니다..

예를 들어 다음과 같은 코드가 있다고 해 봅시다.

```python
from request.models import Response  # 하위 모듈의 하위 구성 요소 import 사용

def is_response(instance):
    return isinstance(instance, Response)
```

이 코드는 다음과 같이 문제를 해결할 수 있습니다.

```python
# requests.models.Response로 바꾸기.
# 장점: 깔끔하고 error-prone하지 않음.
from resoup import requests  # requests 모듈 import
def is_response(instance):
    return isinstance(instance, requests.models.Response)  # requests.models.Response로 변경함
```

```python
# Response 정의하기.
# 장점: 코드를 수정할 필요가 없음.
from resoup import requests
Response = requests.models.Response

def is_response(instance):
    return isinstance(instance, Response)
```

개인의 선호에 따라 원하는 방식으로 사용하시면 됩니다.

### ResponseProxy

`ResponseProxy`는 이 라이브러리에서 requests.get/options/head/post/put/patch/delete를 사용할 경우의 리턴값입니다. 기존 Response와 100% 호환되면서도 추가적인 함수 6개를 제공합니다.

#### 호환성

이 파트에서는 주석에 내용을 적었습니다.

```python
>>> # 두 모듈을 동시에 사용해야 하니 이름을 변경하겠습니다.
>>> import requests as orginal_requests
>>> from resoup import requests as utils_requsts
>>>
>>> # requests 모듈은 Response를 응답합니다.
>>> response1 = orginal_requests.get("https://peps.python.org/pep-0020/")  # 정적인 웹사이트
>>> print(response1)
<Response [200]>
>>> print(type(response1))  # Response 객체
<class 'requests.models.Response'>
>>> # resoup.requests모듈은 ResponseProxy를 응답합니다.
>>> response2 = utils_requsts.get("https://peps.python.org/pep-0020/")
>>> print(response2)
<Response [200]>
>>> print(type(response2))  # ResponseProxy 객체
<class 'resoup.response_proxy.ResponseProxy'>
>>>
>>> # 다음의 모든 검사들을 통과합니다.
>>> assert response1.text == response2.text
>>> assert response1.status_code == response2.status_code
>>> assert response1.url == response2.url
>>> assert response1.content == response2.content
>>>
>>> # 하지만 RequestsProxy에는 이러한 추가적인 기능들이 존재합니다.
>>> print(response2.soup())
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8"/>
...
<script src="../_static/wrap_tables.js"></script>
<script src="../_static/sticky_banner.js"></script>
</body>
</html>
>>> print(response2.soup_select('title'))
[<title>PEP 20 – The Zen of Python | peps.python.org</title>, <title>Following system colour scheme</title>, <title>Selected dark colour scheme</title>, <title>Selected light colour scheme</title>]
>>> print(response2.soup_select_one('p', no_empty_result=True).text)
Long time Pythoneer Tim Peters succinctly channels the BDFL’s guiding
principles for Python’s design into 20 aphorisms, only 19 of which
have been written down.
>>>
>>> from requests.models import Response
>>> # RequestsProxy는 Requsests의 subclass입니다.
>>> # 따라서 isinstance 검사를 통과합니다.
>>> isinstance(response2, Response)
True
>>> # 물론 subclass이기 때문에 '==' 검사는 통과하지 않습니다.
>>> type(response1) == type(response2)
False
```

#### 기본 구조

`ResponseProxy`에는 여러 모듈들이 있으며, 크게 세 가지 종류로 분류됩니다.

* soup류: `.soup()`, `.soup_select()`, `.soup_select_one()`
  기본적인 함수입니다.
* xml류: `.xml()`, `.xml_select()`, `.xml_select_one()`
  soup류에서 parser가 'xml'인 경우입니다.

각각의 종류에는 세 가지 함수가 있으며 함수 각각의 기능은 다음과 같습니다.

* `.soup()`/`.xml()`: BeatifulSoup로 해석된 코드가 나옵니다.
* `.soup_select()`/`.xml_select()`: `.soup().select()`와 비슷합니다.
* `.soup_select_one()`/`.xml_select_one()`: `.soup().select_one()`과 비슷합니다.

자세한 내용은 아래를 살펴보세요.

#### `.soup()`

`.soup()`는 텍스트나 response를 받아 `BeatifulSoup`로 내보냅니다.

이때 인자는 response와 response.text 모두 가능하지만 response를 사용하는 것을 권합니다.
그러면 더욱 상세한 오류 메시지를 받을 수 있습니다.

```python
>>> from resoup import requests
>>>
>>> response = requests.get("https://python.org")
>>> response.soup()  # BeatifulSoup에서 사용 가능한 모든 parameter 사용 가능
<!DOCTYPE html>
...
</body>
</html>
```

이 함수는 사실상 `BeatifulSoup`를 통과시키는 것과 같습니다. 아래의 코드는 위의 코드와 거의 같습니다.

```python
>>> import requests
>>> from bs4 import BeautifulSoup
>>>
>>> response = requests.get("https://python.org")
>>> BeautifulSoup(response.text)
<!DOCTYPE html>
<!DOCTYPE html>
...
</body>
</html>
```

parser가 없을 경우 `BeatifulSoup`는 `FeatureNotFound`에러가 나오지만 `.soup()`는 `NoParserError`가 나옵니다.

#### `.soup_select()`

`.soup_select()`는 텍스트나 response를 받아 BeatifulSoup의 Tag로 내보냅니다. `selector` parameter는 CSS 선택자를 받습니다.

```python
>>> from resoup import requests
>>>
>>> response = requests.get("https://python.org")
>>> response.soup_select("p")
[<p><strong>Notice:</strong> While JavaScript is not essential for this website
...]
```

아래의 코드는 위의 코드와 유사하게 동작합니다.

```python
>>> import requests
>>> from bs4 import BeautifulSoup
>>>
>>> response = requests.get('https://python.org')
>>> soup = BeautifulSoup(response.text).select('p')
>>> soup
[<p><strong>Notice:</strong> While JavaScript is not essential for this website
...]
```

이 함수의 독특한 점은, `no_empty_result`라는 parameter의 존재입니다. 이 parameter가 True이면 .select()의 결과가 빈 리스트일때 `EmptyResultError`를 냅니다.

```python
>>> from resoup import requests
>>>
>>> response = requests.get("https://python.org")
>>> response.soup_select("data-some-complex-and-error-prone-selector")
[]
>>>
>>> response = requests.get("https://python.org")
>>> response.soup_select(
...     "data-some-complex-and-error-prone-selector",
...     no_empty_result=True)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "...souptools.py", line 148, in soup_select
    raise EmptyResultError(
resoup.exceptions.EmptyResultError: Result of select is empty list("[]"). This error happens probably because of invalid selector or URL. Check if both selector and URL are valid. Set to False `no_empty_result` if empty list is intended. It may also because of selector is not matched with URL.
selector: data-some-complex-and-error-prone-selector, URL: https://www.python.org/
```

이 함수를 기본적으로 BroadcastList를 출력값으로 설정하고 있습니다. BroadcastList에 대해 자세히 알고 싶다면 아래의 `BroadcastList` 항목을 확인해 보세요.

#### `.soup_select_one()`

`.soup_select_one()`는 텍스트나 response를 받아 BeatifulSoup의 Tag로 내보냅니다. `selector` parameter는 CSS 선택자를 받습니다.

```python
>>> from resoup import requests
>>>
>>> response = requests.get('https://python.org')
>>> response.soup_select_one('p strong', no_empty_result=True)
<strong>Notice:</strong>
```

아래의 코드는 위의 코드와 유사하게 동작합니다.

```python
>>> import requests
>>> from bs4 import BeautifulSoup
>>>
>>> response = requests.get('https://python.org')
>>> soup = BeautifulSoup(response.text, 'html.parser').select('p strong')
>>> if soup is None:  # no_empty_result 관련 확인 코드
...     raise Exception
...
>>> soup
<strong>Notice:</strong>
```

`no_empty_result` parameter가 True이면 .select_one()의 결과가 None일때 `EmptyResultError`를 냅니다.

이 기능은 타입 힌트에서도 유용하게 쓰일 수 있고, 오류를 더 명확히 하는 데에도 도움을 줍니다.

기존 BeatifulSoup에서는 `.select_one()`의 리턴값을 `Tag | None`으로 표시했기 때문에 만약 `.select_one().text`와 같은 코드를 사용하려고 하면 정적 타입 검사 도구들에서 오류를 발생시켰습니다.

특히 `.select_one()`의 결과가 None이 되면 `'NoneType' object has no attribute 'text'`라는 어떤 부분에서 오류가 났는지 한눈에 확인하기 힘든 오류 메시지가 나왔습니다.

`no_empty_result`를 이용하면 이러한 문제들을 해결할 수 있습니다.
`no_empty_result`를 True로 하면 타입 검사 도구들도 조용해지고, 혹시라도 None이 결과값이 될 때  대신 훨씬 더 자세하며 해결책을 포함한 오류 메시지를 만들어 냅니다.

```python
>>> from resoup import requests
>>>
>>> response = requests.get("https://python.org")
>>> print(response.soup_select_one("data-some-complex-and-error-prone-selector"))
None  # 실제로 None이 결과값으로 나오진 않고 그냥 조용히 종료됨.
>>>
>>> response = requests.get("https://python.org")
>>> response.soup_select_one(
...     "data-some-complex-and-error-prone-selector",
...     no_empty_result=True)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "...souptools.py", line 220, in soup_select_one
    raise EmptyResultError(
resoup.exceptions.EmptyResultError: Result of select_one is None. This error happens probably because of invalid selector or URL. Check if both selector and URL are valid. Set to False `no_empty_result` if empty list is intended. It may also because of selector is not matched with URL.  
selector: data-some-complex-and-error-prone-selector, URL: https://www.python.org/
```

#### xml 관련 함수

`ResponseProxy`의 `soup` 관련 함수에서 `soup`를 `xml`로 치환하면 xml 함수가 됩니다.

이 함수들은 parser가 `'xml'`이라는 점을 제외하고는 soup와 차이점이 없습니다.

예시 코드는 다음과 같습니다

```python
>>> from resoup import requests
>>>
>>> response = requests.get('https://www.w3schools.com/xml/plant_catalog.xml')
>>> selected = response.xml_select('LIGHT', no_empty_result=True)
>>> selected
[<LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Sunny</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Sun or Shade</LIGHT>, <LIGHT>Sun or Shade</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Sun or Shade</LIGHT>, <LIGHT>Sun or Shade</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Sunny</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Sunny</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Sunny</LIGHT>, <LIGHT>Sun or Shade</LIGHT>, <LIGHT>Sun or Shade</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Sun</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Shade</LIGHT>]
```

위의 코드는 아래의 코드와 거의 같습니다.

```python
>>> from resoup import requests
>>> from functools import partial
>>>
>>> response = requests.get('https://www.w3schools.com/xml/plant_catalog.xml')
>>> # corespond to `.xml_select()`
>>> xml_select_partial = partial(response.soup_select, parser='xml')
>>> selected = xml_select_partial('LIGHT', no_empty_result=True)
>>> selected
[<LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Sunny</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Sun or Shade</LIGHT>, <LIGHT>Sun or Shade</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Sun or Shade</LIGHT>, <LIGHT>Sun or Shade</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Sunny</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Sunny</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Sunny</LIGHT>, <LIGHT>Sun or Shade</LIGHT>, <LIGHT>Sun or Shade</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Sun</LIGHT>, <LIGHT>Mostly Shady</LIGHT>, <LIGHT>Shade</LIGHT>, <LIGHT>Shade</LIGHT>]
```

#### BroadcastList

`.soup_select()`와 `.xml_select()`의 경우에는 리스트를 값으로 내보냅니다. 이는 `.soup()`나 `.soup_select_one()`에서 기대할 수 있는 `.text`와 같은 파라미터 사용을 어렵게 합니다.

이는 for loop나 리스트 컴프리헨션으로 해결할 수 있습니다.

```python
>>> from resoup import requests
>>> tags_list = requests.get("https://python.org").soup_select("p strong")
>>> [element.text for element in tags_list]
['Notice:', 'relaunched community-run job board']
```

하지만 이것이 마음에 들지 않을 수가 있습니다. 특히 개발 중이라면 빠른 _개발_ 속도를 위해 for loop나 리스트 컴프리헨션을 사용하는 것 외에 더 신속하게 `.text` 등을 적용하는 방법을 고려하고 싶을 수 있습니다.

이 프로젝트의 `.soup_select()`의 기본 리턴값으로 설정된 BroadcastList는 이를 해결하기 위한 방편입니다.

BroadcastList에서는 리스트를 통해 직접 Tag에서 사용되는 속성을 사용할 수 있습니다.

```python
>>> from resoup import requests
>>> tags_list = requests.get("https://python.org").soup_select("p strong")
>>> tags_list
[<strong>Notice:</strong>, <strong>relaunched community-run job board</strong>]
>>> type(tags_list)
<class 'resoup.broadcast_list.TagBroadcastList'>  # BroadcastList가 사용됨
>>> tags_list.text  # 브로드캐스팅
['Notice:', 'relaunched community-run job board']
>>>
>>> tags_list_with_no_broadcast_list = requests.get('https://python.org').soup_select('p', use_broadcast_list=False)
>>> type(tags_list_with_no_broadcast_list)
<class 'bs4.element.ResultSet'>  # BroadcastList가 사용되지 않음
>>> tags_list_with_no_broadcast_list.text
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "...element.py", line 2428, in __getattr__
    raise AttributeError(
AttributeError: ResultSet object has no attribute 'text'. You're probably treating a list of elements like a single element. Did you call find_all() when you meant to call find()?
```

BroadcastList는 다음과 같은 방법을 통해 끌 수 있습니다.

```python
>>> from resoup import requests
>>>
>>> tags_list = requests.get("https://python.org").soup_select("p", use_broadcase_list=False)
>>> type(tags_list)
bs4.element.ResultSet
>>> tags_list.text  # 브로드캐스팅 안 됨
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "...element.py", line 2428, in __getattr__
    raise AttributeError(
AttributeError: ResultSet object has no attribute 'text'. You're probably treating a list of elements like a single element. Did you call find_all() when you meant to call find()?
```

### 특별한 형태의 리스트 getitem

BroadCastList에서는 다음과 같은 특이한 기능이 있습니다.

만약 리스트에 정수나 슬라이스로 getitem을 요청한다면 일반적인 리스트의 역할을 수행합니다.

```python
>>> from resoup import requests
>>> # 값 불러옴()
>>> tag_broadcast_list = requests.cget("https://www.python.org/community/logos/").soup_select("img")
>>> tag_broadcast_list
[<img alt="Python Software Foundation" class="psf-logo" src="/static/img/psf-logo.png"/>,
...
<img alt="Logo device only" src="https://s3.dualstack.us-east-2.amazonaws.com/pythondotorg-assets/media/community/logos/python-logo-only.png" style="height: 48px;"/>,
<img alt="/static/community_logos/python-powered-w-100x40.png" src="/static/community_logos/python-powered-w-100x40.png"/>,
<img alt="/static/community_logos/python-powered-h-50x65.png" src="/static/community_logos/python-powered-h-50x65.png"/>]
>>> # 정수 getitem
>>> tag_broadcast_list[0]
<img alt="Python Software Foundation" class="psf-logo" src="/static/img/psf-logo.png"/>
>>> # 슬라이싱
>>> tag_broadcast_list[3:5]
[<img alt="/static/community_logos/python-powered-w-100x40.png" src="/static/community_logos/python-powered-w-100x40.png"/>,
 <img alt="/static/community_logos/python-powered-h-50x65.png" src="/static/community_logos/python-powered-h-50x65.png"/>]
>>> # 문자열 getitem (브로드캐스팅 적용됨!)
>>> tag_broadcast_list["alt"]
['Python Software Foundation',
 'Combined logo',
 'Logo device only',
 '/static/community_logos/python-powered-w-100x40.png',
 '/static/community_logos/python-powered-h-50x65.png']
```

### CustomDefaults

`CustomDefaults`를 통해 직접 기본값을 설정할 수 있습니다. 이 값으로 일반 get/options/head/post/put/patch/delete 및 c../a../ac.. 함수의 기본값을 효과적으로 설정할 수 있습니다.

```python
>>> from resoup import CustomDefaults
>>>
>>> requests = CustomDefaults(headers={'User-Agent': 'User Agent for Test'})
>>> requests.get('https://httpbin.org/headers').json()['headers']['User-Agent']
'User Agent for Test'
```

## 라이선스 정보

이 프로그램은 MIT 라이선스로 공유됩니다.

이 프로그램의 일부는 [requests(Apache License 2.0)](https://github.com/psf/requests) 라이브러리에 있던 코드를 포함합니다.
Some part of this program contains code from [requests](https://github.com/psf/requests) library.

이 프로그램의 일부는 [typeshed(Apache License 2.0 or MIT License)](https://github.com/python/typeshed) 라이브러리에 있던 코드를 포함합니다.
Some part of this program contains code from [typeshed](https://github.com/python/typeshed) library.

## Relese Note

0.5.2 (2023-12-26): Timeout 오류도 attempts에 걸릴 수 있도록 변경, root에서 사용할 수 있는 변수 추가, 빌드 코드 개선, 코드 개선

0.5.1 (2023-12-9): 버그 수정

0.5.0 (2023-12-9): resoup로 이름 변경, 새 BroadcastList 기본 적용, poetry 사용, 기존 souptools 모듈 제거 및 souptoolsclass 모듈로 대체, 테스트 추가

0.4.1 (2023-11-4): 긴급 버그 수정

0.4.0 (2023-11-4): raise_for_status 기본값 변경, souptoolsclass 추가, avoid_sslerror 추가

0.3.0 (2023-10-05): BroadcastList 복원, sessions_with_tools 추가

0.2.3 (2023-09-19): header 기본값 변경, ConnectionError시 에러 한 개만 보이는 것으로 변경, attempts로 재시도할 때 성공했을 때 메시지 추가, retry에서 url 제거, setup.py와 관련 파일 변경

0.2.2 (2023-09-08): attempt parameter를 attempts로 변경, BroadcastList 제거

0.2.1 (2023-08-31): py.typed 추가, freeze_dict_and_list 추가

0.2.0 (2023-08-27): CustomDefaults 추가

0.1.1 (2023-08-27): 첫 릴리즈
