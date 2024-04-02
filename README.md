public class CsvToXlsxConverter {

    private static final String INPUT_FOLDER = "input_folder";
    private static final String OUTPUT_FOLDER = "output_folder";

    public static void main(String[] args) {
        File inputDirectory = new File(INPUT_FOLDER);
        File outputDirectory = new File(OUTPUT_FOLDER);

        // 创建输出目录
        outputDirectory.mkdirs();

        // 获取输入目录下的所有CSV文件
        File[] csvFiles = inputDirectory.listFiles((dir, name) -> name.toLowerCase().endsWith(".csv"));

        // 转换每个CSV文件为XLSX格式
        if (csvFiles != null) {
            for (File csvFile : csvFiles) {
                String outputFileName = OUTPUT_FOLDER + File.separator + csvFile.getName().replace(".csv", ".xlsx");
                try {
                    convertCsvToXlsx(csvFile.getPath(), outputFileName);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
;
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
