# Flask

学习来源：
- [Flask Documentation](https://flask.palletsprojects.com/en/stable/)

说明：
> 1.本笔记部分内容参考/摘录自 Flask 官方文档。
> 2.仅用于个人学习记录，包含官方示例代码以及个人整理的理解。
> 3.Flask 文档采用 BSD-3-Clause 许可证，版权归 Pallets 团队所有。

————

序言：此笔记系笔者在学习Flask过程中总结的基础概念。如有错漏，敬请谅解，欢迎前往 Issues 提出意见，笔者会及时修正。笔者会尽力完善此笔记，力求精准简洁。

---

### 以下为阅读此笔记时，需要注意的标识：

- **术语** —— 表示特定概念、事物或现象的概括性词汇，具有唯一性、精确性。
- *对于术语的解释* —— 解释术语表示的特定概念、事物或现象，通常出现在术语的首次出现。
- ***重要的*** —— 笔者认为需要掌握的内容（主观判断，仅供参考）。
- ```代码块``` —— 用以整块代码的展示，使用markdown的围栏式。
- `行内代码` —— 用以在行内解释整块代码中的某行代码。
- 引用块 —— 用以提示、补充说明或扩展知识，例如：
> 提示：可开始阅读此笔记。

---

# [目录](#目录)
1. [基础概念](#1基础概念)

---

## 1.基础概念

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "<p>Hello,World!</p>"
```
>from flask import Flask，导入Flask类。
>创建一个Flask类的实例，取名为app。
>使用route()装饰器来确定触发函数的URL。
>根据函数返回值，默认类型是HTML。

注意：程序名应避免命名为flask.py，会引发冲突，建议app.py或wsgi.py。

运行程序时需以flask或python命令(使用app.py或wsgi.py作为程序名则不需要--app参数)：
>flask --app hello run。
>python -m flask --app hello run。

调试模式的开启是--debug参数，启用调试模式可在命令行里使用run指令，添加--host=0.0.0.0，即可使服务器公开。

```python
from markupsafe import escape

@app.route("/<name>")
def hello(name):
    return f"Hello, {escape(name)}!"
```
其中的escape()可将用户输入进行转义，以防止注入攻击，该方法默认自动转义，在使用字符串拼接时需手动使用，例如该例。
路径中的name则从URL中捕获一个值传递给视图函数。
可使用使用 route() 装饰器将函数绑定到 URL。

```python
from markupsafe import escape

@app.route('/user/<username>')
def show_user_profile(username):
    # show the user profile for that user
    return f'User {escape(username)}'

@app.route('/post/<int:post_id>')
def show_post(post_id):
    # show the post with the given id, the id is an integer
    return f'Post {post_id}'

@app.route('/path/<path:subpath>')
def show_subpath(subpath):
    # show the subpath after /path/
    return f'Subpath {escape(subpath)}'
```
第二个/后为variable_name变量，函数可接收该变量，也可使用转换器进行转换，转换器类型如下:

| 类型             | 接受数据类型               |
| -------------- | -------------------- |
| `string` （字符串） | （默认）接受任何不带斜杠的文本      |
| `int` （正整数）    | 接受正整数                |
| `float` （浮点数）  | 接受正浮点数               |
| `path` （路径）    | 与 `string` 类似，但也接受斜杠 |
| `uuid` （唯一标识符） | 接受 UUID 字符串          |

```python
@app.route('/projects/')
def projects():
    return 'The project page'

@app.route('/about')
def about():
    return 'The about page'
```
projects端点规范URL末尾带有一个斜杠，该路径就像文件夹，如末尾没有携带斜杠，会重定向到带有斜杠的路径。
about端点规范URL末尾没有斜杠，该路径就像文件，访问带有尾部斜杠的该类端点时会显示404错误。
其区分就在于URL末尾是否有斜杠。

```python
from flask import url_for

@app.route('/')
def index():
    return 'index'

@app.route('/login')
def login():
    return 'login'

@app.route('/user/<username>')
def profile(username):
    return f'{username}\'s profile'

with app.test_request_context():
    print(url_for('index'))
    print(url_for('login'))
    print(url_for('login', next='/'))
    print(url_for('profile', username='John Doe'))
```
构建指向特定函数的URL，使用url_for()函数，接受函数名称为第一个参数，及任意数量的参数，每个参数对应于URL规则的一个变量部分，未知的变量作为查询参数附加到URL中。

```python
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return do_the_login()
    else:
        return show_the_login_form()
```
在使用Flask时，一个路径只响应GET请求，不过可以使用route()装饰器的methods参数处理不同的http方法。
```python
@app.get('/login')
def login_get():
    return show_the_login_form()

@app.post('/login')
def login_post():
    return do_the_login()
```
也可使用不同的方法，将不同请求分离。
如果存在 `GET`，Flask 会自动添加对 `HEAD` 方法的支持，并根据 [HTTP RFC](https://www.ietf.org/rfc/rfc2068.txt) 处理 `HEAD` 请求。同样，`OPTIONS` 也会自动为您实现。

```python
url_for('static', filename='style.css')
```
Flask的静态文件需在包中或模块旁创建一个名为static的文件夹，静态文件就放在里面，使用以上示例在程序中生成对应URL。

```python
from flask import render_template

@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
    return render_template('hello.html', person=name)
```
生成HTML需要模版，Flask配备了jinja2模版引擎，可使用render_template渲染模版，只需提供模版名称及参数，此方法会在template文件夹中寻找模版，和static相同，这个文件夹在包中或模块旁，目录结构如下：
**情况 1**: 一个模块:
/application.py
/templates
    /hello.html

**情况 2**: 一个包:
/application
    /`__init__`.py
    /templates
        /hello.html

对于Flask来说，访问请求数据使用的是request对象，该对象为全局，且做到多线程安全是因为上下问局部变量。
上下文局部变量在于一个请求到达时，会判断其是否为活动上下文，并将当前应用程序和WSGI环境进行绑定。
在进行单元测试时，请求对象的代码突然出错，因为其上下文局部变量，所以测试时需要自己创建一个请求对象将其绑定在上下文，最简单的方法是使用test_request_context上下文管理器和with()语句：
```python
from flask import request

with app.test_request_context('/hello', method='POST'):
    # now you can do something with the request until the
    # end of the with block, such as basic assertions:
    assert request.path == '/hello'
    assert request.method == 'POST'
```
另一种是将整个WSGI环境传递给request_context()方法：
```python
with app.request_context(environ):
    assert request.method == 'POST'
```

```python
from flask import request

@app.route('/login', methods=['POST', 'GET'])
def login():
    error = None
    if request.method == 'POST':
        if valid_login(request.form['username'],
                       request.form['password']):
            return log_the_user_in(request.form['username'])
        else:
            error = 'Invalid username/password'
    # the code below is executed if the request method
    # was GET or the credentials were invalid
    return render_template('login.html', error=error)
```
导入request，请求方法可用methods属性获取，访问表单数据，POST或PUT请求传输的数据，可使用form属性。
如form属性为空，会引发一个特殊的KeyErroe错误，可像正常错误一样捕获，也可无作为，显示400错误页面，所以大多数情况下无需处理。
访问URL中提交的参数(？key=value)，可使用args属性：
```python
searchword = request.args.get('key', '')
```

只需确保不要忘记在 HTML 表单上设置 enctype="multipart/form-data" 属性，就可以使用 Flask 轻松处理上传的文件。
上传的文件将被临时存储在一个临时位置或内存中，可使用file属性进行访问，且具有save()方法，允许将此文件存储在服务器上：
```python
from flask import request

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/uploaded_file.txt')
```
也可知道文件在上传前的文件名，通过访问filename属性，但不可信，使用Werkzeug 提供的secure_filename()可以上传前文件名存储在服务器上：
```python
from werkzeug.utils import secure_filename

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        file = request.files['the_file']
        file.save(f"/var/www/uploads/{secure_filename(file.filename)}")
```

cookies属性可访问Cookie，设置cookie，可使用响应对象的set_cookie方法，如需使用会话，请使用Flask的会话。
读取：
```python
from flask import request

@app.route('/')
def index():
    username = request.cookies.get('username')
    # use cookies.get(key) instead of cookies[key] to not get a
    # KeyError if the cookie is missing.
```
存储：
```python
from flask import make_response

@app.route('/')
def index():
    resp = make_response(render_template(...))
    resp.set_cookie('username', 'the username')
    return resp
```
Cookie 是设置在响应对象上的，Flask 会将它们转换为响应对象，可使用make_response()函数修改。

```python
from flask import abort, redirect, url_for

@app.route('/')
def index():
    return redirect(url_for('login'))

@app.route('/login')
def login():
    abort(401)
    this_is_never_executed()
```
将用户重定向到另一个端点，可使用redirect()函数，带一个错误代码提前中止请求，使用abort()函数。
也可使用errorhandler()装饰器自定义错误页面：
```python
from flask import render_template

@app.errorhandler(404)
def page_not_found(error):
    return render_template('page_not_found.html'), 404
```
render_template()调用后面的404，表示未找到。

视图函数的返回值会自动转换为响应对象，其逻辑如下：
1. 如果返回了类型正确的响应对象，则它将直接从视图返回。
2. 如果它是一个字符串，则使用该数据和默认参数创建一个响应对象。
3. 如果它是一个返回字符串或bytes的迭代器或生成器，则它将被视为流响应。
4. 如果它是一个字典或列表，则使用 jsonify()创建一个响应对象。
5. 如果返回的是元组，则元组中的项可以提供额外的信息。此类元组的格式必须为 (response, status), (response, headers), 或 (response, status, headers) 。status值将覆盖状态码，headers可以是包含附加HTTP标头值的列表或字典。
6. 如果这些都不符合，Flask 将假定返回值是一个有效的 WSGI 应用程序并将其转换为响应对象。
想要在视图中获取响应对象的结果，可以使用 make_response() 函数：
```python
from flask import render_template

@app.errorhandler(404)
def not_found(error):
    return render_template('error.html'), 404
```
使用make_response()包装返回表达式然后获取响应对象修改，最后返回：
```python
from flask import make_response

@app.errorhandler(404)
def not_found(error):
    resp = make_response(render_template('error.html'), 404)
    resp.headers['X-Something'] = 'A value'
    return resp
```

编写 API 时，JSON 是一种常见的响应格式，从视图返回一个 `dict` 或 `list` ，它将被转换为 JSON 响应。
```python
@app.route("/me")
def me_api():
    user = get_current_user()
    return {
        "username": user.username,
        "theme": user.theme,
        "image": url_for("user_image", filename=user.image),
    }

@app.route("/users")
def users_api():
    users = get_all_users()
    return [user.to_json() for user in users]
```
这是将数据传递给 jsonify()函数的快捷方式，该函数将序列化任何支持的 JSON 数据类型。这意味着字典或列表中的所有数据都必须是可 JSON 序列化的。
对于数据库模型等复杂类型，您需要先使用序列化库将数据转换为有效的 JSON 类型。

除了请求对象之外，还有一个名为session的对象，可以在几个请求间存储特定于用户的信息，它基于 cookie 实现，并对 cookie 进行加密签名。这意味着用户可以查看你的 cookie 内容，但无法修改，除非他们知道用于签名的密钥。
要使用sessions，必须设置一个密钥。
```python
from flask import session

# Set the secret key to some random bytes. Keep this really secret!
app.secret_key = b'_5#y2L"F4Q8z\n\xec]/'

@app.route('/')
def index():
    if 'username' in session:
        return f'Logged in as {session["username"]}'
    return 'You are not logged in'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form method="post">
            <p><input type=text name=username>
            <p><input type=submit value=Login>
        </form>
    '''

@app.route('/logout')
def logout():
    # remove the username from the session if it's there
    session.pop('username', None)
    return redirect(url_for('index'))
```
使用以下命令可以快速生成用于 `Flask.secret_key` (或 SECRET_KEY的值:
>python -c 'import secrets; print(secrets.token_hex())’
>Flask 会将传入session对象的值序列化为 Cookie。如果发现某些值在请求之间无法保持，且 Cookie 确实已启用，而没有收到清晰的错误消息，请检查页面响应中 Cookie 的大小，并将其与 Web 浏览器支持的大小进行比较。

---

# *"日复一日，必有精进！"*