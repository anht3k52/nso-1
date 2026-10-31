# Tóm tắt dự án NSO TIẾN BỊP

> **Ngày kiểm tra:** 01/11/2026 (Asia/Bangkok)  
> **Phạm vi:** toàn bộ thư mục `D:\TeraBoxDownload\NSO TIẾN BỊP`.

## 1. Kết luận nhanh

Thư mục này không chỉ chứa một ứng dụng đơn lẻ mà là một **bộ sản phẩm game Ngọc Rồng/Ninja School**, gồm:

1. **Web quản trị và cổng người chơi Laravel/PHP** ở thư mục gốc.
2. **Game server Java** ở `NSO_FINAL/` (Maven, tạo file JAR).
3. **Client/source Java legacy** ở `PROJECT 217_LAU/SOURCE_JAVA_217/`.
4. **Tài nguyên Unity/client phụ trợ** ở `PROJECT 217_UNITY/` và thư mục `sv2/`.
5. Nhiều file đóng gói lớn (`.zip`, `.rar`), video, bộ cài và bản sao SQL.

README hiện tại ở thư mục gốc vẫn là README mặc định của Laravel, chưa mô tả hệ thống NSO thực tế. File này là bản tóm tắt kỹ thuật thay thế cho việc đọc nhanh dự án.

## 2. Cấu trúc chính

| Thành phần | Vai trò | Công nghệ/định dạng |
|---|---|---|
| Thư mục gốc (`app`, `config`, `database`, `public`, `resources`, `routes`) | Website, API nội bộ và trang quản trị | Laravel 7, PHP, Blade, MySQL/MongoDB |
| `NSO_FINAL/` | Game server hiện đại | Java 8/16 source target trong POM, Docker/OpenJDK 17, Maven |
| `PROJECT 217_LAU/` | Client/source Java legacy và bộ công cụ build | Java ME/legacy, NetBeans/Ant, ảnh và class đã biên dịch |
| `PROJECT 217_UNITY/` | Tài nguyên/client Unity phụ trợ | Unity package, DLL, runtime .NET, video |
| `sv2/` | Trang HTML rất nhỏ (index) | HTML |
| `nsonew.sql` | Dump CSDL chính | MariaDB/MySQL, khoảng 72 bảng |
| `WEB_FINAL.zip`, `NSO_FINAL.rar`, `PROJECT 217_*.zip` | Bản đóng gói/phát hành hoặc sao lưu | ZIP/RAR |

### Thống kê đã kiểm tra

- Laravel module: **16 module** (`Backend`, `BuffItem`, `Charging`, `Currency`, `GiftCode`, `Giveaways`, `Group`, `Home`, `Item`, `ItemCheck`, `Momo`, `News`, `Payment`, `Player`, `Setting`, `User`).
- Module Laravel có khoảng **23 controller**, **40 model**, **90 Blade view**.
- Các file route trong module có tổng cộng khoảng **165 khai báo `Route::...`**.
- `public/` có khoảng **10.252 file**, chiếm khoảng **455 MB** (phần lớn là asset game/web).
- `NSO_FINAL/src/main/java/`: **270 file Java**, khoảng **77.987 dòng**.
- `PROJECT 217_LAU/SOURCE_JAVA_217/`: khoảng **2.845 file**, gồm source Java, class đã biên dịch, PNG và tài nguyên legacy.

## 3. Web Laravel/PHP

### Công nghệ và dependency

- PHP yêu cầu theo `composer.json`: `^7.2.5`.
- Laravel Framework `^7.24`.
- MySQL qua PDO/Eloquent; có thêm MongoDB qua `jenssegers/mongodb` và `mongodb/mongodb`.
- Gói đáng chú ý: Socialite, CKFinder, reCAPTCHA, Spatie Permission, Yajra DataTables, Guzzle, Laravel UI, phpseclib.
- Frontend dùng Laravel Mix 5, Webpack, Sass, Axios và Lodash (`package.json`).

### Kiến trúc

- `app/Modules/ServiceProvider.php` tự quét các module, nạp `routes.php` và đăng ký thư mục `Views` của từng module.
- `app/Providers/MetronicServiceProvider.php` khởi tạo giao diện/admin Metronic.
- Route web mặc định ở `routes/web.php`; route API mặc định ở `routes/api.php` hiện gần như trống. Phần lớn endpoint được khai báo trong từng module.
- Middleware `admin` chỉ cho phép tài khoản đã đăng nhập có role `Administrator` truy cập khu vực quản trị (`config('app.backend_uri')`, mặc định là `/admin`).
- Provider xác thực dùng model `App\Modules\User\Models\User` và Spatie Permission.

### Các nhóm chức năng chính

