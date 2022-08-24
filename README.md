# Laravel Testing Conventions

## Cấu trúc thư mục
- Tất cả **Unit Tests** **PHẢI** được đặt trong thư mục `tests/Unit`
  ```phpt
    php artisan make:test Models/UserTest --unit
  ```
- Tất cả **Feature Tests** **PHẢI** được đặt trong thư mục `tests/Feature`
  ```phpt
    php artisan make:test Auth/LoginTest
  ```
- Nội dung bên trong thư mục `Unit` hay `Feature` **PHẢI** có cấu trúc giống với cấu trúc bên trong thư mục `app`. Ví dụ như Unit Test cho file `app/Models/User.php` PHẢI được viết bên trong file `tests/Unit/Models/UserTest.php`

## Quy tắc đặt tên
- Toàn bộ test file **PHẢI** có namespace riêng. Namespace PHẢI được bắt đầu với `Tests\` (hoặc `ASampleProjectTests\`), và tiếp đó là cấu trúc thư mục. Ví dụ, namespace cho file `tests\Unit\Models\UserTest.php` PHẢI là `Tests\Unit\Models\UserTest` (hoặc `ASampleProjectTests\Unit\Models\UserTest`)
- Để đảm bảo tính dễ đọc, nên sử dụng `snake_case` khi viết tên cho các hàm test. Do một hàm test phải bắt đầu bằng `test`, thế nên phần comment phải có `@test`:
    ```phpt
        /** @test */
        public function user_can_view_login()
        {
            //...
        }
    ```
- Những hàm không làm nhiệm vụ test có thể tuân theo quy tắc đặt tên của PSR-2, tức dùng `camelCase` để đặt tên.

## [Yêu cầu trong Feature Test](FeatureTest)


