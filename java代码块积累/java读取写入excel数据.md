### 使用场景

在java开发中，有时需要使用excel读取和写入数据。

可以使用 poi来处理。



### 引入依赖

```
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>3.16</version>
        </dependency>
        <!--xlsx(07)-->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>3.16</version>
        </dependency>
```



### Poi 工具类：

```
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.*;
import java.math.BigDecimal;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Map;

/**
 * Util提供的所有静态方法返回的对象都是Workbook，可根据需求再做处理
 */
@Slf4j
@Data
public class PoiUtil {
    public static final int EXCEL_2003 = 2003;
    public static final int EXCEL_2007 = 2007;
    public static final String DATE_PATTERN = "yyyy-MM-dd HH:mm:ss";
    public static final String XLSX = "xlsx";
    public static final String XLS = "xls";

    private PoiUtil() {
    }

    public static Workbook getWorkbook(String fileName) {
        Workbook workbook = null;
        try (FileInputStream fis = new FileInputStream(fileName)) {
            workbook = getWorkbook(fis, fileName);

        } catch (Exception e) {
            log.error("getWorkbook error.fileName:{}", fileName, e);
        }
        return workbook;
    }

    public static Sheet getSheet(String fileName) {
        Workbook workbook = getWorkbook(fileName);
        if (workbook == null) {
            return null;
        }
        return workbook.getSheetAt(0);
    }

    /**
     * 根据文件名，判断版本号，获取Excel poi对象
     *
     * @param fileName
     * @param in
     * @return
     */
    public static Workbook getWorkbook(InputStream in, String fileName)  {
        if (in == null) {
            return null;
        }
        int edition = getEdition(fileName);
        try {
            if (edition == EXCEL_2003) {
                return new HSSFWorkbook(in);
            } else if (edition == EXCEL_2007) {
                return new XSSFWorkbook(in);
            }
        } catch (IOException e) {
            log.error("getWorkbook error", e);
        }
        return null;
    }



    /**
     * 判断excel文件是2003版的xls文件，还是xlsx文件
     *
     * @param fileName
     * @return
     */
    public static int getEdition(String fileName) {
        int edition;
        if (StringUtils.isEmpty(fileName)) {
            return 0;
        }
        String xlsxFile = fileName.substring(fileName.length() - 4);
        String xlsFile = fileName.substring(fileName.length() - 3);
        if (XLSX.equals(xlsxFile)) {
            edition = EXCEL_2007;
        } else if (XLS.equals(xlsFile)) {
            edition = EXCEL_2003;
        } else {
            edition = 0;
        }
        return edition;
    }

    public static String getCellString(Cell cell) {
        return getCellString(cell, DATE_PATTERN);
    }


    /**
     * 获取单元格的String值
     * 文本型直接获取,数值型的一般是double类型，日期和数字要分开处理
     *
     * @param cell
     * @param datePattern 日期格式。如：yyyy-MM-dd HH:mm:ss
     * @return
     */
    public static String getCellString(Cell cell, String datePattern) {
        if (cell == null) {
            return "";
        }
        String cellStr = "";
        if (cell.getCellTypeEnum() == CellType.STRING) {
            cellStr = cell.getStringCellValue();
        } else if (cell.getCellTypeEnum() == CellType.NUMERIC
                && DateUtil.isCellDateFormatted(cell)) {
            //读取日期。DateUtil.isCellDateFormatted是 poi用于判断是否日期的方法
            Date date = cell.getDateCellValue();
            cellStr = new SimpleDateFormat(datePattern).format(date);
        } else if (cell.getCellTypeEnum() == CellType.NUMERIC) {
            // 表格中返回的数字类型是科学计数法因此不能直接转换成字符串格式
            BigDecimal bigDecimal = BigDecimal.valueOf(cell.getNumericCellValue());
            cellStr = bigDecimal.toString();
        } else if (cell.getCellTypeEnum() == CellType.FORMULA) {
            cellStr = BigDecimal.valueOf(cell.getNumericCellValue()).toPlainString();
        } else if (cell.getCellTypeEnum() == CellType.BOOLEAN) {
            cellStr = Boolean.toString(cell.getBooleanCellValue());
        }
        //去掉多余的空格和换行符
        return cellStr.trim();

    }

    public static List<List<String>> getRowColumnList(String fileName, int startRow) {
        return getRowColumnList(fileName, startRow, 0, 0, DATE_PATTERN);
    }

    public static List<List<String>> getRowColumnList(String fileName, int startRow, int startCol) {
        return getRowColumnList(fileName, startRow, startCol, 0, DATE_PATTERN);
    }


    /**
     * 从指定excel表格中逐行读取数据
     *
     * @param fileName
     * @param startRow   第几行开始读.从零开始。
     * @param startCol   第几列开始读.从零开始。
     * @param indexSheet 读取excel的哪个工作簿
     * @param datePattern 日期格式。如：yyyy-MM-dd HH:mm:ss
     * @return
     */
    public static List<List<String>> getRowColumnList(String fileName, int startRow, int startCol,
                                                      int indexSheet, String datePattern) {
        List<List<String>> rowColumnList = new ArrayList<>();
        Workbook workbook = getWorkbook(fileName);
        if (workbook == null) {
            return rowColumnList;
        }
        // 获取指定表对象
        Sheet sheet = workbook.getSheetAt(indexSheet);
        if (sheet == null) {
            return rowColumnList;
        }
        // 获取最大行数
        int rowNum = sheet.getLastRowNum();
        for (int i = startRow; i <= rowNum; i++) {
            List<String> oneRow = new ArrayList<>();
            Row row = sheet.getRow(i);
            if (row == null) {
                oneRow.add("");
                rowColumnList.add(oneRow);
                continue;
            }
            // 根据当前指针所在行数计算最大列数
            int colNum = row.getLastCellNum();
            for (int j = startCol; j < colNum; j++) {
                // 确定当前单元格
                Cell cell = row.getCell(j);
                String cellValue = getCellString(cell, datePattern);
                // 添加某一列的数据
                oneRow.add(cellValue);
            }
            rowColumnList.add(oneRow);
        }
        return rowColumnList;
    }

    /**
     * 根据给定的数据直接生成workbook
     *
     * @param workbook
     * @param sheetName
     * @param data
     * @return
     */
    public static Workbook createExcel(Workbook workbook, String sheetName, List<List<String>> data) {
        Sheet sheet = workbook.createSheet(sheetName);
        for (int i = 0; i < data.size(); i++) {
            List<String> oneRow = data.get(i);
            Row row = sheet.createRow(i);
            for (int j = 0; j < oneRow.size(); j++) {
                Cell cell = row.createCell(j);
                cell.setCellValue(oneRow.get(j));
            }
        }
        return workbook;
    }

    /**
     * 往指定的sheet表中插入数据，插入的方法是提供一组valueMap。int[]是2维数组代表需要插入的数据坐标，从0开始
     *
     * @param workbook
     * @param sheetIndex
     * @param valueMap
     * @return
     */
    public static Workbook insertExcel(Workbook workbook, int sheetIndex, Map<int[], String> valueMap) {
        Sheet sheet = workbook.getSheetAt(sheetIndex);
        for (Map.Entry<int[], String> cellEntry : valueMap.entrySet()) {
            int x = cellEntry.getKey()[0];
            int y = cellEntry.getKey()[1];
            String value = cellEntry.getValue();
            Row row = sheet.getRow(y);
            Cell cell = row.getCell(x);
            cell.setCellValue(value);
        }
        return workbook;
    }

    /**
     * 删除指定行
     *
     * @param workbook
     * @param sheetIndex
     * @param rowIndex
     * @return
     */
    public static Workbook removeRow(Workbook workbook, int sheetIndex, int rowIndex) {
        Sheet sheet = workbook.getSheetAt(sheetIndex);
        int lastRowNum = sheet.getLastRowNum();
        if (rowIndex >= 0 && rowIndex < lastRowNum) {
            sheet.shiftRows(rowIndex + 1, lastRowNum, -1);
        }
        if (rowIndex == lastRowNum) {
            sheet.removeRow(sheet.getRow(rowIndex));
        }
        return workbook;
    }

    /**
     * 将workbook转化为输入流
     *
     * @param workbook
     * @return
     */
    public static InputStream workbookConvertorStream(Workbook workbook) {
        InputStream in = null;
        try {
            //临时缓冲区
            ByteArrayOutputStream out = new ByteArrayOutputStream();
            //创建临时文件
            workbook.write(out);
            byte[] bookByteAry = out.toByteArray();
            int size = bookByteAry.length;
            in = new ByteArrayInputStream(bookByteAry);
            out.close();
            workbook.close();

            log.info("workbook文件大小== {}", size);

        } catch (Exception e) {
            log.error("workbook to inputStream exception catch.", e);
        }
        return in;
    }


    /**
     * 刷新流的缓存区,将workbook数据写入excel
     *
     * @param fileName
     * @param workbook
     */
    public static void writeWorkbookToExcel(String fileName, Workbook workbook)  {
        try (FileOutputStream fos = new FileOutputStream(fileName)) {
            fos.flush();
            workbook.write(fos);
            workbook.close();
        } catch (Exception e) {
            log.error("writeWorkbookToExcel error", e);
        }


    }




}
```



