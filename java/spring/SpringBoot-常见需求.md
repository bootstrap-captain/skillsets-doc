# CSV

- Comma Separated Value

## Apache Common CSV

[GitHub](https://github.com/apache/commons-csv)

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-csv</artifactId>
    <version>1.13.0</version>
</dependency>
```

### 导出文件

- springboot中，生成csv文件，前端发送请求后，自动下载文件

```java
package com.citi.file;

import lombok.AllArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor
public class Student {
    private int id;
    private String name;
    private String email;
    private String address;
}
```

```java
package com.citi.file;

import lombok.extern.slf4j.Slf4j;
import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVPrinter;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.io.StringWriter;
import java.util.List;

@Service
@Slf4j
public class CSVService {

    public String convertDataToString(List<Student> data, String[] headers) {
        /*1. 定义CSV的header，分割符*/
        CSVFormat csvFormat = CSVFormat.DEFAULT
                .builder()
                .setHeader(headers)
                .setDelimiter(',')
                .get();

        /*2. 读取数据到文件中 */
        try (StringWriter sw = new StringWriter();
             CSVPrinter printer = new CSVPrinter(sw, csvFormat)) {
            for (Student student : data) {
                printer.printRecord(student.getId(), student.getName(), student.getEmail(), student.getAddress());
            }
            /*3. 得到具体的CSV文件的字符串*/
            return sw.toString();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

```java
package com.citi.controller;

import com.citi.file.CSVService;
import com.citi.file.Student;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.List;

@Slf4j
@RestController
@RequestMapping("/export")
public class ReplayExportController {

    @Autowired
    private CSVService csvService;

    @GetMapping("/csv")
    public void exportCSV(HttpServletResponse response) throws IOException {
        String fileName = "data.csv";
        response.setContentType("text/csv");
        response.setHeader(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + fileName + "\"");
        response.setCharacterEncoding(StandardCharsets.UTF_8.name()); // 确保中文不会乱码
        response.getWriter().write(initStudent());
    }


    public String initStudent() {
        List<Student> students = new ArrayList<>();
        students.add(new Student(123, "张三", "beijing", "192.com"));
        students.add(new Student(345, "李四", "nanjing", "183.com"));
        students.add(new Student(456, "王五", "美国", "qq.com"));
        String[] headers = {"id", "姓名", "email", "address"};
        return csvService.convertDataToString(students, headers);
    }
}
```

