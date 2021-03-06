[!code-csharp[Main](../../tutorials/first-web-api/sample/TodoApi/Controllers/TodoController2.cs?name=snippet_todo1)]

前面的代码：

* 定义空控制器类。 在接下来的部分中，我们将添加方法来实现 API。
* 构造函数使用[依赖关系注入](xref:fundamentals/dependency-injection)将数据库上下文 (`TodoContext `) 注入到控制器中。 数据库上下文将在控制器中的每个 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) 方法中使用。
* 构造函数将一个项（如果不存在）添加到内存数据库。

## <a name="getting-to-do-items"></a>获取待办事项

若要获取待办事项，请将下面的方法添加到 `TodoController` 类中。

[!code-csharp[Main](../../tutorials/first-web-api/sample/TodoApi/Controllers/TodoController.cs?name=snippet_GetAll)]

这些方法实现两种 GET 方法：

* `GET /api/todo`
* `GET /api/todo/{id}`

以下是 `GetAll` 方法的 HTTP 响应示例：

```
HTTP/1.1 200 OK
   Content-Type: application/json; charset=utf-8
   Server: Microsoft-IIS/10.0
   Date: Thu, 18 Jun 2015 20:51:10 GMT
   Content-Length: 82

   [{"Key":"1", "Name":"Item1","IsComplete":false}]
   ```

稍后在本教程中将演示如何使用 [Postman](https://www.getpostman.com/) 或 [curl](https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man1/curl.1.html) 查看 HTTP 响应。

### <a name="routing-and-url-paths"></a>路由和 URL 路径

`[HttpGet]` 特性指定 HTTP GET 方法。 每个方法的 URL 路径构造如下所示：

* 在控制器的路由特性中采用模板字符串：

[!code-csharp[Main](../../tutorials/first-web-api/sample/TodoApi/Controllers/TodoController.cs?name=TodoController&highlight=3)]

* 将“[Controller]”替换为控制器的名称，即在控制器类名称中去掉“Controller”后缀。 对于此示例，控制器类名称为“Todo”控制器，根名称为“todo”。 ASP.NET Core [路由](xref:mvc/controllers/routing)不区分大小写。
* 如果 `[HttpGet]` 特性具有路由模板（如 `[HttpGet("/products")]`），则将它追加到路径。 此示例不使用模板。 有关详细信息，请参阅[使用 Http [Verb] 特性的特性路由](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes)。

在 `GetById` 方法中：

```csharp
[HttpGet("{id}", Name = "GetTodo")]
public IActionResult GetById(long id)
```

`"{id}"` 是 `todo` 项 的 ID 的占位符变量。 调用 `GetById` 时，它会将 URL 中“{id}”的值分配给方法的 `id` 参数。

`Name = "GetTodo"` 创建一个命名的路由，使你能够 HTTP 响应中链接到此路由。 稍后将使用示例进行解释。 有关详细信息，请参阅[路由到控制器操作](xref:mvc/controllers/routing)。

### <a name="return-values"></a>返回值

`GetAll` 方法返回 `IEnumerable`。 MVC 自动将对象序列化为 [JSON](http://www.json.org/)，并将 JSON 写入响应消息的正文中。 在假设没有未经处理的异常的情况下，此方法的响应代码为 200。 （未经处理的异常将转换为 5xx 错误。）

相反，`GetById` 方法返回多个常规的 `IActionResult` 类型，它表示一系列返回类型。 `GetById` 具有两个不同的返回类型：

* 如果没有任何项与请求的 ID 匹配，此方法将返回 404 错误。  此通过返回 `NotFound` 来实现。

* 否则，此方法将返回具有 JSON 响应正文的 200。 此通过返回 `ObjectResult` 来完成
