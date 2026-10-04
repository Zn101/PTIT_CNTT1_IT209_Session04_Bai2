# Bài 2: Quản lý nhánh và Giải quyết xung đột (Merge Conflict)

**Session 04 – Exercise 2** · Đường dẫn nộp bài: `homework/session_04/ex2/`

> Dòng mô tả dự án (đã giải quyết xung đột): Dự án quản lý mã nguồn bằng Git – phiên bản ổn định trên main, bổ sung tính năng cập nhật từ feature-update.

## 1. Mục tiêu

- Tạo mới và di chuyển giữa các nhánh cục bộ.
- Cố ý tạo ra Merge Conflict và xử lý thủ công.
- Hoàn thành merge commit và hiểu cơ chế 3-Way Merge.

## 2. Các lệnh đã thực hiện

### Bước 1 – Chuẩn bị repository và commit gốc trên `main`

```bash
mkdir session04-ex2
cd session04-ex2
git init
echo "Dòng mô tả dự án: phiên bản ban đầu" > README.md
git add README.md
git commit -m "Initial commit: add README.md"
```

### Bước 2 – Tạo nhánh `feature-update` và sửa dòng mô tả

```bash
git switch -c feature-update
echo "Dòng mô tả dự án: bổ sung tính năng cập nhật từ feature-update" > README.md
git add README.md
git commit -m "feature: update project description"
```

### Bước 3 – Quay lại `main` và sửa cùng dòng đó

```bash
git switch main
echo "Dòng mô tả dự án: phiên bản ổn định trên main" > README.md
git add README.md
git commit -m "main: update project description"
```

Hai nhánh đã sửa **cùng một dòng** của `README.md` theo hai cách khác nhau.

### Bước 4 – Gộp nhánh và gặp xung đột

```bash
git merge feature-update
```

Kết quả:

```text
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

## 3. Các bước giải quyết xung đột thủ công

**3.1. Xác định tệp bị xung đột**

```bash
git status
```

Git báo `both modified: README.md` trong mục *Unmerged paths*.

**3.2. Mở tệp và đọc các ký hiệu đánh dấu**

```text
<<<<<<< HEAD
Dòng mô tả dự án: phiên bản ổn định trên main
=======
Dòng mô tả dự án: bổ sung tính năng cập nhật từ feature-update
>>>>>>> feature-update
```

| Ký hiệu | Ý nghĩa |
| --- | --- |
| `<<<<<<< HEAD` | Bắt đầu phần nội dung của nhánh hiện tại (`main`). |
| `=======` | Ranh giới giữa hai phiên bản. |
| `>>>>>>> feature-update` | Kết thúc phần nội dung của nhánh được gộp vào. |

**3.3. Sửa tệp bằng tay**

Xóa cả ba dòng ký hiệu, giữ lại nội dung kết hợp từ hai nhánh:

```text
Dòng mô tả dự án: phiên bản ổn định trên main, bổ sung tính năng cập nhật từ feature-update
```

**3.4. Kiểm tra không còn ký hiệu xung đột**

```bash
grep -nE '^(<<<<<<<|=======|>>>>>>>)' README.md
```

Không in ra dòng nào nghĩa là tệp đã sạch.

**3.5. Đánh dấu đã giải quyết và hoàn tất merge commit**

```bash
git add README.md
git commit -m "Merge branch 'feature-update': resolve conflict in README.md"
```

## 4. Kiểm tra kết quả

```bash
git log --graph --oneline
```

Kết quả:

```text
<!-- DÁN KẾT QUẢ THỰC TẾ CỦA BẠN VÀO ĐÂY, ví dụ:
*   e4f5a6b (HEAD -> main) Merge branch 'feature-update': resolve conflict in README.md
|\
| * c3d4e5f (feature-update) feature: update project description
* | b2c3d4e main: update project description
|/
* a1b2c3d Initial commit: add README.md
-->
```

Ảnh chụp màn hình:

![Đồ thị nhánh từ git log --graph --oneline](./screenshot.png)

## 5. Phân tích cơ chế 3-Way Merge

Hai nhánh đã rẽ ra khác nhau nên Git không thể Fast-Forward mà phải so sánh **3 phiên bản**:

| Phiên bản | Commit | Vai trò |
| --- | --- | --- |
| Base | `Initial commit` | Tổ tiên chung gần nhất của hai nhánh. |
| Ours | `main: update project description` | Nhánh hiện tại (`HEAD`). |
| Theirs | `feature: update project description` | Nhánh được gộp vào. |

- Dòng nào chỉ một bên thay đổi so với Base → Git tự lấy thay đổi đó.
- Dòng nào cả hai bên cùng thay đổi khác nhau → Git không tự quyết định được, sinh ra xung đột và chèn ký hiệu đánh dấu.
- Sau khi giải quyết, Git tạo **merge commit có 2 commit cha** (một từ `main`, một từ `feature-update`), thể hiện bằng hai đường nhập lại trên đồ thị.

## 6. Kết luận

- Đã tạo và chuyển đổi giữa hai nhánh `main` và `feature-update`.
- Đã tạo xung đột trên cùng một dòng của `README.md` và xử lý thủ công các ký hiệu `<<<<<<<`, `=======`, `>>>>>>>`.
- Lịch sử hiển thị merge commit với 2 nhánh tổ tiên.
