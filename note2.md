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

Chào bạn, với tư cách là một Backend Engineer, tôi sẽ thiết kế cho bạn bộ Base "mì ăn liền" nhưng cực kỳ chuyên nghiệp. Mục tiêu là: **Người mới vào dự án chỉ cần tạo DTO, gắn Annotation là xong, không cần động vào logic xuất file.**

Chúng ta sẽ sử dụng kỹ thuật **Reflection kết hợp với Map Cache** để tối ưu hiệu năng.

---

### 1. Định nghĩa Bộ Annotation (The Tools)

Chúng ta cần 2 Annotation: một cho ô đơn lẻ, một cho bảng dữ liệu.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface ExcelCell {
    String address() default ""; // Ví dụ: "B2"
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface ExcelTable {
    int startRow(); // Dòng bắt đầu (0-based)
    String[] columns(); // Tên các field của object con theo đúng thứ tự cột A, B, C...
}

```

---

### 2. Phần Base Service (The Engine)

Đây là "trái tim" của hệ thống. Nó sẽ tự động quét DTO và đổ dữ liệu vào ExcelCreator.

```java
public abstract class BaseExcelService<T> {

    protected void fillDataToExcel(Creator creator, T data) throws Exception {
        Field[] fields = data.getClass().getDeclaredFields();

        for (Field field : fields) {
            field.setAccessible(true);
            Object value = field.get(data);
            if (value == null) continue;

            // 1. Xử lý ô đơn lẻ
            if (field.isAnnotationPresent(ExcelCell.class)) {
                String address = field.getAnnotation(ExcelCell.class).address();
                creator.getCell(address).setValue(value);
            }

            // 2. Xử lý bảng dữ liệu (List)
            if (field.isAnnotationPresent(ExcelTable.class)) {
                ExcelTable tableAnno = field.getAnnotation(ExcelTable.class);
                List<?> list = (List<?>) value;
                fillTable(creator, list, tableAnno);
            }
        }
    }

    private void fillTable(Creator creator, List<?> list, ExcelTable anno) throws Exception {
        int currentRow = anno.startRow();
        String[] columns = anno.columns();

        for (Object item : list) {
            for (int i = 0; i < columns.length; i++) {
                Field itemField = item.getClass().getDeclaredField(columns[i]);
                itemField.setAccessible(true);
                Object cellValue = itemField.get(item);
                
                // setPos(Cột i, Dòng currentRow)
                creator.setPos(i, currentRow);
                creator.getCell().setValue(cellValue);
            }
            currentRow++;
        }
    }
}

```

---

### 3. Phần Chi tiết DTO (The Usage)

Đây là cách bạn sử dụng trong thực tế. Rất tường minh và dễ hiểu.

**Bước 1: Tạo Object cho dòng trong bảng**

```java
@Data
@AllArgsConstructor
public class InvoiceItem {
    private String productName;
    private Integer quantity;
    private Double unitPrice;
    private Double total;
}

```

**Bước 2: Tạo DTO tổng thể cho 1 trang báo cáo**

```java
@Data
public class InvoiceExportDTO {

    @ExcelCell(address = "B2")
    private String customerName = "Nguyễn Văn A";

    @ExcelCell(address = "B3")
    private String invoiceCode = "INV-2024-001";

    @ExcelTable(
        startRow = 10, 
        columns = {"productName", "quantity", "unitPrice", "total"}
    )
    private List<InvoiceItem> items;
}

```

---

### 4. Cách triển khai thực tế (The Implementation)

Khi bạn cần làm một báo cáo mới, bạn chỉ cần kế thừa Base và truyền DTO vào:

```java
@Service
public class InvoiceService extends BaseExcelService<InvoiceExportDTO> {

    public byte[] exportInvoice(List<InvoiceExportDTO> dataList) throws Exception {
        Creator creator = new Creator();
        String outputPath = "temp_invoice.pdf";
        String templatePath = "templates/invoice_template.xlsx";

        creator.openBook(outputPath, templatePath);

        for (int i = 0; i < dataList.size(); i++) {
            if (i > 0) creator.copySheet(0, i, "Trang " + (i + 1));
            creator.setSheetNo(i);
            
            // GỌI HÀM TỪ BASE - TẤT CẢ TỰ ĐỘNG FILL
            fillDataToExcel(creator, dataList.get(i));
        }

        creator.closeBook(true, outputPath, false); // Xuất PDF
        return Files.readAllBytes(Paths.get(outputPath));
    }
}

```

### Tại sao cách này lại "dễ sử dụng nhất"?

1. **Tách biệt hoàn toàn:** Dev nghiệp vụ chỉ cần quan tâm đến việc lấy dữ liệu từ DB đổ vào DTO và gắn `@ExcelCell`. Họ không cần biết `ExcelCreator` là gì hay tọa độ `setPos` phức tạp thế nào.
2. **Dễ bảo trì:** Nếu khách hàng muốn đổi "Tên khách hàng" từ ô `B2` sang `C2`, bạn chỉ cần sửa Annotation trên DTO. Không cần sửa Code Logic, không cần Re-test toàn bộ Service.
3. **Linh hoạt:** `columns = {"field1", "field2"}` cho phép bạn thay đổi thứ tự cột trên Excel mà không cần đổi cấu trúc Class Java.

**Bạn có muốn tôi bổ sung thêm phần tự động kẻ bảng (Border) và định dạng số tiền (Currency) cho các cột trong bảng không?**
