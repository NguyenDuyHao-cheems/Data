---
description: Đọc toàn bộ codebase thực tế, so sánh với nội dung các file docs hiện tại, và tự động cập nhật những phần đã lệch. Dùng sau khi thay đổi code lớn hoặc trước khi merge PR.
---

# /sync-docs — Đồng bộ tài liệu với codebase

$ARGUMENTS

---

## Mục tiêu

Agent sẽ tự quét codebase, so sánh với nội dung **6 file tài liệu** trong `docs/` và file `AGENTS.md`, sau đó cập nhật những phần đã lỗi thời — **không hỏi xác nhận từng file**, chỉ báo cáo tổng hợp khi xong.

---

## Bản đồ tài liệu (Source of Truth)

| File docs | Section markers | Nguồn dữ liệu thực tế trong codebase |
|-----------|----------------|--------------------------------------|
| `docs/ARCHITECTURE.md` | `<!-- BEGIN:service-communication -->`, `<!-- BEGIN:data-flow -->` | `docker-compose.yml`, `core_backend/app/main.py`, `ai_engine/app/main.py`, `core_backend/app/services/ai_client.py` |
| `docs/API_CONTRACT.md` | `<!-- BEGIN:core-backend-api -->`, `<!-- BEGIN:ai-engine-api -->`, `<!-- BEGIN:error-codes -->` | `core_backend/app/domains/*/router.py`, `core_backend/app/domains/*/schemas.py`, `ai_engine/app/nlp/schemas.py` |
| `docs/DATABASE.md` | Toàn bộ file | `core_backend/app/domains/*/models.py`, `alembic/versions/` |
| `docs/TECH_STACK.md` | `<!-- BEGIN:tech-stack-mapping -->` | `Frontend/package.json`, `core_backend/requirements.txt`, `ai_engine/requirements.txt` |
| `docs/UI_System_Design.md` | Toàn bộ file | `Frontend/src/app/globals.css`, `Frontend/src/components/` |
| `docs/foder_description.md` | `<!-- BEGIN:folder-description -->` | Cấu trúc thư mục thực tế (quét bằng `ls` / `Get-ChildItem`) |
| `AGENTS.md` | `<!-- BEGIN:docs-index -->` | Danh sách file trong `docs/` |

---

## Quy trình thực hiện

### Phase 1 — Thu thập dữ liệu thực tế

Đọc song song tất cả nguồn sau (KHÔNG sửa gì ở bước này):

1. **Dependency files**:
   - `Frontend/package.json` → dependencies + devDependencies
   - `core_backend/requirements.txt`
   - `ai_engine/requirements.txt`

2. **API surface** (router + schema):
   - Tất cả file `router.py` trong `core_backend/app/domains/*/`
   - Tất cả file `schemas.py` trong `core_backend/app/domains/*/`
   - `ai_engine/app/nlp/schemas.py` + `ai_engine/app/nlp/embeddings.py`

3. **Infra config**:
   - `docker-compose.yml` (ports, service names, env_file)
   - `core_backend/app/main.py` (CORS origins, router prefixes)
   - `core_backend/app/core/config.py` (env var names)

4. **Database models**:
   - Tất cả file `models.py` trong `core_backend/app/domains/*/`
   - `alembic/versions/` (nếu tồn tại)

5. **Folder scan**:
   - `list_dir` đệ quy trên root project (depth 3)

### Phase 2 — So sánh & Phân loại

Với mỗi file docs, so sánh dữ liệu thực tế (Phase 1) với nội dung hiện tại:

| Trạng thái | Ý nghĩa | Hành động |
|------------|---------|-----------|
| ✅ Đồng bộ | Nội dung docs khớp codebase | Bỏ qua |
| ⚠️ Lệch | Thông tin cũ hoặc thiếu | Cập nhật nội dung trong section markers |
| ❌ Thiếu section | Section marker tồn tại nhưng nội dung rỗng | Sinh nội dung mới |
| 🆕 Phát hiện mới | Có endpoint/model/package mới chưa được document | Thêm vào section phù hợp |

