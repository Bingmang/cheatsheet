# Linux

## 常用指令

curl发送POST带数组的json

```bash
curl -X POST http://localhost:8000/api -H "Content-Type:application/json" -d '{"account": "username", "password": "secret", "majors": ["1", "2"], "classes": []}'
```

curl上传文件（单个和多个）

```bash
curl -X POST http://127.0.0.1:8000/upload -F "upload=@/Users/ghost/Desktop/pic.jpg" -H "Content-Type: multipart/form-data"
curl -X POST http://127.0.0.1:8000/multi/upload -F "upload=@/Users/ghost/Desktop/pic.jpg" -F "upload=@/Users/ghost/Desktop/journey.png" -H "Content-Type: multipart/form-data"
```