- **Home/User:** trang chủ, đăng nhập/đăng ký, hồ sơ, đổi mật khẩu/số điện thoại, kích hoạt tài khoản, tải client, cộng đồng, bảng xếp hạng.
- **Backend:** dashboard, server, CPU usage, thao tác quản trị và chuyển dữ liệu.
- **Player/Item/ItemCheck:** quản lý nhân vật, lịch sử, vật phẩm, kiểm tra item và cập nhật dữ liệu người chơi.
- **Charging/Payment:** nạp thẻ/ATM, phí nạp, lịch sử, quy đổi, chuyển tiền, gift code và quản lý giao dịch.
- **Momo:** quản lý tài khoản/đơn hàng MoMo, cron đồng bộ MoMo/Vietcombank/Thesieure và endpoint ví.
- **BuffItem/GiftCode/Giveaways:** gửi item/tiền, gift code, sự kiện/quà tặng.
- **News/Group/Setting/Currency:** tin tức, nhóm, cấu hình game và tiền tệ.

### Dữ liệu

`nsonew.sql` tạo database `nsonew` và khoảng **72 bảng**. Có thể chia thành:

- **Tài khoản/phân quyền:** `users`, `groups`, `roles`, `permissions`, các bảng liên kết role/permission, log đăng nhập.
- **Kinh tế/thanh toán:** `currencies`, `transactions`, `payments`, `deposit`, `chargings`, `chargings_telco`, `charging_fees`, `momo_accounts`, `gift_codes`, `giveaways`, `quydoi`, `ctv_money`...
- **Dữ liệu game:** `players`, `clan`, `clan_member`, `item`, `item_option`, `servers`, `stores`, `task`, `skill`, `map`, `npc`, `monster`, `effect`, `event_points`, `ranking_list`...
- **Dữ liệu phụ trợ:** `news`, `comments`, `settings`, `options`, `others`, `napcard`, `napatm`, `sql_backup`...

> **Lưu ý:** số lượng bảng trong dump lớn hơn nhiều so với các migration hiện có (chỉ khoảng 7 migration nghiệp vụ). Vì vậy, dump SQL hiện là nguồn dữ liệu/khởi tạo đầy đủ hơn; không nên chỉ chạy `php artisan migrate` rồi giả định hệ thống đã đủ bảng.

## 4. Game server Java (`NSO_FINAL`)

### Thành phần

- Maven project với artifact `NinjaSchool`, final name `Nso`.
- Main class: `com.nsoz.server.NinjaSchool`.
- Các package chính: `server`, `network`, `netty`, `model`, `map`, `mob`, `npc`, `item`, `skill`, `effect`, `event`, `clan`, `store`, `task`, `db`...
- Có HikariCP cho MySQL và driver MongoDB; Netty/socket cho network; Quartz cho lịch; Log4j/SLF4J cho log.
- `Data/` chứa dữ liệu game; `target/` đã có artifact build trong bản sao hiện tại.

### Luồng khởi động

`NinjaSchool.main()` đọc `config.properties`, khởi tạo kết nối SQL, tùy chọn kết nối MongoDB, load NPC/mob/map/task/item/skill/effect/store/event/ranking/clan, sau đó khởi động server.

### Cổng và cấu hình mặc định được khai báo

- Game server: `14444` (`config.properties`).
- WebSocket: `8020`.
- RMI: `10000`.
- MySQL: `3306` qua `mysql.properties`.
- MongoDB có thể bật bằng `mongodb.bat=true`; code có cơ chế xóa history MongoDB cũ hơn 7 ngày.

### Build/chạy dự kiến

```text
cd NSO_FINAL
mvn clean package
java -server -jar -Dfile.encoding=UTF-8 -Xms2G -Xmx2G target/Nso-jar-with-dependencies.jar
```

Dockerfile dùng Maven 3.8.4/OpenJDK 17, copy `Data/`, `config.properties`, `mysql.properties` và expose cổng `14444`.

## 5. Client/source legacy và Unity

### `PROJECT 217_LAU`

- Có `SOURCE_JAVA_217/`, thư mục build/preprocessed, source `.java`, nhiều `.class`, `.png`, `.jad`, `.jar`, `.properties`.
- Đây là code client Java legacy đã qua nhiều bước build/biên dịch hoặc decompile; tên class/biến ở một số file đã bị rút gọn/obfuscate.
- Có sẵn NetBeans/Ant metadata và các bộ cài JDK/JRE/NetBeans/WTK, không nên đưa các bộ cài này vào source control.

### `PROJECT 217_UNITY`

- Chứa `ninja.unitypackage`, DLL/runtime Windows, video HD và thư mục `String-En`.
- Không phải một Unity project đầy đủ theo cấu trúc chuẩn (không thấy bộ `Assets/`, `ProjectSettings/`, `Packages/` hoàn chỉnh trong bản kiểm tra), nên cần xác định đây là asset/bản build phụ trợ trước khi chỉnh sửa.

