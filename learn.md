# WeChatMsgArchive 使用指南

## 前置条件

- Windows 系统（不支持 Mac/Linux）
- PC 版微信已安装并**正在运行**
- Python 3.10+
- 建议以管理员权限运行（读取内存需要）

---

## 操作步骤

### 第一步：安装依赖

```bash
pip install -r requirements.txt
```

### 第二步：解密微信数据库

运行 `example/1-decrypt.py`，它会自动：
1. 检测微信版本（3.x 或 4.0）
2. 从微信进程内存中提取加密密钥
3. 解密数据库文件

解密结果会输出到：
- 微信 3.x：`./wxid_xxx/Msg/` 目录
- 微信 4.0：`./wxid_xxx/db_storage/` 目录

### 第三步：查找好友的用户名

运行 `example/2-contact.py`，会列出所有联系人（用户名、昵称、备注）。找到你要导出的那个好友的 `username`（通常是 `wxid_xxx` 格式）。

需要修改文件中的 `db_dir` 指向第二步解密后的目录，并设置 `db_version`（3 或 4）。

### 第四步：导出聊天记录

运行 `example/3-exporter.py`，修改以下关键参数：

```python
from wxManager import DatabaseConnection
from exporter import HtmlExporter
from exporter.config import FileType

# 指向解密后的数据库目录
db_dir = './wxid_xxx/db_storage/'
db_version = 4  # 或 3，取决于你的微信版本

conn = DatabaseConnection(db_dir, db_version=db_version)
database = conn.get_interface()

# 填入第三步找到的好友用户名
contact = database.get_contact_by_username('wxid_xxxxxxxx')

exporter = HtmlExporter(
    database,
    contact,
    output_dir='./data/',      # 输出目录
    type_=FileType.HTML,
    message_types=None,         # None = 导出所有消息类型
    time_range=['2020-01-01 00:00:00', '2026-12-31 23:59:59'],  # 时间范围
)
exporter.start()
```

---

## 支持的导出格式

| 格式 | 特点 |
|------|------|
| **HTML** | 最完整，支持图片、视频、语音、表情、引用、合并转发等 |
| TXT | 纯文本 |
| CSV/XLSX | 适合数据分析 |
| DOCX | Word 文档 |
| JSON | 适合 AI 训练 |
| Markdown | Markdown 格式 |

推荐用 **HTML** 格式，还原效果最好。

---

## 注意事项

1. **微信必须在运行**才能完成第二步的密钥提取
2. 不能恢复已删除的消息
3. 仅限个人使用，不得用于非法用途
