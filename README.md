import org.apache.commons.csv.*;

import java.io.*;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.*;

public class CSVProcessor {
    public static void main(String[] args) {
        String sourceDirectory = "source_directory_path";
        String destinationDirectory = "destination_directory_path";

        processCSVs(sourceDirectory, destinationDirectory);
    }

    public static void processCSVs(String sourceDir, String destinationDir) {
        File sourceFolder = new File(sourceDir);
        File destinationFolder = new File(destinationDir);

        if (!sourceFolder.exists() || !sourceFolder.isDirectory()) {
            System.err.println("Source directory does not exist or is not a directory.");
            return;
        }

        if (!destinationFolder.exists() && !destinationFolder.mkdirs()) {
            System.err.println("Failed to create destination directory.");
            return;
        }

        File[] files = sourceFolder.listFiles((dir, name) -> name.toLowerCase().endsWith(".csv"));
        if (files == null || files.length == 0) {
            System.out.println("No CSV files found in the source directory.");
            return;
        }

        for (File file : files) {
            processCSV(file, destinationDir);
        }

        System.out.println("CSV processing completed.");
    }

    public static void processCSV(File inputFile, String destinationDir) {
        try (Reader reader = Files.newBufferedReader(Paths.get(inputFile.getPath()), StandardCharsets.UTF_8);
             CSVParser csvParser = new CSVParser(reader, CSVFormat.DEFAULT.withFirstRecordAsHeader())) {

            // Get header names
            List<String> headerNames = new ArrayList<>(csvParser.getHeaderMap().keySet());

            // Process data rows
            int colIndex = headerNames.indexOf("COL");
            int expectedValue = 1;
            Map<String, String> firstValues = new HashMap<>();
            for (CSVRecord record : csvParser) {
                String colValue = record.get(colIndex);
                switch (headerNames.get(colIndex).toUpperCase()) {
                    case "COL":
                        colValue = fixOrder(colValue, expectedValue);
                        expectedValue++;
                        break;
                    case "FORMAT":
                        colValue = "-".equals(colValue) ? "" : colValue;
                        break;
                    case "PRO":
                        if (colValue.isEmpty() && !headerNames.contains("PRO")) {
                            colValue = record.get("PRA");
                        }
                        break;
                    case "CD":
                    case "ISD":
                        colValue = colValue.isEmpty() ? "N" : colValue;
                        break;
                    case "SOURCE":
                        colValue = "NA".equals(colValue) || "--NA--".equals(colValue) ? "" : colValue;
                        break;
                    // Add additional header processing here if needed
                    case "TABLE":
                    case "PARTITION":
                        if (!firstValues.containsKey(headerNames.get(colIndex))) {
                            firstValues.put(headerNames.get(colIndex), colValue.toUpperCase());
                        }
                        break;
                }
            }

            // Generate new file name
            String newFileName = "";
            if (firstValues.containsKey("TABLE")) {
                newFileName += firstValues.get("TABLE");
            }
            if (firstValues.containsKey("PARTITION")) {
                if (!newFileName.isEmpty()) {
                    newFileName += "_";
                }
                newFileName += firstValues.get("PARTITION");
            }
            newFileName += ".csv";

            // Write to new CSV file
            try (FileWriter writer = new FileWriter(destinationDir + File.separator + newFileName);
                 CSVPrinter csvPrinter = new CSVPrinter(writer, CSVFormat.DEFAULT)) {

                // Write header
                csvPrinter.printRecord(headerNames);

                // Process data rows again to print to the new CSV file
                csvParser.forEach(record -> {
                    try {
                        csvPrinter.printRecord(record);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                });
            }

            System.out.println("Processed: " + inputFile.getName() + " => " + newFileName);
        } catch (IOException | CsvValidationException e) {
            e.printStackTrace();
        }
    }

    private static String fixOrder(String value, int expectedValue) {
        // Check if the value matches the expected order
        int colValue = Integer.parseInt(value);
        if (colValue != expectedValue) {
            // If not, correct the value to match the expected order
            value = Integer.toString(expectedValue);
        }
        return value;
    }
}
