Ý tưởng của bạn rất hay và thực tế! Thay vì phải hard-code vị trí ô (`A1`, `B2`) trong Service, chúng ta sẽ sử dụng **Custom Annotation** để đánh dấu vị trí ngay trên DTO. Điều này giúp code cực kỳ "sạch" và dễ bảo trì.

Dưới đây là cách xây dựng bộ Base Service sử dụng **Reflection** trong Java để đọc các Annotation này.

---

### 1. Tạo Custom Annotation `@ExcelCell`

Chúng ta định nghĩa một Annotation để đánh dấu tọa độ ô trong Excel.

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface ExcelCell {
    String address() default ""; // Ví dụ: "A1"
    int row() default -1;        // Hoặc dùng tọa độ số
    int col() default -1;
}

```

### 2. Tạo DTO sử dụng Annotation

Bây giờ, bạn chỉ cần "mapping" các field với các ô trong file Excel template.

```java
public class InvoiceDTO {
    
    @ExcelCell(address = "B2")
    private String customerName;

    @ExcelCell(address = "B3")
    private String invoiceNo;

    @ExcelCell(row = 10, col = 1) // Tương đương ô A11
    private Double totalAmount;

    // Getters and Setters...
}

```

### 3. Xây dựng Base Service với Reflection

Phần quan trọng nhất là logic đọc Annotation và đổ dữ liệu vào `ExcelCreator`.

```java
import jp.co.adv.excelcreator.Creator;
import java.lang.reflect.Field;
import java.util.List;

public abstract class AnnotationBaseExportService<T> extends BaseExcelCreatorService<T> {

