## 返回状态码和消息格式
### 1. Client Errors
1. Sending invalid JSON will result in a `400 Bad Request` response.

```
HTTP/1.1 400 Bad Request
Content-Length: 35

{"message":"Problems parsing JSON"}
```

2. Sending the wrong type of JSON values will result in a `400 Bad Request` response.

```
HTTP/1.1 400 Bad Request
Content-Length: 40

{"message":"Body should be a JSON object"}
```

3. Sending invalid fields will result in a `422 Unprocessable Entity` response.

```
HTTP/1.1 422 Unprocessable Entity
Content-Length: 149

{
  "message": "Validation Failed",
  "errors": [
    {
      "resource": "Issue",
      "field": "title",
      "code": "missing_field"
    }
  ]
}
```

All error objects have resource and field properties so that your client can tell what the problem is. There's also an error code to let you know what is wrong with the field. These are the possible validation error codes:

| Error Name | Description |
|--------|--------|
| `missing` | This means a resource does not exist. |
| `missing_field` | This means a required field on a resource has not been set. |
| `invalid` | This means the formatting of a field is invalid. The documentation for that resource should be able to give you more specific information. |
| `already_exists` | This means another resource has the same value as this field. This can happen in resources that must have some unique key (such as Label names).|

### 2. HTTP Redirects

| Status Code | Description |
| ------- | ------- |
| `301` | Permanent redirection. The URI you used to make the request has been superseded by the one specified in the Location header field. This and all future requests to this resource should be directed to the new URI. |
| `302`, `307` | Temporary redirection. The request should be repeated verbatim to the URI specified in the Location header field but clients should continue to use the original URI for future requests. |

### 3. HTTP Verbs
### 4. Authentication
Authenticating with invalid credentials will return `401 Unauthorized`:
```
curl -i https://api.github.com -u foo:bar
HTTP/1.1 401 Unauthorized

{
  "message": "Bad credentials",
  "documentation_url": "https://developer.github.com/v3"
}
```
After detecting several requests with invalid credentials within a short period, the API will temporarily reject all authentication attempts for that user (including ones with valid credentials) with `403 Forbidden`:
```
curl -i https://api.github.com -u valid_username:valid_password
HTTP/1.1 403 Forbidden

{
  "message": "Maximum number of login attempts exceeded. Please try again later.",
  "documentation_url": "https://developer.github.com/v3"
}
```
## validator
```bash
$ vim app/Http/Requests/IKSUserRequest.php
class IKSUserRequest extends Request
{
    public function rules()
    {
        return [
            'email' => 'unique:iksuser,email|email',
            'ipmask' => 'required|string',
        ];
    }
}
$ php artisan make:request IKSUserController
$ vim app/Http/Controllers/IKSUserController.php
class IKSUserController extends Controller
{
    public function update(IKSUserRequest $request, $id) // 更新, PUT
    {
    }
}
$ vim app/Exceptions/Handler.php
class Handler extends ExceptionHandler
{
    public function render($request, Exception $e)
    {
        if ($e instanceof ValidationException) {
            return $this->handleValidationException($request, $e);
        }

        $statusCode = $e->getStatusCode();
        $message = $e->getMessage();
        return response()->json(array('message' => $message), $statusCode);
    }

    protected function handleValidationException($request, $e)
    {
        $errors = @$e->validator->errors();
        $ret = [];
        $ret['message'] = 'Validation Failed';
        $ret['errors'] = [];
        $errors = $errors->toArray();
        foreach ($errors as $key => $error) {
            $ret['errors'][] = [
                'resource' => 'Issue',
                'field' => $key,
                'code' => $error,
            ];
        }
        return response()->json($ret, $status = 422, $headers = [], $options = JSON_PRETTY_PRINT);
    }
}
```