### Phase 3 — Áp dụng thay đổi

**Nguyên tắc sửa file:**

1. **Chỉ sửa nội dung giữa `<!-- BEGIN:xxx -->` và `<!-- END:xxx -->`**. Không thêm/xóa markers.
2. **Giữ nguyên format** (bảng Markdown, code block, heading level) như bản gốc.
3. **Không xóa comment hoặc description** đã có trừ khi thông tin sai.
4. Nếu cần thêm section mới, đặt marker `<!-- BEGIN:new-section -->` ... `<!-- END:new-section -->`.
5. **File `AGENTS.md`**: chỉ cập nhật bảng `docs-index` nếu có file mới trong `docs/`.

**Ưu tiên sửa theo thứ tự:**

```
1. docs/API_CONTRACT.md     (dễ lệch nhất — endpoint thay đổi thường xuyên)
2. docs/TECH_STACK.md       (package.json / requirements thay đổi khi install)
3. docs/DATABASE.md         (model thay đổi khi migration)
4. docs/ARCHITECTURE.md     (ít thay đổi — chỉ khi thêm/xóa service)
5. docs/foder_description.md (chỉ khi thay đổi cấu trúc thư mục)
6. docs/UI_System_Design.md (chỉ khi thay đổi design tokens)
7. AGENTS.md                (chỉ khi thêm/xóa file docs)
```

### Phase 4 — Báo cáo

Sau khi hoàn tất, in bảng tóm tắt:

```
📋 Kết quả đồng bộ docs
┌──────────────────────────┬────────────┬──────────────────────────────┐
│ File                     │ Trạng thái │ Thay đổi                     │
├──────────────────────────┼────────────┼──────────────────────────────┤
│ docs/API_CONTRACT.md     │ ⚠️ Updated │ Thêm endpoint POST /api/...  │
│ docs/TECH_STACK.md       │ ✅ Synced  │ —                            │
│ docs/DATABASE.md         │ ⚠️ Updated │ Cập nhật bảng dishes         │
│ docs/ARCHITECTURE.md     │ ✅ Synced  │ —                            │
│ docs/foder_description.md│ ✅ Synced  │ —                            │
│ docs/UI_System_Design.md │ ✅ Synced  │ —                            │
│ AGENTS.md                │ ✅ Synced  │ —                            │
└──────────────────────────┴────────────┴──────────────────────────────┘
```

---

## Chế độ chạy

### Mặc định: Full Sync
```
/sync-docs
```
Quét toàn bộ 7 file, cập nhật tất cả phần lệch.

### Chỉ định file cụ thể
```
/sync-docs api
/sync-docs database
/sync-docs architecture
/sync-docs tech-stack
/sync-docs folder
/sync-docs ui
```
Chỉ quét và cập nhật file tương ứng.

### Dry Run (chỉ báo cáo, không sửa)
```
/sync-docs --dry-run
```
Chạy Phase 1 + 2, in bảng so sánh nhưng KHÔNG sửa file.

---

## Quy tắc an toàn

1. **Không bao giờ xóa** section marker `<!-- BEGIN/END -->`.
2. **Không tạo file docs mới** ngoài 6 file đã đăng ký + `AGENTS.md`. Nếu phát hiện cần file mới → báo cho user.
3. **Nếu phát hiện mâu thuẫn** giữa 2 source (VD schema nói `Optional[float]` nhưng router docstring nói `required`) → ưu tiên Pydantic schema, đánh dấu `⚠️` trong report.
4. **Giữ nguyên ngôn ngữ** của file gốc (tiếng Việt → tiếng Việt, tiếng Anh → tiếng Anh).
5. **Comment code/biến** vẫn giữ tiếng Anh theo convention.