    @Override
    protected void fillData(Creator creator, T item, int index) {
        // Lấy tất cả các field của Class DTO
        Field[] fields = item.getClass().getDeclaredFields();

        for (Field field : fields) {
            // Kiểm tra xem field có đánh dấu @ExcelCell không
            if (field.isAnnotationPresent(ExcelCell.class)) {
                ExcelCell annotation = field.getAnnotation(ExcelCell.class);
                field.setAccessible(true); // Cho phép đọc private field

                try {
                    Object value = field.get(item);
                    if (value != null) {
                        fillCell(creator, annotation, value);
                    }
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    private void fillCell(Creator creator, ExcelCell annotation, Object value) {
        // Ưu tiên sử dụng địa chỉ trực tiếp như "A1"
        if (!annotation.address().isEmpty()) {
            creator.getCell(annotation.address()).setValue(value);
        } 
        // Nếu không có address thì dùng tọa độ row, col
        else if (annotation.row() != -1 && annotation.col() != -1) {
            creator.setPos(annotation.col(), annotation.row());
            creator.getCell().setValue(value);
        }
    }
}

```

---

### 4. Cách sử dụng cực kỳ đơn giản

Bây giờ, khi bạn tạo một Service mới, bạn **không cần viết lại logic fill data** nữa.

```java
@Service
public class ReportService extends AnnotationBaseExportService<InvoiceDTO> {
    // Chỉ cần kế thừa, logic fillData đã được xử lý tự động ở Base bằng Reflection
}

```

---

### Phân tích từ góc độ BrSE (Ưu & Nhược điểm)

**Ưu điểm:**

* **Maintainability:** Khi khách hàng đổi template (ví dụ: chuyển tên khách hàng từ ô B2 sang C5), bạn chỉ cần sửa 1 chỗ duy nhất trong DTO, không cần đụng vào code logic.
* **Speed:** Code cực nhanh cho các báo cáo có hàng chục field.
* **Readability:** Nhìn vào DTO là biết ngay dữ liệu sẽ nằm ở đâu trên file Excel.

**Lưu ý kỹ thuật:**

1. **Reflection Performance:** Reflection hơi chậm một chút so với gọi trực tiếp. Tuy nhiên, với việc xuất báo cáo (vài chục đến vài trăm trang), sự chênh lệch này là không đáng kể (mili giây).
2. **Trường hợp danh sách (Table):** Nếu bạn có một danh sách các mặt hàng (List Items) đổ vào dạng bảng, bạn nên kết hợp thêm một Annotation `@ExcelTable` hoặc xử lý thủ công một chút trong hàm `fillData` ghi đè (Override).

Bạn có muốn tôi cải tiến thêm để bộ Base này xử lý được cả **danh sách (Table/List)** tự động tăng dòng không?

Tuyệt vời! Là một **BrSE**, mình rất khuyến khích việc tự động hóa phần **Table (danh sách)** vì đây là phần dễ gây lỗi nhất (tràn trang, vỡ format) khi làm báo cáo.

Để xử lý Table, chúng ta sẽ cần thêm một Annotation nữa để đánh dấu: **"Field này là một danh sách và cần được đổ vào bảng từ dòng nào"**.

Dưới đây là giải pháp nâng cấp:

### 1. Tạo thêm Annotation `@ExcelTable`

Annotation này sẽ đặt trên một `List` trong DTO.

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface ExcelTable {
    int startRow(); // Dòng bắt đầu (0-based)
    String[] columnMapping(); // Thứ tự các field của object con (ví dụ: "id", "name", "price")
}

```

### 2. Cấu trúc DTO hoàn chỉnh

```java
public class ExportDTO {
    // Phần header đơn lẻ
    @ExcelCell(address = "B2")
    private String reporterName;

    // Phần danh sách (Table)
    @ExcelTable(
        startRow = 10, 
        columnMapping = {"id", "productName", "quantity", "price"}
    )
    private List<ItemDTO> items;

    // Getters, Setters...
}

public class ItemDTO {
    private Integer id;
    private String productName;
    private Integer quantity;
    private Double price;
    // Getters, Setters...
}

```

### 3. Nâng cấp Base Service (Xử lý Reflection cho Table)

Mình sẽ cập nhật hàm `fillData` để nhận diện cả `@ExcelCell` và `@ExcelTable`.

```java
import jp.co.adv.excelcreator.Creator;
import java.lang.reflect.Field;
import java.util.Collection;
import java.util.List;

public abstract class AnnotationBaseExportService<T> extends BaseExcelCreatorService<T> {

    @Override
    protected void fillData(Creator creator, T item, int index) {
        Field[] fields = item.getClass().getDeclaredFields();

        for (Field field : fields) {
            field.setAccessible(true);
            try {
                // 1. Xử lý Cell đơn lẻ
                if (field.isAnnotationPresent(ExcelCell.class)) {
                    ExcelCell cellAnnos = field.getAnnotation(ExcelCell.class);
                    fillSingleCell(creator, cellAnnos, field.get(item));
                }
                
                // 2. Xử lý Table (Danh sách)
                else if (field.isAnnotationPresent(ExcelTable.class)) {
                    ExcelTable tableAnnos = field.getAnnotation(ExcelTable.class);
                    Object listData = field.get(item);
                    if (listData instanceof List) {
                        fillTableData(creator, tableAnnos, (List<?>) listData);
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    private void fillTableData(Creator creator, ExcelTable config, List<?> dataList) throws Exception {
        int currentRow = config.startRow();
        String[] mappings = config.columnMapping();

        for (Object rowData : dataList) {
            // Duyệt từng cột theo mapping đã định nghĩa
            for (int col = 0; col < mappings.length; col++) {
                Field rowField = rowData.getClass().getDeclaredField(mappings[col]);
                rowField.setAccessible(true);
                Object value = rowField.get(rowData);

                // Ghi dữ liệu vào ô (col, currentRow)
                creator.setPos(col, currentRow);
                creator.getCell().setValue(value);
            }
            currentRow++; // Xuống dòng tiếp theo
            
            // Nếu dùng ExcelCreator, bạn có thể cần chèn thêm dòng nếu danh sách dài
            // creator.rowInsert(currentRow); 
        }
    }

    private void fillSingleCell(Creator creator, ExcelCell annos, Object value) {
        if (value == null) return;
        if (!annos.address().isEmpty()) {
            creator.getCell(annos.address()).setValue(value);
        } else if (annos.row() != -1 && annos.col() != -1) {
            creator.setPos(annos.col(), annos.row());
            creator.getCell().setValue(value);
        }
    }
}

```

---

### 4. Phân tích Kỹ thuật (BrSE Insight)

* **Tính linh hoạt:** `columnMapping = {"id", "productName"}` cực kỳ quan trọng. Nó cho phép bạn chọn thứ tự cột xuất hiện trong Excel mà không phụ thuộc vào thứ tự khai báo field trong Java.
* **Vấn đề Page Break (Ngắt trang):** Nếu Table quá dài (vượt quá 1 trang PDF), bạn cần tính toán logic:
* Sử dụng hàm `creator.rowInsert()` để đẩy các dòng dưới xuống, tránh ghi đè vào Footer.
* Hoặc nếu template có khung cố định (ví dụ chỉ chứa được 20 dòng), bạn phải chia nhỏ `List<ItemDTO>` ra nhiều trang (mỗi trang 20 items).


* **Hiệu năng:** Việc dùng `getDeclaredField` trong vòng lặp Table có thể tối ưu bằng cách **Cache** các Field object vào một `Map` để không phải tìm lại ở mỗi dòng.

### Bước tiếp theo:

Bạn có muốn tôi xử lý phần **Format** (định dạng số tiền, ngày tháng, hoặc kẻ khung tự động cho Table) luôn không? Phần này sẽ giúp báo cáo trông chuyên nghiệp (Professional) hơn rất nhiều.
