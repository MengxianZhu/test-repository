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
             CSVParser csvParser = new CSVParser(reader, CSVFormat.DEFAULT.withFirstRecordAsHeader());
             FileWriter writer = new FileWriter(destinationDir + File.separator + inputFile.getName());
             CSVPrinter csvPrinter = new CSVPrinter(writer, CSVFormat.DEFAULT)) {

            // Write header
            List<String> headerNames = new ArrayList<>(csvParser.getHeaderMap().keySet());
            csvPrinter.printRecord(headerNames);

            // Process data rows
            for (CSVRecord record : csvParser) {
                List<String> values = new ArrayList<>();
                for (String header : headerNames) {
                    String value = record.get(header);

                    // Process specific headers
                    switch (header.toUpperCase()) {
                        case "FORMAT":
                            value = "-".equals(value) ? "" : value;
                            break;
                        case "PRO":
                            if (value.isEmpty() && !csvParser.getHeaderMap().containsKey("PRO")) {
                                value = record.get("PRA");
                            }
                            break;
                        case "CD":
                        case "ISD":
                            value = value.isEmpty() ? "N" : value;
                            break;
                        case "SOURCE":
                            value = "NA".equals(value) || "--NA--".equals(value) ? "" : value;
                            break;
                        // Add additional header processing here if needed
                    }

                    values.add(value);
                }
                csvPrinter.printRecord(values);
            }

            csvPrinter.flush();

            System.out.println("Processed: " + inputFile.getName());
        } catch (IOException | CsvValidationException e) {
            e.printStackTrace();
        }
    }
}
