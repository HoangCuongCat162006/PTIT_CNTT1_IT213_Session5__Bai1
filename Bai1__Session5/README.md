# BÁO CÁO BÀI TẬP: BÀI 1 - KHÔI PHỤC COMMIT ĐÃ MẤT BẰNG GIT REFLOG

- **Họ và tên học viên:** Hoàng Cường Cát
- **Email:** cathoangfb1@gmail.com
- **Khóa học:** IT_209 - Quản lý phiên bản mã nguồn với Git & GitHub
- **Session:** Session 05 - Nâng cao với Git (Advanced Git Techniques)
- **Bài tập:** Bài 1 (ex1)
- **Đường dẫn nộp bài trên GitHub:** `homework/session_05/ex1/`

---

## 1. MỤC TIÊU BÀI HỌC
- Hiểu rõ cơ chế Git lưu trữ lịch sử tham chiếu cục bộ thông qua **Reflog** (Reference Logs).
- Sử dụng thành thạo lệnh `git reflog` để truy vết, xác định mã hash của commit đã bị tách rời/mất dấu (*dangling commit / orphaned commit*).
- Khôi phục thành công commit và toàn bộ mã nguồn quan trọng sau sự cố chạy nhầm lệnh `git reset --hard` mà không cần viết lại mã thủ công.

---

## 2. BỐI CẢNH & YÊU CẦU ĐỀ BÀI
- **Bối cảnh sự cố:** Trong quá trình làm việc cục bộ, người dùng vô tình thực thi lệnh:
  ```bash
  git reset --hard HEAD~1
  ```
  Lệnh này đã lùi con trỏ `HEAD` và nhánh hiện tại về commit trước đó, đồng thời xóa sạch thay đổi trong Working Directory và Staging Area. Commit chứa tính năng quan trọng vừa phát triển hoàn toàn biến mất khỏi `git log`.
- **Ràng buộc:**
  - Tuyệt đối không được viết lại code thủ công.
  - Bắt buộc phải sử dụng cơ chế **Git Reflog** để tìm lại và phục hồi trạng thái commit cũ.

---

## 3. CÁC BƯỚC TIẾN HÀNH THỰC TẾ

### Bước 1: Khởi tạo dự án thử nghiệm và tạo commit tính năng
1. Khởi tạo repository Git cục bộ:
   ```bash
   git init
   ```
2. Tạo commit ban đầu để thiết lập nhánh chính:
   ```bash
   echo "Khoi tao du an ban dau" > init.txt
   git add init.txt
   git commit -m "Khoi tao du an"
   ```
   *Kết quả:* Tạo thành công commit gốc `35b39c4`.

3. Tạo tính năng quan trọng theo yêu cầu đề bài:
   - Tạo file `feature.txt` với nội dung `Day la tinh nang quan trong`.
   - Tiến hành stage và commit:
   ```bash
   echo "Day la tinh nang quan trong" > feature.txt
   git add .
   git commit -m "Them tinh nang quan trong"
   ```
   *Kết quả terminal:*
   ```text
   [master f4375d0] Them tinh nang quan trong
    1 file changed, 1 insertion(+)
    create mode 100644 feature.txt
   ```

4. Kiểm tra lịch sử commit trước khi xảy ra sự cố:
   ```bash
   git log --oneline
   ```
   *Output:*
   ```text
   f4375d0 (HEAD -> master) Them tinh nang quan trong
   35b39c4 Khoi tao du an
   ```

---

### Bước 2: Giả lập sự cố vô tình chạy `git reset --hard HEAD~1`
Mô phỏng thao tác sai lầm của lập trình viên:
```bash
git reset --hard HEAD~1
```
*Kết quả terminal:*
```text
HEAD is now at 35b39c4 Khoi tao du an
```

---

### Bước 3: Kiểm tra và xác nhận commit cùng mã nguồn đã biến mất
1. Kiểm tra lại lịch sử commit:
   ```bash
   git log --oneline
   ```
   *Output:*
   ```text
   35b39c4 (HEAD -> master) Khoi tao du an
   ```
   > **Nhận xét:** Commit `f4375d0 - Them tinh nang quan trong` đã biến mất hoàn toàn khỏi cây lịch sử commit thông thường của nhánh `master`.

2. Kiểm tra thư mục làm việc (Working Tree):
   ```bash
   dir /b
   # hoặc lệnh PowerShell: Get-ChildItem
   ```
   *Output:*
   ```text
   init.txt
   ```
   > **Nhận xét:** File `feature.txt` đã bị xóa sạch khỏi thư mục làm việc.

---