## 6. Kiểm tra đã thực hiện và giới hạn

- Đã kiểm tra cây thư mục, module, route, controller/model/view, `composer.json`, `package.json`, cấu hình database, migration, dump SQL, POM và các file chạy Docker/batch.
- **Chưa chạy được Laravel/Artisan:** máy kiểm tra không có lệnh `php` và `composer` trong `PATH`.
- **Chưa build Maven:** máy có JDK 17 nhưng không có lệnh `mvn` trong `PATH`; do đó chưa xác nhận compile/runtime.
- Bộ test Laravel hiện chỉ là `ExampleTest` mặc định và một unit test `assertTrue`; chưa có test nghiệp vụ đáng kể.

## 7. Vấn đề/rủi ro cần xử lý

### Bảo mật

1. Các file `.env`, `NSO_FINAL/config.properties`, `NSO_FINAL/mysql.properties` và dump SQL có thể chứa thông tin kết nối/secret. Cần **đổi toàn bộ mật khẩu/token** nếu các file này từng được chia sẻ hoặc commit.
2. Không commit `.env`, dump dữ liệu thật, log, file build và archive lớn lên repository công khai.
3. `APP_URL` trong `.env` đang có dạng chưa hoàn chỉnh (`https://`); cần đặt URL thật khi triển khai.

### Tính nhất quán mã nguồn

1. `app/Modules/ServiceProvider.php` đăng ký validator `twofactor` trỏ tới `App\Modules\TwoFactor\Controllers\FTwoFactor`, nhưng không thấy module `TwoFactor` trong cây module hiện tại. Nếu rule này được dùng, request có thể lỗi runtime.
2. `app/Modules/Momo/Routes/` có các file route riêng (`api.php`, `web.php`, `backend.php`) nhưng module loader chỉ tự nạp `app/Modules/*/routes.php`. Cần xác minh các route riêng này có được include ở nơi khác hay đang bị bỏ quên.
3. Một số route của `Group` và `Home` có cấu trúc frontend tương tự nhau; cần kiểm tra trùng endpoint/name khi bật đầy đủ ứng dụng.
4. Migration không phản ánh đầy đủ schema trong `nsonew.sql`; quy trình deploy phải quy định rõ dùng migration hay import dump.

### Khả năng bảo trì

- README gốc chưa có hướng dẫn cài đặt, kiến trúc hay thông tin vận hành.
- Tên file có typo (`BuffIemController.php`) và mã nguồn có dấu hiệu copy/ghép nhiều phiên bản.
- Có nhiều archive/video/bộ cài trong workspace, làm tăng kích thước và gây nhầm lẫn giữa source, build và backup.

## 8. Quy trình cài đặt đề xuất

### Web Laravel

1. Cài PHP tương thích Laravel 7, Composer, extension PDO MySQL và extension MongoDB nếu dùng MongoDB.
2. Chạy `composer install` trong thư mục gốc.
3. Tạo `.env` riêng cho môi trường, đặt `APP_KEY`, URL, MySQL (`nsonew`) và MongoDB tùy chọn.
4. Import `nsonew.sql` vào MariaDB/MySQL (sao lưu trước khi import).
5. Chạy `php artisan key:generate`, `php artisan storage:link` nếu cần và kiểm tra quyền ghi `storage/`/`bootstrap/cache/`.
6. Cài Node.js/npm, chạy `npm install` rồi `npm run development` hoặc `npm run production`.
7. Trỏ web server (Apache/Nginx) vào thư mục `public/`, không trỏ document root vào thư mục gốc dự án.

### Game server Java

1. Cài JDK 17 và Maven.
2. Kiểm tra `config.properties`, `mysql.properties`, dữ liệu `Data/`, cổng firewall và database.
3. Build bằng Maven, sau đó chạy JAR với heap phù hợp (file mẫu dùng `-Xms2G -Xmx2G`).
4. Chỉ bật MongoDB nếu đã kiểm tra dữ liệu, credential và nhu cầu vận hành.

## 9. Checklist đề xuất tiếp theo

- [ ] Tách source web, game server, client legacy và asset thành các repository/thư mục phát hành riêng.
- [ ] Thu hồi/đổi secret xuất hiện trong cấu hình và dump đã chia sẻ.
- [ ] Bổ sung `.env.example` không chứa secret thật.
- [ ] Quyết định một nguồn schema duy nhất: migration đầy đủ hoặc SQL dump versioned.
- [ ] Sửa/kiểm tra module `TwoFactor`, route Momo và route trùng.
- [ ] Cập nhật README với hướng dẫn web/server/client.
- [ ] Thêm smoke test cho login, admin middleware, nạp tiền, gift code, callback thanh toán và route cron.
- [ ] Thiết lập CI: PHP/Composer test, lint, Maven build và kiểm tra secret trước khi commit.

