# Simple PHP MVC

✅: Done.
❌: Not done yet.

- Request:
  - Get url, queryString, method ✅
  - Get cookie, session, localStorage, bodydata of PACTCH, PUT ❌
- Router:
  - Routing to controller ✅
  - Routing with method ✅
  - Pass params to controller ✅
  - Validate type of params ❓
  - Pass queries to controller ✅
  - Route not found ✅
- Database:
  - Connect and close MySQL database ✅
- Response:
  - Response a json format data ✅
  - Response with status code ❌
  - Response other format data ❌
- View:
  - Render a html file in Views ✅
  - Render a html file in Views with datas ✅
  - Render a template ❌
- Middleware:
  - Register middleware to controller ✅
- Error:
  - if `SHOW_ERRORS = TRUE` in `Config.php`
    - Catch Error and Exception then render error page with stack trace ✅
  - If `SHOW_ERRORS = FALSE` in `Config.php`
    - Render 404 page ✅
    - Render 500 page ❌

---

## Document

### Naming file rule

Files in Controllers folder: `{controllerName}.Controller.php`
Files in Models folder: `{modelName}.Model.php`
Files in Middlewares folder: `{middlewareName}.middleware.php`
Files in View folder: `{viewName}.php`

### Add a new route

Creating a route '/' with 'GET' method call a method index() in Home Controller:

```php
// index.php
<?php
 $app = new App();
 $app->get('/', "Home@index")
?>
```

Getting parameters:

```php
// index.php
<?php
 $app = new App();
 $app->get('products/{id:\d+}', "Product@index")
?>

// Controllers/Product.Controller.php
<?php
namespace Controllers;

class ProductController extends BaseController
{
    private ProductModel $productModel;
    public function register()
    {
        $this->productModel = new ProductModel();
    }

    public function getAll()
    {
        $products = $this->productModel->getAll();
        Response::sendJson($products);
    }
}
?>
```

Using callback function:

```php
// index.php
<?php
 $app = new App();
 $app->get('/', function () {
    View::render("Home");
 })
?>
```

### Example about database connect in model

```php
// Models/Product.php
<?php
namespace Models;

use Core\DB;
use Schemas\Product;

class ProductModel {

    private function convertRowToProduct($row) {
        return new Product($row["id_product"], $row["name"]);
    }

    public function getAll(): array {
        $products = [];
        $sql = "SELECT * FROM product";
        $result = DB::getDB()->execute_query($sql);
        DB::close();
        while ($row = $result->fetch_assoc()) {
            $products[] = $this->convertRowToProduct($row);
        }
        return $products;
    }
}
```

### Render a view by passing array of data

```php
// Controllers/Product.Controller.php
<?php
    namespace Controllers;
    use Models\Product;
    class ProductController extends BaseController {
        public function getProduct(int $id) {
            $product = $this->productModel->getProduct($id);
            View::render("ProductDetail", [
                'product' => $product,
            ]);
        }
    }
?>
```

### Getting post data from request

```php
// Controllers/Product.Controller.php
<?php
    namespace Controllers;
    use Models\ProductModel;
    class ProductController extends BaseController {

        // @POST method
        public function updateProduct(int $id)
        {
            // Check and get require bodydata
            $bodyData = Validate::getBodyData(['name']);

            $product = new Product($id, $bodyData['name']);
            if ($this->productModel->updateProduct($product)) {
                Response::sendJson("Update success");
            } else {
                Response::sendJson("Failed to update product: id = $id", 400);
            }
        }

    }
?>
```

### Using middleware

Creating a middleware

```php
<?php
// Middlewares/auth.middleware.php
namespace Middlewares;

use Core\Middleware;
use Core\Request;
use Core\Response;

class AuthMiddleware extends Middleware {
    // execute a middleware
    public function execute(Request $request) {
        // Do some auth validation
    }
}
```

Registering

```php
<?php
// Controllers/Product.Controller.php
namespace Controllers;

use Core\BaseController;
use Middlewares\AuthMiddleware;

class ProductController extends BaseController
{
    public function register() {
        $this->registerMiddleware(new AuthMiddleware());
    }
}
```

---

## Thư mục root

1. .htaccess

   - File dùng để custom lại url khi chạy trên trình duyệt.

2. Config.php

   - File chứa các thiết lập về database và hiển thị errors.

3. index.php
   - File chính dùng để chạy Server.

---

## Thư mục Core

1. App.php

   - `App` đóng gói các files trong thư mục Core thành 1 class duy nhất.

2. Request.php

   - `Request` dùng để nhận và xử lý các request sau đó gửi cho `Router.php` .

3. Router.php

   - `Router` sẽ điều hướng và gửi dữ liệu đã nhận tới các controller tương ứng với `Route` đó

4. BaseController.php

   - `BaseController` là class cơ sở của các Controller.

5. Database.php

   - `Database` dùng để quản lý kết nối với CSDL.

6. Reponse.php

   - `Response` dùng để gửi dữ liệu lại cho trình duyệt

7. View.php

   - `View` dùng để render các file html trong thư mục Views cùng với các dữ liệu (nếu có)

8. Error.php
   - `Error` là class custom lại thông báo khi có Error hoặc Exception

## Reference

- [PHP-MVC-REST-API](https://github.com/afgprogrammer/PHP-MVC-REST-API)
- [php-mvc](https://github.com/daveh/php-mvc)
