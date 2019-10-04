Bộ lọc (Filter)
=======

Bộ lọc là các đối tượng chạy trước và / hoặc sau các [controller actions](structure-controllers.md#actions). Ví dụ,
bộ lọc kiểm soát truy cập có thể chạy trước các action để đảm bảo rằng chúng được phép truy cập bởi những người dùng cuối cụ thể;
bộ lọc nén nội dung có thể chạy sau các action để nén nội dung phản hồi trước khi gửi chúng cho người dùng cuối..

Một bộ lọc có thể bao gồm một bộ lọc trước (bộ lọc logic được áp dụng *trước* các action) và / hoặc bộ lọc sau (logic được áp dụng
*sau* các action).


## Sử dụng bộ lọc <span id="using-filters"></span>

Các bộ lọc cơ bản là một loại [hành vi](concept-behaviors.md) đặc biệt. Do đó, sử dụng các bộ lọc cũng giống như
[sử dụng hành vi](concept-behaviors.md#attaching-behaviors). Bạn có thể khai báo các bộ lọc trong lớp controller
bằng việc ghi đè phương thức [[yii\base\Controller::behaviors()|behaviors()]] như sau:

```php
public function behaviors()
{
    return [
        [
            'class' => 'yii\filters\HttpCache',
            'only' => ['index', 'view'],
            'lastModified' => function ($action, $params) {
                $q = new \yii\db\Query();
                return $q->from('user')->max('updated_at');
            },
        ],
    ];
}
```

Theo mặc định, các bộ lọc được khai báo trong lớp controller sẽ được áp dụng cho *tất cả* các action trong controller đó. Tuy nhiên,
bạn có thể chỉ định rõ ràng những hành động nào nên áp dụng bộ lọc bằng cách định cấu hình qua thuộc tính
[[yii\base\ActionFilter::only|only]]. Trong ví dụ trên, bộ lọc `HttpCache` chỉ được áp dụng tới các action
`index` và `view`. Bạn cũng có thể cấu hình vào thuộc tính [[yii\base\ActionFilter::except|except]] để đưa vào danh sách loại bỏ
một vài action không áp dụng bộ lọc.

Bên cạn các controller, bạn có thể khai báo bộ lọc trong các [module](structure-modules.md) hoặc trong [ứng dụng](structure-applications.md).
Khi bạn làm như vậy, các bộ lọc sẽ áp dụng cho *tất cả* các action của controller thuộc về module hoặc ứng dụng,
trừ khi bạn cấu hình thuộc tính cho bộ lọc' [[yii\base\ActionFilter::only|only]] và [[yii\base\ActionFilter::except|except]]
như được mô tả trên.

> Lưu ý: Khi khai báo các bộ lọc trong các module hoặc ứng dụng, bạn nên dùng [routes](structure-controllers.md#routes)
  thay vì các định danh action trong các thuộc tính [[yii\base\ActionFilter::only|only]] và [[yii\base\ActionFilter::except|except]].
  Điều này bởi vì các định danh của action một mình không thể chỉ định đầy đủ các action trong phạm vi của một mô-đun hoặc ứng dụng.

Khi nhiều bộ lọc được cấu hình cho một hành động, chúng được áp dụng theo các quy tắc được mô tả dưới đây:

* Lọc trước
    - Áp dụng các bộ lọc được khai báo trong ứng dụng theo thứ tự chúng được liệt kê trong phương thức `behaviors()`.
    - Áp dụng các bộ lọc được khai báo trong mô-dun theo thứ tự chúng được liệt kê trong phương thức `behaviors()`.
    - Áp dụng các bộ lọc được khai báo trong controller theo thứ tự chúng được liệt kê trong phương thức `behaviors()`.
    - Nếu bất kỳ bộ lọc nào hủy bỏ thực thi hành động, các bộ lọc (bao gồm lọc trước và lọc sau) sau khi nó sẽ không được
     áp dụng.
* Chạy hành động nếu nó vượt qua bộ lọc trước.
* Lọc sau
    - Áp dụng các bộ lọc được khai báo trong bộ điều khiển theo thứ tự ngược lại, chúng được liệt kê trong phương thức `behaviors()`.
    - Áp dụng các bộ lọc được khai báo trong bộ mô-dun theo thứ tự ngược lại, chúng được liệt kê trong phương thức `behaviors()`.
    - Áp dụng các bộ lọc được khai báo trong ứng dụng theo thứ tự ngược lại, chúng được liệt kê trong phương thức `behaviors()`.


## Tạo các bộ lọc <span id="creating-filters"></span>

Để tạo một bộ lọc hành độc mới, ta kế thừa từ lớp [[yii\base\ActionFilter]] và ghi đè lên phương thức
[[yii\base\ActionFilter::beforeAction()|beforeAction()]] và/hoặc phương thức [[yii\base\ActionFilter::afterAction()|afterAction()]]
. Phương thức đầu được thực hiện trước một action trong khi phương thức sau được thực hiện sau khi action đã được thực thi.
Giá trị trả về của phương thức [[yii\base\ActionFilter::beforeAction()|beforeAction()]] xác định xem một hành động nên được thực hiện hay không
. Nếu là `false`, các bộ lọc sau cái này sẽ bị bỏ qua và hành động sẽ không được thực thi.

Ví dụ sau đây cho thấy bộ lọc ghi lại thời gian thực hiện hành động:

```php
namespace app\components;

use Yii;
use yii\base\ActionFilter;

class ActionTimeFilter extends ActionFilter
{
    private $_startTime;

    public function beforeAction($action)
    {
        $this->_startTime = microtime(true);
        return parent::beforeAction($action);
    }

    public function afterAction($action, $result)
    {
        $time = microtime(true) - $this->_startTime;
        Yii::debug("Action '{$action->uniqueId}' thực thi trong $time giây.");
        return parent::afterAction($action, $result);
    }
}
```


## Bộ lọc lõi <span id="core-filters"></span>

Yii cung cấp một bộ các bộ lọc thường được sử dụng, được tìm thấy chủ yếu dưới không gian `yii\filters`. Sau đây, 
chúng tôi sẽ giới thiệu ngắn gọn về các bộ lọc này


### Bộ lọc kiểm soát truy cập [[yii\filters\AccessControl|AccessControl]] <span id="access-control"></span>

AccessControl cung cấp kiểm soát truy cập đơn giản dựa trên một bộ [[yii\filters\AccessControl::rules|quy tắc]].
Đặc biệt, trước khi một action được thực thi, AccessControl sẽ xem xét các quy tắc được liệt kê và tìm quy tắc đầu tiên phù hợp
cho bối cảnh hiện tại (chẳng hạn như địa chỉ IP của người dùng, trạng thái đăng nhập, vv.) Quy tắc phù hợp sẽ quyết định xem có cho phép hoặc
từ chối thực hiện hành động được yêu cầu không. Nếu không có quy tắc phù hợp, truy cập sẽ bị từ chối.

Ví dụ sau đây cho thấy làm thế nào để cho phép người dùng được xác thực truy cập vào hành động `create` và `update`
trong khi từ chối tất cả người dùng khác truy cập hai hành động này.

```php
use yii\filters\AccessControl;

public function behaviors()
{
    return [
        'access' => [
            'class' => AccessControl::className(),
            'only' => ['create', 'update'],
            'rules' => [
                // cho phép người dùng xác thực
                [
                    'allow' => true,
                    'roles' => ['@'],
                ],
                // mọi thứ khác bị từ chối theo mặc định
            ],
        ],
    ];
}
```

Để biết thêm chi tiết về kiểm soát truy cập nói chung, vui lòng tham khảo thêm ở mục [Authorization](security-authorization.md).


### Bộ lọc phương thức xác thực (Authentication Method Filters) <span id="auth-method-filters"></span>

Bộ lọc phương thức xác thực được sử dụng để xác thực người dùng bằng nhiều phương pháp khác nhau, như
[HTTP Basic Auth](http://en.wikipedia.org/wiki/Basic_access_authentication), [OAuth 2](http://oauth.net/2/).
Các lớp bộ lọc này đều nằm trong không gian tên `yii\filters\auth`.

The following example shows how you can use [[yii\filters\auth\HttpBasicAuth]] to authenticate a user using
an access token based on HTTP Basic Auth method. Lưu ý rằng để làm việc này, lớp
[[yii\web\User::identityClass|định danh user]] phải thực thi từ phương thức [[yii\web\IdentityInterface::findIdentityByAccessToken()|findIdentityByAccessToken()]].

```php
use yii\filters\auth\HttpBasicAuth;

public function behaviors()
{
    return [
        'basicAuth' => [
            'class' => HttpBasicAuth::className(),
        ],
    ];
}
```

Bộ lọc phương thức xác thực thường được sử dụng để triển khai RESTful APIs. Để biết thêm chi tiết, tham khảo thêm ở mục
RESTful [Authentication](rest-authentication.md).


### [[yii\filters\ContentNegotiator|ContentNegotiator]] <span id="content-negotiator"></span>

ContentNegotiator supports response format negotiation and application language negotiation. It will try to
determine the response format and/or language qua việc kiểm tra các tham số `GET` và HTTP header `Accept`.

Trong ví dụ sau, ContentNegotiator được thiết lập để hỗ trợ các định dạng phản hồi JSON và XML, và định dạng ngôn ngữ
English (United States) và German.

```php
use yii\filters\ContentNegotiator;
use yii\web\Response;

public function behaviors()
{
    return [
        [
            'class' => ContentNegotiator::className(),
            'formats' => [
                'application/json' => Response::FORMAT_JSON,
                'application/xml' => Response::FORMAT_XML,
            ],
            'languages' => [
                'en-US',
                'de',
            ],
        ],
    ];
}
```

Các định dạng và ngôn ngữ thường cần được xác định sớm hơn nhiều trong quá trình
[vòng đời ứng dụng](structure-applications.md#application-lifecycle). Vì lý do này, ContentNegotiator
được thiết kế theo cách mà nó cũng có thể được sử dụng như một [bootstrapping component](structure-applications.md#bootstrap)
bên cạnh việc được sử dụng như một bộ lọc. Ví dụ, bạn có thể cấu hình nó trong [cấu hình ứng dụng](structure-applications.md#application-configurations)
như sau:

```php
use yii\filters\ContentNegotiator;
use yii\web\Response;

[
    'bootstrap' => [
        [
            'class' => ContentNegotiator::className(),
            'formats' => [
                'application/json' => Response::FORMAT_JSON,
                'application/xml' => Response::FORMAT_XML,
            ],
            'languages' => [
                'en-US',
                'de',
            ],
        ],
    ],
];
```

> Thông tin: Trong trường hợp loại nội dung và ngôn ngữ ưa thích không thể được xác định từ yêu cầu, định dạng và ngôn ngữ đầu tiên
  được liệt kê trong tham số [[formats]] và [[languages]] được sử dụng.



### [[yii\filters\HttpCache|HttpCache]] <span id="http-cache"></span>

HttpCache thực hiện bộ nhớ đệm phía máy khách bằng cách sử dụng HTTP headers `Last-Modified` và `Etag`.
Ví dụ,

```php
use yii\filters\HttpCache;

public function behaviors()
{
    return [
        [
            'class' => HttpCache::className(),
            'only' => ['index'],
            'lastModified' => function ($action, $params) {
                $q = new \yii\db\Query();
                return $q->from('user')->max('updated_at');
            },
        ],
    ];
}
```

Vui lòng tham khảo mục [HTTP Caching](caching-http.md) để biết thêm chi tiết về việc sử dụng HttpCache.


### [[yii\filters\PageCache|PageCache]] <span id="page-cache"></span>

PageCache implements server-side caching of whole pages. In the following example, PageCache is applied
to the `index` action to cache the whole page for maximum 60 seconds or until the count of entries in the `post`
table changes. It also stores different versions of the page depending on the chosen application language.

```php
use yii\filters\PageCache;
use yii\caching\DbDependency;

public function behaviors()
{
    return [
        'pageCache' => [
            'class' => PageCache::className(),
            'only' => ['index'],
            'duration' => 60,
            'dependency' => [
                'class' => DbDependency::className(),
                'sql' => 'SELECT COUNT(*) FROM post',
            ],
            'variations' => [
                \Yii::$app->language,
            ]
        ],
    ];
}
```

Please refer to the [Page Caching](caching-page.md) section for more details about using PageCache.


### [[yii\filters\RateLimiter|RateLimiter]] <span id="rate-limiter"></span>

RateLimiter implements a rate limiting algorithm based on the [leaky bucket algorithm](http://en.wikipedia.org/wiki/Leaky_bucket).
It is primarily used in implementing RESTful APIs. Please refer to the [Rate Limiting](rest-rate-limiting.md) section
for details about using this filter.


### [[yii\filters\VerbFilter|VerbFilter]] <span id="verb-filter"></span>

VerbFilter checks if the HTTP request methods are allowed by the requested actions. If not allowed, it will
throw an HTTP 405 exception. In the following example, VerbFilter is declared to specify a typical set of allowed
request methods for CRUD actions.

```php
use yii\filters\VerbFilter;

public function behaviors()
{
    return [
        'verbs' => [
            'class' => VerbFilter::className(),
            'actions' => [
                'index'  => ['get'],
                'view'   => ['get'],
                'create' => ['get', 'post'],
                'update' => ['get', 'put', 'post'],
                'delete' => ['post', 'delete'],
            ],
        ],
    ];
}
```

### [[yii\filters\Cors|Cors]] <span id="cors"></span>

Cross-origin resource sharing [CORS](https://developer.mozilla.org/en-US/docs/HTTP/Access_control_CORS) is a mechanism that allows many resources (e.g. fonts, JavaScript, etc.)
on a Web page to be requested from another domain outside the domain the resource originated from.
In particular, JavaScript's AJAX calls can use the XMLHttpRequest mechanism. Such "cross-domain" requests would
otherwise be forbidden by Web browsers, per the same origin security policy.
CORS defines a way in which the browser and the server can interact to determine whether or not to allow the cross-origin request.

The [[yii\filters\Cors|Cors filter]] should be defined before Authentication / Authorization filters to make sure the CORS headers
will always be sent.

```php
use yii\filters\Cors;
use yii\helpers\ArrayHelper;

public function behaviors()
{
    return ArrayHelper::merge([
        [
            'class' => Cors::className(),
        ],
    ], parent::behaviors());
}
```

Also check the section on [REST Controllers](rest-controllers.md#cors) if you want to add the CORS filter to an
[[yii\rest\ActiveController]] class in your API.

The Cors filtering could be tuned using the [[yii\filters\Cors::$cors|$cors]] property.

* `cors['Origin']`: array used to define allowed origins. Can be `['*']` (everyone) or `['http://www.myserver.net', 'http://www.myotherserver.com']`. Default to `['*']`.
* `cors['Access-Control-Request-Method']`: array of allowed verbs like `['GET', 'OPTIONS', 'HEAD']`.  Default to `['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'HEAD', 'OPTIONS']`.
* `cors['Access-Control-Request-Headers']`: array of allowed headers. Can be `['*']` all headers or specific ones `['X-Request-With']`. Default to `['*']`.
* `cors['Access-Control-Allow-Credentials']`: define if current request can be made using credentials. Can be `true`, `false` or `null` (not set). Default to `null`.
* `cors['Access-Control-Max-Age']`: define lifetime of pre-flight request. Default to `86400`.

Ví dụ, allowing CORS for origin : `http://www.myserver.net` with method `GET`, `HEAD` and `OPTIONS` :

```php
use yii\filters\Cors;
use yii\helpers\ArrayHelper;

public function behaviors()
{
    return ArrayHelper::merge([
        [
            'class' => Cors::className(),
            'cors' => [
                'Origin' => ['http://www.myserver.net'],
                'Access-Control-Request-Method' => ['GET', 'HEAD', 'OPTIONS'],
            ],
        ],
    ], parent::behaviors());
}
```

You may tune the CORS headers by overriding default parameters on a per action basis.
Ví dụ adding the `Access-Control-Allow-Credentials` for the `login` action could be done like this :

```php
use yii\filters\Cors;
use yii\helpers\ArrayHelper;

public function behaviors()
{
    return ArrayHelper::merge([
        [
            'class' => Cors::className(),
            'cors' => [
                'Origin' => ['http://www.myserver.net'],
                'Access-Control-Request-Method' => ['GET', 'HEAD', 'OPTIONS'],
            ],
            'actions' => [
                'login' => [
                    'Access-Control-Allow-Credentials' => true,
                ]
            ]
        ],
    ], parent::behaviors());
}
```
