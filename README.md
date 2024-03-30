import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVParser;
import org.apache.commons.csv.CSVRecord;

import java.io.*;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class CsvToExcelWithDefaultValues {

    public static void main(String[] args) {
        String csvFilePath = "input.csv";
        String excelFilePath = "output.xlsx";

        convertCsvToExcelWithDefaultValues(csvFilePath, excelFilePath);
    }

    public static void convertCsvToExcelWithDefaultValues(String csvFilePath, String excelFilePath) {
        try (Workbook workbook = new XSSFWorkbook();
             FileOutputStream fileOut = new FileOutputStream(excelFilePath);
             Reader reader = new FileReader(csvFilePath);
             CSVParser csvParser = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(reader)) {

            Sheet sheet = workbook.createSheet("Sheet1");

            // 指定所有标题的顺序
            List<String> allHeaders = Arrays.asList("Name", "Age", "Salary", "Position", "Sex");

            // 创建标题行并设置默认值
            Row headerRow = sheet.createRow(0);
            for (int i = 0; i < allHeaders.size(); i++) {
                headerRow.createCell(i).setCellValue(allHeaders.get(i));
            }

            // 写入CSV数据到Excel中
            int rowNum = 1;
            for (CSVRecord csvRecord : csvParser) {
                Row row = sheet.createRow(rowNum++);
                for (int i = 0; i < allHeaders.size(); i++) {
                    String value = "";
                    if (csvRecord.isMapped(allHeaders.get(i))) {
                        value = csvRecord.get(allHeaders.get(i)).trim();
                    }
                    row.createCell(i).setCellValue(value);
                }
            }

            workbook.write(fileOut);
            System.out.println("Excel file created successfully!");

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
