# Bài 4 - Quản lý tệp tin bỏ qua (.gitignore) và Sửa lịch sử (Amend)

## 1. Mục tiêu

- Cấu hình tệp tin ẩn `.gitignore` để Git tự động bỏ qua file nhạy cảm (`credentials.txt`).
- Gỡ file đã commit ra khỏi cache theo dõi của Git bằng `git rm --cached` mà **không xóa file vật lý** trên ổ đĩa.
- Sửa thông điệp của commit gần nhất bằng `git commit --amend`.

## 2. Tạo file credentials.txt

Tạo file `credentials.txt` để mô phỏng file chứa thông tin bảo mật:

```text
username=admin
password=123456
```

Commit đầu tiên đã vô tình đưa file nhạy cảm này vào Git:

```bash
git add credentials.txt
git commit -m "Add project credentials"
```

## 3. Cấu hình .gitignore

Tạo file `.gitignore` ở thư mục gốc dự án với nội dung:

```text
credentials.txt
```

Từ đây Git sẽ tự động bỏ qua `credentials.txt`: không theo dõi, không báo trạng thái thay đổi và `git add` mặc định sẽ không thêm file này nữa.

## 4. Gỡ file khỏi cache bằng git rm --cached

### Lệnh thực hiện

```bash
git rm --cached credentials.txt
```

### Giải thích cách gỡ file khỏi cache

`git rm --cached` chỉ xóa file khỏi **index** (vùng theo dõi tạm thời của Git), không đụng đến file trên ổ đĩa:

| Thao tác | `git rm --cached` | `rm` / `del` |
|---|---|---|
| Gỡ khỏi index (ngừng theo dõi) | Có | Có (sau khi commit) |
| Xóa file vật lý trên ổ đĩa | **Không** | Có |

Các bước hoàn tất gỡ file (ràng buộc: không xóa file vật lý, giữ lại trong thư mục làm việc cục bộ):

```bash
git rm --cached credentials.txt   # gỡ khỏi index, file vẫn còn trên đĩa
git add .gitignore                # Git bỏ qua file trong tương lai
git commit -m "Remove sensitive credentials and update gitignore"
```

Kiểm tra file đã không còn được theo dõi nhưng vẫn còn trên đĩa:

```bash
$ git ls-files
.gitignore
README.md
Screenshot 2026-10-02 125122.png

$ ls -l credentials.txt
-rwxrwxrwx 1 th_phong th_phong 31 Oct  2 12:09 credentials.txt
```

→ `credentials.txt` biến mất khỏi danh sách theo dõi của Git, nhưng file vẫn tồn tại nguyên vẹn trên ổ đĩa.

## 5. Sửa lịch sử bằng git commit --amend

`git commit --amend` thay thế commit gần nhất bằng một commit mới (sửa message và/hoặc nội dung) mà **không tạo commit mới** trong lịch sử:

```bash
git commit --amend -m "Remove sensitive credentials and update gitignore"
```

- Thông điệp ban đầu: `Remove credentials from tracking`
- Thông điệp sau khi amend: `Remove sensitive credentials and update gitignore`

Bằng chứng từ `git reflog` (commit cũ bị thay thế bằng thao tác `commit (amend)`):

```text
a47498d HEAD@{0}: commit (amend): Remove sensitive credentials and update gitignore
b4f0917 HEAD@{5}: commit: Remove credentials from tracking
```

## 6. Kết quả kiểm tra (ghi lại tại thời điểm thực hiện)

> Lưu ý: SHA của commit sẽ thay đổi mỗi lần chạy `--amend` — đó chính là hành vi thay thế commit, nên giá trị hash bên dưới chỉ mang tính minh họa tại thời điểm quan sát.

### `git status`

```text
On branch master
nothing to commit, working tree clean
```

→ `credentials.txt` **không xuất hiện** ở bất kỳ mục nào (không Staged, không Modified, không Untracked).

### `git status --ignored`

```text
On branch master
Ignored files:
  (use "git add -f <file>..." to include in what will be committed)
	credentials.txt

nothing to commit, working tree clean
```

→ `credentials.txt` nằm trong nhóm **Ignored** — Git chủ động bỏ qua file nhạy cảm này.

### `git log -n 1` (chứng minh đã dùng `--amend`)

```text
commit 29104863e7969853d37c649dd1d7605b01a36ecb
Author: La The Phong <latp1102@gmail.com>
Date:   Fri Oct 2 12:14:25 2026 +0700

    Remove sensitive credentials and update gitignore
```

→ Commit gần nhất hiển thị thông điệp **đã sửa đổi thành công** bằng `--amend` (ban đầu là `Remove credentials from tracking`).

### Toàn bộ lịch sử commit

```text
2910486 Remove sensitive credentials and update gitignore
d36a213 Add project credentials
```

## 7. Các file nộp bài

- `.gitignore` — cấu hình bỏ qua `credentials.txt`
- `README.md` — báo cáo giải thích `git rm --cached` và kết quả `git log -n 1`
- `Screenshot 2026-10-02 125122.png` — ảnh chụp `git status` (không còn theo dõi `credentials.txt`) và `git log -n 1` (thông điệp đã sửa bằng `--amend`)
