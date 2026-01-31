# PM Data Generator (Tool giả lập dữ liệu)

Tool sinh dữ liệu cho **ClickHouse** và **MySQL**.


## 1. Cách sử dụng
Chạy tool với các tham số dòng lệnh tùy chọn:

```bash
./pm-gen [OPTIONS]
```

| Flag | Mô tả | Mặc định |
| :--- | :--- | :--- |
| `--config` | Đường dẫn file cấu hình | `config.yaml` |
| `--dbs` | Danh sách DB cần insert (cách nhau dấu phẩy) | (Tất cả trong config) |
| `--tables` | Danh sách bảng cụ thể (ưu tiên hơn config pattern) | (Rỗng) |
| `--numOfRecords` | Số lượng bản ghi cần sinh cho mỗi bảng | `100000` |

**Ví dụ:**
```bash
# Chạy cho ClickHouse, 1 triệu bản ghi
./pm-gen --dbs clickhouse --numOfRecords 1000000

# Chạy cho MySQL, bảng cụ thể
./pm-gen --dbs mysql --tables g_nr_cell_availability --numOfRecords 5000
```

## 2. Cấu hình (`config.yaml`)

### Databases
Định nghĩa các kết nối database. Hỗ trợ `clickhouse` và `mysql`.
```yaml
databases:
  clickhouse:
    type: clickhouse
    dsn: "clickhouse://user:pass@localhost:9000/db"
  mysql:
    type: mysql
    dsn: "user:pass@tcp(localhost:3306)/db"
```

### Table Pattern
Tự động quét và insert cho các bảng khớp mẫu nếu không chỉ định `--tables`.
```yaml
table_patterns:
  - "g_nr*"  # Quét tất cả bảng bắt đầu bằng g_nr
```

### Rules (Quy tắc sinh dữ liệu)
Định nghĩa trong `default_rules` (áp dụng chung) hoặc `tables.<tên_bảng>.columns`.

| Loại Rule (`type`) | Tham số | Mô tả |
| :--- | :--- | :--- |
| `int_range` | `min`, `max` | Sinh số trong khoảng (có tính tuần tự/chia module). |
| `random_int` | `min`, `max` | Sinh số ngẫu nhiên hoàn toàn (tránh trùng Key). |
| `constant` | `value` | Giá trị cố định. |
| `set` | `values: []` | Chọn ngẫu nhiên 1 giá trị từ danh sách. |
| `time_range` | `start`, `end` | Random thời gian trong khoảng (Format: YYYY-MM-DD HH:MM:SS). |
| `template` | `value` | Chuỗi mẫu, thay thế `{{number}}` bằng số thứ tự dòng. |

**Ví dụ Config:**
```yaml
default_rules:
  ne_id:
    type: random_int
    min: 1000
    max: 999999
  rat_type:
    type: set
    values: ["5G", "4G"]
  cell_name:
    type: template
    value: "Cell_{{number}}"
```
