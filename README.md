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
            Row headerRow = sheet.getRow(0);
            int nameIndex = -1;
            int ageIndex = -1;
            int sexIndex = -1;
            for (Cell cell : headerRow) {
                String headerName = cell.getStringCellValue();
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
                }
            }

            // Process data rows
            for (int i = 1; i <= sheet.getLastRowNum(); i++) {
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

    private static void setCellStyle(Cell cell, CellStyle style) {
        if (cell != null) {
            cell.setCellStyle(style);
        }
    }
}