### 读取 excel 示例：

```
    /**
     * 读取 Excel
     *
     * @param fileName
     */
    public void readExcel(String fileName) {
        //获得默认的第一个sheet的页面
        Sheet sheet = PoiUtil.getSheet(fileName);
        if (sheet == null) {
            return;
        }
        //从第二行开始读，遍历到最后一行
        for (int i = 1; i <= sheet.getLastRowNum(); i++) {
            //得到每一行
            Row row = sheet.getRow(i);
            if (row == null) {
                continue;
            }
            //得到每一行的每一列
            for (int j = 0; j < row.getLastCellNum(); j++) {
                Cell cell = row.getCell(j);
                // 根据excel中单元格的属性，来用不同的格式取得有效值
                String cellValue = PoiUtil.getCellString(cell);
                System.out.println("单元格内容为:" + cellValue);
            }
        }
    }

```



### 读取 excel 示例二：

```
    public void read(String fileName) {
        //从第二行开始读
        List<List<String>> rowColumnList = PoiUtil.getRowColumnList(fileName, 1);
        for (List<String> list : rowColumnList) {
            for (String column : list) {
                System.out.print( column + ",");
            }
            System.out.println("");
        }
    }

```



### 写入 excel ：

```
    public void writeExcel(String file) {
        Workbook workbook = PoiUtil.getWorkbook(file);
        if (workbook == null) {
            return;
        }
        Sheet sheet = workbook.getSheetAt(0);
        if (sheet == null) {
            return;
        }
        //获取第一行
        Row row0 = sheet.getRow(0);
        //在第一行第一列写入数据,类型为string
        if (row0 != null) {
            Cell cell0 = row0.createCell(0, CellType.STRING);
            cell0.setCellValue("Order");
        }
        //在第二行开始写入数据
        for (int i = 1; i <= sheet.getLastRowNum(); i++) {
            Row row = sheet.getRow(i);
            if (row != null) {
                Cell cell = row.createCell(0, CellType.STRING);
                cell.setCellValue(String.valueOf(i + 1));
            }
        }
        //刷新excel数据
        PoiUtil.writeWorkbookToExcel(file, workbook);

        System.out.println("writeExcel end.");


    }

```


