# Khai Thác Lỗ Hổng Cross-Site Scripting

> VulnLab: Cross-Site Scripting (XSS)

> Thực Hiện: Nguyễn Khánh Hào

> Cập Nhật Lần Cuối: 4/10/2024

# Mục Lục
[1. Basic Reflected](#1.-basic-reflected)

[2. Stored Message](#2.-stored-message)

[3. Basic Dom-Based](#3.-basic-dom-based)

[4. HTML Attribute Manipulation](#4.-html-attribute-manipulation)

[5. Welcome to Our Gallery](#5.-welcome-to-our-gallery)

[6. User Agent](#6.-user-agent)

[7. News](#7.-news)

[8. File Upload](#8.-file-upload)

# Nội Dung

# 1. Basic Reflected


![image](https://github.com/user-attachments/assets/dc63751b-1674-4010-a731-ea0e8d9b653d)

Sau khi access vào lab thì chúng ta có thể thấy được một search bar

![image](https://github.com/user-attachments/assets/db02f5c0-81d9-4ac3-8250-b2b6e2570d01)


Dựa vào hint của đề thì có thể đoán ra ngay là chúng ta sẽ trigger được XSS ở chức năng này, nhưng trước tiên hãy cùng mình kiểm tra source xem những đoạn code được developer xây dựng như thế nào.

![image](https://github.com/user-attachments/assets/b1babd6c-bec8-42d0-9598-4c8355de10a6)


Có thể thấy param `q` được gửi qua method GET không hề validate input và chúng ta có thể hoàn toàn control được biến `$_GET['q']` để trigger XSS.
Chúng ta có thể fuzzing bằng cách nhét các tag html bình thường để test xem có thể đưa untrusted data vào chức năng đó hay không.

![image](https://github.com/user-attachments/assets/3bf2372d-9f02-4db0-b252-ff6b705f7c5a)


Với payload `asd<h1>hello</h1>` thì chúng ta có thể thấy được tag `<h1>` của mình đã được highlight và không hề bị filter.

Final payload: `<script>prompt(2)</script>`

![image](https://github.com/user-attachments/assets/dac964c3-0e7d-4523-9b76-62b04253f5be)

# 2. Stored Message

![image](https://github.com/user-attachments/assets/38a9f5a2-1553-4fdb-84da-8fba8dcdae64)

Đối với lab này chúng ta sẽ focus vào chức năng gửi message.

![image](https://github.com/user-attachments/assets/443697ed-e162-4503-9722-b591fd635ff2)

Sau khi login, trang web có một chức năng gửi message, mình sẽ ngó sơ qua source code để hiểu rõ hơn hành vi của chức năng này là gì.

![image](https://github.com/user-attachments/assets/e8d21469-efa6-4e35-8820-5128643066c5)

Có thể thấy untrusted data nằm ở biến `$_POST['mes']` khi trang web không hề kiểm dữ liệu khi user nhập vào.

Final payload: `<img src=x onerror=alert(2)>`

![image](https://github.com/user-attachments/assets/4cd02986-8c8d-4ef5-a6a7-f8208a580c02)

# 3. Basic Dom-Based

![image](https://github.com/user-attachments/assets/7d53d7cf-0f8e-4cec-8fb9-4e6fff8fa3af)

Sau khi access vào lab thì chúng ta có một chức năng là tính diện tích của một hình tam giác bằng cách nhập vào chiều cao (Height) và đáy (Base).

![image](https://github.com/user-attachments/assets/7ffb9e38-6c4e-40a2-bbd2-83b7af85662a)

![image](https://github.com/user-attachments/assets/f0972404-91a5-446e-9bdd-20d7c60ab70e)

Height và Base được truyền vào biến `$_GET['height']` và `$_GET['base']`. Nó sẽ lấy giá trị từ biến `$strings['alert']` và concatenate vào `ans` và tính ra kết quả của của diện tích tam giác. 

Có thể thấy thuộc tính innerHTML rất dễ bị XSS nếu không xử lý đúng cách, ở đây chúng ta có thể control được `$strings['alert']` và trigger XSS.

Final payload: `;alert(2)`

![image](https://github.com/user-attachments/assets/a2d003b6-597d-490b-adc4-1c8734b61954)

# 4. HTML Attribute Manipulation

![image](https://github.com/user-attachments/assets/2d91f137-c30d-4ffa-a667-c5ac9cfd3a24)

Sau khi access vào lab thì có thể thấy trang web có chức năng tạo vé xem phim cho user. 

![image](https://github.com/user-attachments/assets/77446453-c698-49bd-bcae-33a265af895c)

![image](https://github.com/user-attachments/assets/ab94053f-e564-4f2e-a98c-c90eced2aed3)

Sau khi xem qua source code thì các kí tự như `<` và `>` đều đã bị encode, cụ thể thì `<` sẽ thành `%3C` và `>` là `%3E`.

![image](https://github.com/user-attachments/assets/96d6fdd0-eaf2-4d67-9aa9-1ac2f85c72e8)

Sau khi xác định untrusted data nằm ở biến `$_GET['name']`, thì chúng ta có thể thấy nó sẽ lấy giá trị của param `name` và gán cho biến `$ticketname`, sau đó hàm `encodeB` được gọi để encode các kí tự `<`, `>`.

![image](https://github.com/user-attachments/assets/6d0350aa-f648-4fef-86c9-12ac4cb68a31)

Input của chúng ta ta được đưa vào attribute href của tag `<a>`.

![image](https://github.com/user-attachments/assets/9fa1ac4b-7179-4583-b1c1-46d11dcd63b8)

Final payload: `"onmouseover="alert(document.domain)"`

![image](https://github.com/user-attachments/assets/cc76d939-1802-4a5f-a471-e5c24dd1161e)

# 5. Welcome to Our Gallery

![image](https://github.com/user-attachments/assets/8ff620f5-0f0c-4e29-b768-8e1f35a4d844)

Có thể thấy trang web cho chúng ta chức năng xem ảnh thông qua `$_GET['img']`

![image](https://github.com/user-attachments/assets/718c2913-8ce1-4b7b-8794-acda31c8c5a1)

![image](https://github.com/user-attachments/assets/1235bf3b-faf5-4c44-9589-2da3ec791c31)

Đối với lab này thì chúng ta có thể trigger XSS với payload sau: `" onerror="alert(2)"`

![image](https://github.com/user-attachments/assets/1abc95a1-7454-49ea-89d8-25edbc8c233b)

# 6. User Agent

![image](https://github.com/user-attachments/assets/8c7812ce-190f-44c0-be76-a4a32a3b486e)

Có thể thấy đây là trang web để admin có thể xem user agent của người dùng.

![image](https://github.com/user-attachments/assets/d2b623dc-b455-4b40-80a0-0e6006bf2bdf)

Biến `$_SERVER['HTTP_USER_AGENT']` chứa thông tin về user agent được đưa vào database mà không đề validate input từ phía người dùng.

![image](https://github.com/user-attachments/assets/3d6ad7a0-518e-4d9a-a56b-78033183d8a4)

Để trigger được XSS mình sẽ sử dụng burp suite để capture lại request và modify trường User-Agent.

Final payload: `<script>alert(2)</script>`

![image](https://github.com/user-attachments/assets/bf4544f1-195a-46b9-958c-670f9c34909a)

![image](https://github.com/user-attachments/assets/373df013-08ce-4ca6-bf61-c58555cdaa22)

# 7. News

![image](https://github.com/user-attachments/assets/228b5ace-8064-4d30-858d-214715d56685)

Trang web có chức năng chia sẻ tin tức cho bạn bè.
Input của ta được đưa vào attribute href của tag `<a>`.

![image](https://github.com/user-attachments/assets/84a91eb7-c3a6-4f9e-ac66-660c195f3075)

Có thể thấy biến `$_POST['link']` và `$_POST['title']` đã được hàm `htmlspecialchars()` encode các kí tự nguy hiểm như `<`, `>` để tránh XSS.

![image](https://github.com/user-attachments/assets/c0bb2fc2-b502-4922-bb35-e3455a6075de)

Final payload: `javascript:alert(2)`

![image](https://github.com/user-attachments/assets/8c9a74ea-46e4-4a01-837f-357090988f46)

# 8. File Upload

![image](https://github.com/user-attachments/assets/7f8742bd-617a-4382-b1ca-55e2204fe61e)

Trang web có chức năng upload hình ảnh, chúng ta sẽ review qua source để xem hành vi của chức năng này như thế nào.

![image](https://github.com/user-attachments/assets/dbf42650-a142-4ff5-9798-5734a82a942e)

Biến `$_FILES["image"]["name"]` có thể bị thao túng nếu không kiểm soát chặt chẽ tên của file được upload lên.

Vậy câu hỏi đặt ra là: Nếu chúng ta đặt tên của file là một đoạn javascript thì chuyện gì sẽ xảy ra ???

Do laptop mình không đặt được file name có kí tự như `<`, `>` nên mình sử dụng cách đó là URL encode lại đoạn payload (`"><script>alert(2)</script>`).

![image](https://github.com/user-attachments/assets/a6d36053-5292-4c01-bfad-4efcc5977fad)

Final payload: `%22%3e%3c%73%63%72%69%70%74%3e%61%6c%65%72%74%28%32%29%3c%2f%73%63%72%69%70%74%3e`

![image](https://github.com/user-attachments/assets/b6c835ef-5e54-46de-b861-dde10a6ffb4a)
