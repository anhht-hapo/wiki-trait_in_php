#Tìm hiểu về Trait trong PHP
##Trait là gì ?

**Trait** có thể hiểu như một class, là nơi tập hợp một nhóm phương thức (*method*) mà chúng ta muốn sử dụng trong class khác. Cũng giống như Abstract Class, chúng ta không thể khởi tạo một đối tượng từ **Trait**.

*Ví dụ 1: Minh họa về Trait*

Giả sử ta có các class sau:
```php
class Database
{
    public function listUsers()
    {
        return "List users!";
    }
}

class Users
{
}

class Report
{

}
```
####Đặt vấn đề :

Nếu ta muốn sử dụng lại method listUsers trong class Database cho class Users hoặc class Report thì làm thế nào ? Trong trường hợp này thì chắc chắn các bạn sẽ nghĩ đến từ khóa **extends** đúng không? Và sẽ viết như thế này:
```php
class Database
{
    public function listUsers()
    {
        return "List users!";
    }
}

class Report extends Database
{
    public function reportUsers()
    {
        $this->listUsers();
    }
}

class Users extends Database
{
    public function index()
    {
        $this->listUsers();
    }
}
```
Nếu trường hợp nào cũng sử dụng **extends** để kế thừa thì rất rối. Vì vậy đây là lúc sử dụng đến **Trait**
```php
trait Database
{
    public function listUsers()
    {
        return "List users!";
    }
}

class Users
{
    use Database;

    public function reportUsers()
    {
        $this->listUsers();
    }
}

class Report
{
    use Database;

    public function index()
    {
        $this->listUsers();
    }
}
```
* Qua ví dụ này chúng ta thấy không cần **extends** nhưng chúng ta vẫn sử dụng được các method listUsers(). Tiện lợi hơn nhiều.
* Để sử dụng trait trong PHP thì các bạn chỉ cần dùng từ khóa use để gọi trait bạn muốn sử dụng trong code của bạn. Sau đó bạn có thể sử dụng các phương thức trong trait mà bạn đã use.
## Sử dụng nhiều trait trong 1 class
```php
trait Database
{
    public function listUsers()
    {
        return "List users!";
    }
}

trait Authenticate
{
    public function authorize()
    {
        return "Authorized!";
    }
}
class Users
{
}
```
####Đặt vấn đề :
Nếu ta muốn sử dụng lại method listUsers trong trait Database và sử dụng method authorize trong trait Authenticate cho class Users thì làm thế nào ? PHP đã hỗ trợ hết. Hãy theo dõi ví dụ dưới đây để hiểu rõ hơn về trường hợp này nào?
```php
trait Database
{
    public function listUsers()
    {
        return "List users!";
    }
}

trait Authenticate
{
    public function authorize()
    {
        return "Authorized!";
    }
}

class Users
{
    use Database, Authenticate;

    public function __construct()
    {
        $this->authorize();
    }

    public function index()
    {
        $this->listUsers();
    }
}
```
* Qua ví dụ trên hẳn là các bạn đã hiểu được cách sử dụng đúng không nào, chúng ta vẫn dùng từ khóa **use** với tên các trait được ngăn cách bằng dấu phẩy.
* Nhìn có vẻ ổn đấy, nhưng thử suy nghĩ thêm một tí nữa, chẳng hạn trong các trait lại có cùng tên method, mình muốn sử dụng method A của class A. Nhưng, trong trait B cũng có method A. Vậy giải quyết như thế nào đây ? Đừng lo lắng các bạn à, PHP cũng tính toán trường hợp này rồi. Để hiểu được cách giải quyết thì các bạn hãy theo dõi ví dụ sau:
```
trait Admin
{
    public function checkLogin()
    {
        return "Check login";
    }

    public function isAdmin()
    {
        return "Ok";
    }
}

trait Authenticate
{
    public function checkLogin()
    {
        return "Check login";
    }

    public function isAdmin()
    {
        return "Ok";
    }
}

class Users
{
}
```
####Đặt vấn đề :
1. Trong class Users tôi muốn sử dụng cả 2 trait.
2. Tôi muốn sử dụng method checkLogin trong trait Authenticate.
3. Tối muốn sử dụng method isAdmin trong trait Admin Để giải quyết các vấn đề trên thì tôi phải làm thế nào? Nhất là khi các method ở 2 trait đều giống nhau! Hãy theo dõi ví dụ dưới đây các bạn sẽ lần lượt solve các problem trên.
```php
trait Admin
{
    public function checkLogin()
    {
        return "Check login";
    }

    public function isAdmin()
    {
        return "Ok";
    }
}

trait Authenticate
{
    public function checkLogin()
    {
        return "Check login";
    }

    public function isAdmin()
    {
        return "Ok";
    }
}

class Users
{
    use Authenticate, Admin
    {
        Authenticate::checkLogin insteadof Admin;
        //Sử dụng method checkLogin ở trait Authenticate thay cho  method checkLogin ở trait Admin
        Admin::isAdmin insteadof Authenticate
        //Sử dụng method isAdmin ở trait Admin thay cho method isAdmin ở trait Authenticate
    }

    public function __construct()
    {
        $this->checkLogin();
        $this->isAdmin();
    }
}
```
* Qua ví dụ lần này thì các bạn đã hiểu về cách solve conflict method giữa các trait khi sử dụng nhiều trait trong 1 class rồi đúng không nào.
* Chúng ta chỉ cần dùng từ khóa insteadof để giải quyết trường hợp này
##Trait lồng nhau
**Đặt vấn đề :**
* Giả sử có 3 trait: trait Authenticate, Admin, Permisson.
* Nếu sử dụng cả 3 trait này vào 1 class thì theo cách thông thường thì dùng từ khóa **use**.
* Câu hỏi đặt ra là có cách gì đơn giản hơn không, muốn dùng 10 trait thì không lẽ use từng trait vào class. Vậy có cách nào đơn giản hơn không ? Cách giải quyết như sau: 
```php
trait Admin
{
    public function checkLogin()
    {
        return "Check login";
    }
}

trait Permission
{
    public function checkPermission()
    {
        return "permission";
    }
}

trait Authenticate
{
    use Admin, Permission;
    //chỉ cần gọi các trait vào bằng từ khóa use là giải quyết xong problem
}
class Users
{
    use Authenticate;

    public function __construct()
    {
        $this->checkLogin();
        $this->isAdmin();
    }
}
```
##Traits khác với Abstract Class thế nào?
**Trait** khác với Abstract Class vì nó không dựa trên sự thừa kế. Tưởng tượng rằng nếu class Post và class Comment phải kế thừa từ một AbstractSocial class. Chúng ta dường như muốn nhiều hơn là chỉ share post và comment lên mạng xã hội. Tuy nhiên việc sử dụng abstract class khiến chúng ta phải xây dựng một mô hình kế thừa hết sức phức tạp như sau:
```php
class AbstractValidate extends AbstractCache {}
class AbstractSocial extends AbstractValidate {}
class Post extends AbstractSocial {}
```
##Traits khác với Interfaces thế nào?
Có thể nói về cơ bản thì **Trait** và **Interface** khá giống nhau về tính chất sử dụng. Cả hai đều không thể sử dụng nếu không có một class được implement cụ thể. Tuy nhiên chúng cũng có những sự khác nhau hết sức rõ rệt.

