import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVParser;
import org.apache.commons.csv.CSVRecord;

import java.io.*;
import java.util.ArrayList;
import java.util.List;

public class CsvToExcelWithColor {

    public static void main(String[] args) {
        String csvFilePath = "input.csv";
        String excelFilePath = "output.xlsx";

        convertCsvToExcelWithColor(csvFilePath, excelFilePath);
    }

    public static void convertCsvToExcelWithColor(String csvFilePath, String excelFilePath) {
        try (Workbook workbook = new XSSFWorkbook();
             FileOutputStream fileOut = new FileOutputStream(excelFilePath);
             Reader reader = new FileReader(csvFilePath);
             CSVParser csvParser = new CSVParser(reader, CSVFormat.DEFAULT.withHeader())) {

            Sheet sheet = workbook.createSheet("Sheet1");
            CellStyle greenStyle = createGreenCellStyle(workbook);

            int rowNum = 0;
            for (CSVRecord csvRecord : csvParser) {
                Row row = sheet.createRow(rowNum++);
                int colNum = 0;
                for (String header : csvParser.getHeaderMap().keySet()) {
                    String value = csvRecord.get(header).trim();
                    Cell cell = row.createCell(colNum++);
                    cell.setCellValue(value);

                    // 如果值为空，设置绿色背景
                    if (value.isEmpty()) {
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
