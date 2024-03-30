import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVParser;
import org.apache.commons.csv.CSVRecord;

import java.io.*;
import java.util.Arrays;
import java.util.List;

public class CsvToExcelWithColorIfEmpty {

    public static void main(String[] args) {
        String csvFilePath = "input.csv";
        String excelFilePath = "output.xlsx";

        convertCsvToExcelWithColorIfEmpty(csvFilePath, excelFilePath);
    }

    public static void convertCsvToExcelWithColorIfEmpty(String csvFilePath, String excelFilePath) {
        try (Workbook workbook = new XSSFWorkbook();
             FileOutputStream fileOut = new FileOutputStream(excelFilePath);
             Reader reader = new FileReader(csvFilePath);
             CSVParser csvParser = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(reader)) {

            Sheet sheet = workbook.createSheet("Sheet1");

            // 指定所有标题的顺序
            List<String> allHeaders = Arrays.asList("Name", "Age", "Salary", "Position", "Sex");

            // 创建标题行
            Row headerRow = sheet.createRow(0);
            for (int i = 0; i < allHeaders.size(); i++) {
                headerRow.createCell(i).setCellValue(allHeaders.get(i));
            }

            // 写入CSV数据到Excel中
            int rowNum = 1;
            for (CSVRecord csvRecord : csvParser) {
                Row row = sheet.createRow(rowNum++);
                for (int i = 0; i < allHeaders.size(); i++) {
                    String header = allHeaders.get(i);
                    String value = csvRecord.isMapped(header) ? csvRecord.get(header).trim() : "";
                    Cell cell = row.createCell(i);
                    cell.setCellValue(value);

                    // 如果值为空，设置绿色背景
                    if (value.isEmpty()) {
                        CellStyle greenStyle = createGreenCellStyle(workbook);
                        cell.setCellStyle(greenStyle);
                    }
                }
            }

            workbook.write(fileOut);
            System.out.println("Excel file created successfully!");

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static CellStyle createGreenCellStyle(Workbook workbook) {
        CellStyle style = workbook.createCellStyle();
        style.setFillForegroundColor(IndexedColors.LIGHT_GREEN.getIndex());
        style.setFillPattern(FillPatternType.SOLID_FOREGROUND);
        return style;
    }
}
