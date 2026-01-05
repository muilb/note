# Hướng dẫn xuất nhiều trang PDF từ Excel Template

## Mô tả
Xuất nhiều trang PDF từ một Excel template có 1 sheet, mỗi content trong danh sách sẽ tạo ra 1 trang riêng biệt với header và footer.

## Các bước thực hiện

### 1. Chuẩn bị Template Excel
- Tạo file template Excel (.xlsx hoặc .xls) với 1 sheet
- Thiết kế layout cho 1 trang với các biến (ví dụ: `!!TITLE`, `!!CONTENT`, etc.)

### 2. Code mẫu

```java
import jp.co.adv.excelcreator.Creator;
import jp.co.adv.excelcreator.enums.ExcelVersion;
import jp.co.adv.excelcreator.enums.HeaderFooterType;
import java.util.List;

public void exportMultiPagePDF(List<ContentData> contentList) throws Exception {
    Creator creator = new Creator();
    
    // Thiết lập license nếu cần
    // creator.setLicense("path/to/license");
    
    String templatePath = "path/to/template.xlsx";
    String outputPath = "path/to/output.pdf";
    
    // Mở template
    creator.openBook(outputPath, templatePath);
    
    // Thiết lập header và footer
    setupHeaderFooter(creator);
    
    // Xử lý từng content
    for (int i = 0; i < contentList.size(); i++) {
        ContentData content = contentList.get(i);
        
        // Nếu không phải trang đầu, thêm page break
        if (i > 0) {
            creator.getCell("A1").setBreak(true);
        }
        
        // Điền dữ liệu vào các cell
        fillContentToSheet(creator, content, i);
    }
    
    // Đóng và xuất PDF
    creator.closeBook(true, outputPath, false);
}

private void setupHeaderFooter(Creator creator) {
    // Thiết lập header - phần trên cùng của mỗi trang
    // &L = left, &C = center, &R = right
    // &P = số trang hiện tại, &N = tổng số trang
    creator.setHeader(
        "Company Name",           // Left header
        "&20&BDocument Title",    // Center header (font size 20, bold)
        "&D",                     // Right header (ngày tháng)
        HeaderFooterType.Normal
    );
    
    // Thiết lập footer - phần dưới cùng của mỗi trang
    creator.setFooter(
        "Confidential",           // Left footer
        "",                       // Center footer
        "Page &P of &N",         // Right footer (số trang)
        HeaderFooterType.Normal
    );
}

private void fillContentToSheet(Creator creator, ContentData content, int pageIndex) {
    // Tính vị trí dòng bắt đầu cho mỗi trang
    // Giả sử mỗi trang có 30 dòng
    int rowOffset = pageIndex * 30;
    
    // Điền dữ liệu vào các cell với offset
    creator.getCell("A" + (1 + rowOffset)).setValue(content.getTitle());
    creator.getCell("A" + (2 + rowOffset)).setValue(content.getDescription());
    creator.getCell("A" + (3 + rowOffset)).setValue(content.getDetails());
    
    // Có thể thiết lập format cho từng cell
    creator.getCell("A" + (1 + rowOffset)).getAttr().setFontPoint(16);
    creator.getCell("A" + (1 + rowOffset)).getAttr().setFontBold(true);
}

// Class dữ liệu mẫu
class ContentData {
    private String title;
    private String description;
    private String details;
    
    // Getters and setters
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    public String getDetails() { return details; }
    public void setDetails(String details) { this.details = details; }
}
```

### 3. Phương pháp thay thế: Sử dụng nhiều sheet