### Bước 4: Dùng `git reflog` để truy vết commit đã mất
Git duy trì một nhật ký đặc biệt ghi lại mọi sự di chuyển của con trỏ `HEAD` ở máy cục bộ mang tên **Reflog**.

Thực thi lệnh:
```bash
git reflog
```

*Kết quả thực tế hiển thị trên terminal:*
```text
35b39c4 HEAD@{0}: reset: moving to HEAD~1
f4375d0 HEAD@{1}: commit: Them tinh nang quan trong
35b39c4 HEAD@{2}: commit (initial): Khoi tao du an
```

#### Phân tích nhật ký Reflog:
- `HEAD@{0}`: Hành động vừa thực hiện gần nhất là `reset: moving to HEAD~1`, trỏ về commit `35b39c4`.
- `HEAD@{1}`: Trạng thái trước khi thực hiện reset chính là commit `f4375d0` với thông điệp `"Them tinh nang quan trong"`.
- Như vậy, commit mã nguồn quan trọng vẫn còn tồn tại trong kho đối tượng của Git với mã SHA-1 là: **`f4375d0`**.

---

### Bước 5: Thực hiện khôi phục commit

Có 2 phương pháp khôi phục tùy theo nhu cầu:

#### Cách 1: Khôi phục trực tiếp nhánh hiện tại về commit cũ (Khuyên dùng theo đề bài)
Thực hiện lệnh `git reset --hard` trỏ trực tiếp vào mã commit hash tìm được từ reflog:
```bash
git reset --hard f4375d0
# Hoặc sử dụng định danh tham chiếu: git reset --hard HEAD@{1}
```

*Kết quả terminal:*
```text
HEAD is now at f4375d0 Them tinh nang quan trong
```

#### Cách 2: Tạo một nhánh mới từ commit đã mất (Cách an toàn trong dự án lớn)
Nếu không muốn tác động trực tiếp lên nhánh hiện tại, ta có thể tạo nhánh mới giữ commit này:
```bash
git checkout -b recovery-feature f4375d0
```

---

### Bước 6: Kiểm tra và xác nhận kết quả khôi phục
1. Kiểm tra lại lịch sử commit bằng `git log --oneline`:
   ```bash
   git log --oneline
   ```
   *Output:*
   ```text
   f4375d0 (HEAD -> master) Them tinh nang quan trong
   35b39c4 Khoi tao du an
   ```
   > Commit `"Them tinh nang quan trong"` đã trở lại đầu nhánh `master`.

2. Kiểm tra thư mục làm việc và nội dung file:
   ```bash
   Get-ChildItem
   Get-Content feature.txt
   ```
   *Output:*
   ```text
   Mode                 LastWriteTime         Length Name
   ----                 -------------         ------ ----
   -a----         10/4/2026   9:51 PM             29 feature.txt
   -a----         10/4/2026   9:51 PM             24 init.txt

   Day la tinh nang quan trong
   ```
   > File `feature.txt` cùng toàn bộ nội dung nguyên vẹn đã được phục hồi thành công 100%.

---

## 4. BẢN CHẤT CƠ CHẾ HOẠT ĐỘNG CỦA GIT REFLOG
1. **Khác biệt cốt lõi giữa `git log` và `git reflog`:**
   - `git log`: Hiển thị lịch sử duyệt đồ thị commit theo nhánh hiện tại (commit graph traversal). Nếu một commit không còn nhánh hoặc tag nào tham chiếu tới, nó sẽ bị ẩn khỏi `git log`.
   - `git reflog`: Ghi lại nhật ký thay đổi cục bộ của các tham chiếu (`HEAD`, nhánh). Bất kể bạn commit, checkout, rebase, merge hay reset, Git đều ghi một dòng nhật ký vào Reflog.
2. **Cơ chế lưu trữ đối tượng (Object Store) và Garbage Collection (`git gc`):**
   - Lệnh `git reset --hard` chỉ đơn thuần thay đổi con trỏ của nhánh và dọn working tree, chứ **không lập tức xóa dữ liệu của commit**.
   - Commit bị mất trở thành *dangling commit* (commit mồ côi). Git vẫn bảo lưu các commit này trong cơ sở dữ liệu đối tượng tối thiểu từ 30 đến 90 ngày (theo cấu hình `gc.reflogExpire`) trước khi trình dọn rác tự động của Git (`git gc`) tiến hành thu hồi.
3. **Bài học kinh nghiệm:**
   - Trong Git cục bộ, gần như mọi thứ đã từng commit đều có thể khôi phục nhờ `git reflog`.
   - Trước khi thực hiện các lệnh nguy hiểm như `reset --hard`, luôn đảm bảo đã nắm vững cách cứu dữ liệu bằng `reflog`.