**Interface** có thể hiểu như một bản "hợp đồng" (nếu sử dụng) chỉ ra rằng: "đối tượng có thể làm việc này", do vậy bạn phải implement nó thì mới sử dụng được. Trong khi đó **Trait** chỉ nói : "đối tượng có khả năng làm việc này".

Ta hãy cùng đi vào một ví dụ cụ thể như sau:
```php
// Interface
interface Sociable {

  public function like();
  public function share();

}

// Trait
trait Sharable {

  public function share($item)
  {
    // share this item
  }

}

// Class
class Post implements Sociable {

  use Sharable;

  public function like()
  {
    //
  }

}
```

Chúng ta có interface **Sociable** chỉ ra rằng đối tượng Post có thể **like()** và **share()**. Trong khi đó Sharable Trait implement phương thức **share()** và phương thức **like()** lại được implement bởi chính class Post.

```php
$post = new Post;

if($post instanceOf Sociable)
{
  $post->share('hello world');
}
```
##Dùng Traits có lợi thế nào?
* Giảm việc lặp code.
* Tránh được việc kế thừa nhiều tầng nhiều lớp khá phức tạp trong tổng thể hệ thống, sẽ khó maintain sau này.
* Định nghĩa ngắn gọn, sau đó có thể đặt sử dụng ở những nơi cần thiết, sử dụng được ở nhiều class cùng lúc.
##Nhược điểm của Traits
* **Trait** có thể tạo ra các class mang quá nhiều trách nhiệm (responsibility). Trait được tạo ra chủ yếu dựa trên tư tưởng "copy and paste" code giữa các class. Chúng ta có thể dễ dành thêm một tập hợp các phương thức vào class thông qua việc sử dụng **Trait**.
* Sử dụng Trait khiến chúng ta khó khăn trong việc xem tất cả các phương thức của một class, do vậy khó để có thể phát hiện được một phương thức bất kỳ có bị trùng lặp hay không.
* **Trait** cảm giác như công cụ hữu ích cho những kẻ lười, khi muốn thêm vào để giải quyết vấn đề ngay lập tức. Thường thì Composition là một kiến trúc ổn hơn cho việc kế thừa hay sử dụng **Trait**.