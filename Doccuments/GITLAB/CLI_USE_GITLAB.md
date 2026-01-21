##MOT SO LENH CO BAN KHI SU DUNG GITLAB
**1. Nhóm khoi tao & ket noi**

```bash
*git init*: Bien thư mục hiện tại thành một Repo Git.

- Khi nào: Khi bạn bat đau một dự án mới hoàn toàn o máy cục bộ.
```

```bash
*git clone [URL]:* Tai một dự án từ server (GitLab/GitHub) ve máy.

- Khi nào: Khi bạn muon lay code cua team ve làm tiep.
```

```bash
*git remote add origin [URL]:* Ket noi Repo local máy với Repo trên server.

- Khi nào: Sau khi git init, bạn can lệnh này đe biet sẽ "đay" code đi đâu.
```

**2. Nhóm thao tác thay đoi (Quan trọng)**

```bash
*git add .* Đánh dau tat ca các file đã sua đe chuan bị lưu.

- Khi nào: Sau khi bạn viet code xong và muon lưu lại "ban nháp" này.
```

```bash
*git commit -m "mess"* : Lưu chính thức các thay đoi vào lịch su.

- Khi nào: Sau khi add, giúp bạn đặt tên cho phiên ban vừa sua đe sau này de tìm lại.
```

```bash
*git status* : Kiem tra xem file nào đã sua, file nào chưa add.

- Khi nào: Dùng liên tục đe biet mình đang đứng o đâu, tránh commit nhom file.
```

**3. Nhóm huy**

```bash
*git checkout -- [file]* : Huy bo các thay đoi vừa gõ, đưa file ve trạng thái cũ.

- Khi nào: Khi bạn lỡ tay sua sai và muon quay lại lúc chưa sua.
```

```bash
*git fetch*: Chi tai thông tin mới từ server ve nhưng không gộp.

- Khi nào: Khi bạn muon xem server có gì mới mà chưa muon thay đoi code đang viet.
```

