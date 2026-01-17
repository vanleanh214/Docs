**Cài đặt MinIO Client**

## Cài đặt nhanh Terminal

```bash
wget https://dl.min.io/client/mc/release/linux-amd64/mc

chmod +x mc

sudo mv mc /usr/local/bin/
```

🚀 **Connect to S3**

**Connect Storage S3:**

```bash
mc alias set ALIAS_NAME https://s3.evgcloud.net ACCESS_KEY SECRET_KEY
```
Check connect:

```bash
mc ls ALIAS_NAME
```

⚙️ **Command Line Using**

List bucket:

```bash
mc ls ALIAS_NAME/BUCKET_NAME
```
Create bucket:

```bash
mc mb ALIAS_NAME/BUCKET_NAME
```

Delete bucket:

```bash
mc rb ALIAS_NAME/BUCKET_NAME
```

**Upload/Dowload**

Upload file:

```bash
mc cp /path/file.txt myminio/mybucket/
```

Upload folder:

```bash
mc cp -r /path/folder myminio/mybucket/
```

Download file:

```bash
mc cp myminio/mybucket/file.txt /local/path/
```

Xóa file:

```bash
mc rm myminio/mybucket/file.txt
```

Check size bucket:

```bash
mc du myminio/mybucket
```




