---
title: 【POI】poi导入导出
date: 2023-06-02 11:24:49
tags:
  - POI
category: 
  - 后端
---



## POI

### 简介

Jakarta POI 是一套用于访问微软格式文档的Java API。Jakarta POI有很多组件组成，其中有用于操作Excel格式文件的HSSF和用于操作Word的HWPF，在各种组件中目前只有用于操作Excel的HSSF相对成熟。

**POI提供了HSSF、XSSF以及SXSSF三种方式操作Excel**

**HSSF**：Excel97-2003版本，扩展名为.xls。一个sheet最大行数65536，最大列数256。

**XSSF**：Excel2007版本开始，扩展名为.xlsx。一个sheet最大行数1048576，最大列数16384。

**SXSSF**：是在XSSF基础上，POI3.8版本开始提供的支持**低内存占用**的操作方式，扩展名为.xlsx。

> 注意：xssf是将数据存在内存，所以为了防止内存溢出，就出现了sxssf
>
> SXSSFWorkbook w3= new SXSSFWorkbook(1000);

上面 SXSSFWorkbook 设置内存中最多只有1000行数据，当超过这个数据时，就将内存之前的数据删除，并且会在硬盘中生成临时文件。从而保证了低内存消耗。

**Jakarta POI HSSF API组件:**

| 组件          | 描述            |
| ------------- | --------------- |
| HSSFWorkbook  | excel的文档对象 |
| HSSFSheet     | excel的表单     |
| HSSFRow       | excel的行       |
| HSSFCell      | excel的格子单元 |
| HSSFFont      | excel字体       |
| HSSFCellStyle | cell样式        |

**Jakarta POI XSSFAPI组件:**

| 组件          | 描述            |
| ------------- | --------------- |
| XSSFWorkbook  | excel的文档对象 |
| XSSFSheet     | excel的表单     |
| XSSFFRow      | excel的行       |
| XSSFCell      | excel的格子单元 |
| XSSFFont      | excel字体       |
| XSSFCellStyle | cell样式        |

### 导出

#### pom.xml

```java
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>3.14</version>
</dependency>
public void exportExcel(){
    XSSFWorkbook workbook = new XSSFWorkbook();
    XSSFSheet xssfSheet = xssfWorkbook.createSheet("sheet名称");
    XSSFRow title = sheet.createRow(0); //创建行 
    
    String[] titleNameArr = {"姓名","班级","学号","性别","兴趣"};
    for(int i=0;i<titleNameArr.length; i++){
        sheet.setColumnWidth(i, 15 * 256); // 设置单元格宽度
        XSSFCell titleCell = title.createCell(i);//创建单元格子
        titleCell.setCellValue(titleNameArr[i]);
    }
    
    // 获取数据后
    List<Student> studentList = queryData();
    // 根据数据创建内容行以及该行单元格子
    for(int i=0,rowIndex=1;i<studentList.length;i++,rowIndex++){
        XSSFRow dataRow = sheet.createRow(rowIndex);
        Student student = studentList.get(i);
        String[] excelEntityyArr ={student.getName(),stuent.getClassName(),student.getNo(),student.getSex(),student.getHobby()};
        dataRow.createCell(j).setCellValue(excelEntityProperyArr[j]);
        // 5、导出excel
        response.setHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode("表名.xlsx", "UTF-8"));
        ServletOutputStream out = response.getOutputStream();
        xssfWorkbook.write(out);
        out.close();
    }
    
}
```

设置样式

```java
//样式
XSSFFont font = workbook.createFont();
font.setFontName("方正楷体");
font.setFontHeightInPoints((short) 12);
font.setBold(true);
XSSFCellStyle cellStyle = workbook.createCellStyle();
cellStyle.setAlignment(HorizontalAlignment.CENTER);
cellStyle.setFont(font);
```

## 其他

SpringBoot+Poi+ajax实现导出 [excel](https://www.cnblogs.com/CF1314/p/16331514.html)
