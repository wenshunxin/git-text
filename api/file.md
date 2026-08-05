# 文件上传

## 文件上传

上传图片文件，返回访问 URL。

- **Method**: `POST`
- **URL**: `/api/file/upload`
- **Content-Type**: `multipart/form-data`
- **Auth**: 需要认证

### Query 参数

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| module | string | 否 | common | 模块名（分目录存储） |

### Body 参数（multipart/form-data）

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| file | file(binary) | 是 | 图片文件 |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": {
    "fileName": "a1b2c3d4e5f6.jpg",
    "filePath": "avatar/2025/a1b2c3d4e5f6.jpg",
    "url": "/upload/avatar/2025/a1b2c3d4e5f6.jpg"
  }
}
```

### 响应字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| rtState | boolean | 请求状态，true 表示成功 |
| rtMsg | string | 提示消息，成功时为空字符串 |
| rtData.fileName | string | 存储后的文件名（UUID 命名） |
| rtData.filePath | string | 相对存储路径 |
| rtData.url | string | 访问 URL（前端可直接使用） |
