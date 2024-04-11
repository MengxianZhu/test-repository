java
Copy code
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

public class ExcelProcessor {

    public static void main(String[] args) {
        String inputFile = "input.xlsx";

        try {
            // Load the input Excel file
            FileInputStream inputStream = new FileInputStream(inputFile);
            Workbook workbook = new XSSFWorkbook(inputStream);
            Sheet sheet = workbook.getSheetAt(0);

            // Process the header row
            Row headerRow = sheet.getRow(0); // Header row is at index 0
            int nameIndex = -1;
            int ageIndex = -1;
            int sexIndex = -1;
            int orderIndex = -1;
            int dateIndex = -1;
            int typeIndex = -1;
            int formatIndex = -1;
            for (Cell cell : headerRow) {
                String headerName = cell.getStringCellValue().trim(); // Trim the header name
                switch (headerName) {
                    case "name":
                        nameIndex = cell.getColumnIndex();
                        break;
                    case "age":
                        ageIndex = cell.getColumnIndex();
                        break;
                    case "sex":
                        sexIndex = cell.getColumnIndex();
                        break;
                    case "order":
                        orderIndex = cell.getColumnIndex();
                        break;
                    case "date":
                        dateIndex = cell.getColumnIndex();
                        break;
                    case "type":
                        typeIndex = cell.getColumnIndex();
                        break;
                    case "format":
                        formatIndex = cell.getColumnIndex();
                        break;
                }
            }

            // Process data rows
            int expectedOrder = 1;
            for (int i = 1; i <= sheet.getLastRowNum(); i++) { // Data starts from the second row
                Row row = sheet.getRow(i);
                // Check name field
                Cell nameCell = row.getCell(nameIndex);
                if (nameCell == null || nameCell.getCellType() == CellType.BLANK) {
                    setCellStyle(nameCell, getGrayCellStyle(workbook));
                }
                // Check age field
                Cell ageCell = row.getCell(ageIndex);
                if (ageCell == null || ageCell.getCellType() == CellType.BLANK || !isValidAge(ageCell)) {
                    setCellStyle(ageCell, getYellowCellStyle(workbook));
                }
                // Check sex field
                Cell sexCell = row.getCell(sexIndex);
                if (sexCell != null && sexCell.getCellType() == CellType.STRING && sexCell.getStringCellValue().toLowerCase().contains("na")) {
                    sexCell.setCellValue(""); // Replace with empty string
                }
                // Check order field
                Cell orderCell = row.getCell(orderIndex);
                if (orderCell != null && orderCell.getCellType() == CellType.NUMERIC) {
                    int currentOrder = (int) orderCell.getNumericCellValue();
                    if (currentOrder != expectedOrder) {
                        orderCell.setCellValue(expectedOrder); // Correct the order
                    }
                    expectedOrder++;
                }
                // Check date field format
                Cell typeCell = row.getCell(typeIndex);
                Cell formatCell = row.getCell(formatIndex);
                if (typeCell != null && formatCell != null &&
                        typeCell.getCellType() == CellType.STRING && formatCell.getCellType() == CellType.BLANK) {
                    String fieldType = typeCell.getStringCellValue().trim();
                    if ("date".equals(fieldType.toLowerCase())) {
                        setCellStyle(formatCell, getRedCellStyle(workbook)); // Mark red if the format is empty for date type
                    }
                }
            }

            // Write changes to the input Excel file
            FileOutputStream outputStream = new FileOutputStream(inputFile);
            workbook.write(outputStream);
            workbook.close();
            inputStream.close();
            outputStream.close();

            System.out.println("Processing completed.");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static boolean isValidAge(Cell ageCell) {
        String ageValue = ageCell.getStringCellValue();
        return "30".equals(ageValue) || "40".equals(ageValue) || "50".equals(ageValue);
    }

    private static CellStyle getGrayCellStyle(Workbook workbook) {
        CellStyle style = workbook.createCellStyle();
        style.setFillForegroundColor(IndexedColors.GREY_25_PERCENT.getIndex());
        style.setFillPattern(FillPatternType.SOLID_FOREGROUND);
        return style;
    }

    private static CellStyle getYellowCellStyle(Workbook workbook) {
        CellStyle style = workbook.createCellStyle();
        style.setFillForegroundColor(IndexedColors.YELLOW.getIndex());
        style.setFillPattern(FillPatternType.SOLID_FOREGROUND);
        return style;
    }

    private static CellStyle getRedCellStyle(Workbook workbook) {
        CellStyle style = workbook.createCellStyle();
        style.setFillForegroundColor(IndexedColors.RED.getIndex());
        style.setFillPattern(FillPatternType.SOLID_FOREGROUND);
        return style;
    }

    private static void setCellStyle(Cell cell, CellStyle style) {
        if (cell != null) {
            cell.setCellStyle(style);
        }
    }
}
