Chào bạn, khi chuyển từ kiểu `String` sang `int`, chúng ta có lợi thế là không cần lo lắng về chuỗi rỗng (`""`), nhưng lại cần cẩn thận với giá trị mặc định của kiểu dữ liệu nguyên thủy.

Dưới đây là cách triển khai hoàn chỉnh từ SQL XML cho đến Repository để đảm bảo logic "null (hoặc không truyền) thì lấy tất cả" hoạt động mượt mà nhất.

### 1. SQL XML (`orm.xml` hoặc `@NamedNativeQuery`)

Với kiểu `int`, trong SQL chúng ta vẫn sử dụng kỹ thuật "Short-circuit" với toán tử `OR`.

> **Lưu ý quan trọng:** Để tham số có thể nhận giá trị `null`, trong Java bạn phải dùng kiểu Wrapper là `Integer` thay vì kiểu primitive `int`.

```xml
<named-native-query name="Invoice.findByRange">
    <query>
        <![CDATA[
        SELECT * FROM invoices
        WHERE (:fromInvoice IS NULL OR invoice_number >= :fromInvoice)
          AND (:toInvoice IS NULL OR invoice_number <= :toInvoice)
        ORDER BY invoice_number ASC
        ]]>
    </query>
</named-native-query>

```

---

### 2. Repository Interface

Tại đây, bạn sử dụng `@Param` và nhớ dùng kiểu `Integer` để Spring Data JPA có thể truyền giá trị `null` xuống Database.

```java
@Repository
public interface InvoiceRepository extends JpaRepository<Invoice, Long> {
    
    @Query(nativeQuery = true, name = "Invoice.findByRange")
    List<Invoice> findByRange(@Param("fromInvoice") Integer fromInvoice, 
                              @Param("toInvoice") Integer toInvoice);
}

```

---

### 3. Request DTO (Sử dụng cho @RequestBody)

Việc sử dụng `Integer` ở DTO giúp Spring Boot tự động gán `null` nếu Client không gửi trường đó trong JSON.

```java
@Data
public class InvoiceSearchRequest {
    // Sử dụng Integer thay vì int để cho phép null
    private Integer fromInvoice; 
    private Integer toInvoice;
}

```

---

### 4. Tại sao cách này lại "Dễ sử dụng nhất"?

Dưới góc độ **BrSE**, mình phân tích 4 kịch bản mà hệ thống sẽ tự động xử lý:

| Client gửi JSON | `fromInvoice` (Java) | `toInvoice` (Java) | SQL sinh ra (Logic) | Kết quả |
| --- | --- | --- | --- | --- |
| `{ }` | `null` | `null` | `WHERE (T) AND (T)` | Lấy tất cả |
| `{ "fromInvoice": 10 }` | `10` | `null` | `WHERE (num >= 10) AND (T)` | Từ 10 đến hết |
| `{ "toInvoice": 50 }` | `null` | `50` | `WHERE (T) AND (num <= 50)` | Từ đầu đến 50 |
| `{ "from": 10, "to": 50 }` | `10` | `50` | `WHERE (num >= 10) AND (num <= 50)` | Trong khoảng 10-50 |

---

### 5. Một chút tinh tế từ Backend Dev Level 1

Để báo cáo xuất ra không bị lỗi logic, bạn nên thêm một bước kiểm tra nhỏ trong Service trước khi gọi Repository:

```java
public byte[] processExport(InvoiceSearchRequest request) {
    // Tránh trường hợp người dùng nhập ngược (from 100 - to 50)
    if (request.getFromInvoice() != null && request.getToInvoice() != null) {
        if (request.getFromInvoice() > request.getToInvoice()) {
            throw new BusinessException("Số hóa đơn bắt đầu không được lớn hơn số kết thúc!");
        }
    }
    
    List<Invoice> data = repository.findByRange(request.getFromInvoice(), request.getToInvoice());
    // ... gọi tiếp BaseExcelService ...
}

```

### Bước tiếp theo tôi có thể giúp bạn:

Bạn đã nắm rõ cách dùng `Integer` để truyền `null` chưa? Nếu bạn muốn, tôi có thể hướng dẫn bạn cách **Format số hóa đơn** (ví dụ: số `10` thành chuỗi `"INV-000010"`) ngay trong lúc điền vào Excel bằng bộ Base mà chúng ta đã xây dựng. Bạn có muốn thực hiện không?