```java
public void exportMultiPagePDFWithSheets(List<ContentData> contentList) throws Exception {
    Creator creator = new Creator();
    
    String templatePath = "path/to/template.xlsx";
    String outputPath = "path/to/output.pdf";
    
    // Mở template
    creator.openBook(outputPath, templatePath);
    
    // Thiết lập header/footer cho tất cả sheet
    setupHeaderFooter(creator);
    
    // Thêm sheet cho mỗi content (trừ sheet đầu tiên đã có)
    for (int i = 1; i < contentList.size(); i++) {
        creator.addSheet(i); // Thêm sheet mới
    }
    
    // Điền dữ liệu vào từng sheet
    for (int i = 0; i < contentList.size(); i++) {
        creator.setSheetNo(i); // Chọn sheet
        fillContentToSheet(creator, contentList.get(i), 0); // Offset = 0 vì mỗi sheet là 1 trang
    }
    
    // Xuất PDF
    creator.closeBook(true, outputPath, false);
}
```

### 4. Tùy chỉnh Header/Footer nâng cao

```java
private void setupAdvancedHeaderFooter(Creator creator) {
    // Header với nhiều định dạng
    creator.setHeader(
        "&L&B&14Left Text",       // Left: Bold, size 14
        "&C&I&16Center Text",     // Center: Italic, size 16
        "&R&U&D",                 // Right: Underline, ngày tháng
        HeaderFooterType.Normal
    );
    
    // Footer với page number
    creator.setFooter(
        "&L&8Footer Info",        // Left: size 8
        "&C&P/&N",                // Center: Trang hiện tại/Tổng trang
        "&R&T",                   // Right: Thời gian
        HeaderFooterType.Normal
    );
}
```

## Các ký tự đặc biệt trong Header/Footer

| Ký tự | Ý nghĩa |
|-------|---------|
| `&L` | Căn trái |
| `&C` | Căn giữa |
| `&R` | Căn phải |
| `&P` | Số trang hiện tại |
| `&N` | Tổng số trang |
| `&D` | Ngày hiện tại |
| `&T` | Thời gian hiện tại |
| `&B` | In đậm |
| `&I` | In nghiêng |
| `&U` | Gạch chân |
| `&nn` | Font size (ví dụ: `&20` = size 20) |

## Lưu ý

1. Sử dụng `setBreak(true)` để ngắt trang
2. Sử dụng `setHeader()` và `setFooter()` để thiết lập header/footer
3. Có thể sử dụng `HeaderFooterType` để phân biệt trang đầu, trang chẵn/lẻ
4. Phương thức `closeBook()` với tham số cuối là `false` để xuất PDF

## Ví dụ sử dụng

```java
public static void main(String[] args) {
    try {
        // Tạo danh sách content
        List<ContentData> contentList = new ArrayList<>();
        
        for (int i = 1; i <= 10; i++) {
            ContentData data = new ContentData();
            data.setTitle("Tiêu đề trang " + i);
            data.setDescription("Mô tả cho trang " + i);
            data.setDetails("Chi tiết nội dung trang " + i);
            contentList.add(data);
        }
        
        // Xuất PDF
        exportMultiPagePDF(contentList);
        
        System.out.println("Xuất PDF thành công!");
    } catch (Exception e) {
        e.printStackTrace();
    }
}

public void createMultipleSheets(List<ContentData> contentList) {
    Creator creator = new Creator();
    
    String templatePath = "path/to/template.xlsx";
    String outputPath = "path/to/output.xlsx";
    
    // Mở template
    creator.openBook(outputPath, templatePath);
    
    // Copy sheet template cho mỗi content (trừ sheet đầu tiên đã có)
    for (int i = 1; i < contentList.size(); i++) {
        int currentSheetCount = creator.getSheetCount();
        creator.copySheet(0, currentSheetCount, "Sheet_" + (i + 1));
    }
    
    // Điền dữ liệu vào từng sheet
    for (int i = 0; i < contentList.size(); i++) {
        creator.setSheetNo(i);
        
        // Điền dữ liệu cho sheet này
        ContentData content = contentList.get(i);
        creator.getCell("!!TITLE").setValue(content.getTitle());
        creator.getCell("!!DESCRIPTION").setValue(content.getDescription());
    }
    
    creator.closeBook(true);
}
```
