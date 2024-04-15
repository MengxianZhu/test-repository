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

            List<CSVRecord> records = csvParser.getRecords();

            // Process header
            CSVRecord header = records.get(0);
            List<String> headerNames = new ArrayList<>();
            for (int i = 0; i < header.size(); i++) {
                headerNames.add(header.get(i).trim().toUpperCase());
            }

            // Process data rows
            for (int i = 1; i < records.size(); i++) {
                CSVRecord record = records.get(i);

                // Process ISD column
                int isdIndex = getIndexForHeader("ISD", headerNames);
                if (isdIndex != -1 && isdIndex < record.size() && record.get(isdIndex).trim().isEmpty()) {
                    record = setRecordValue(record, isdIndex, "N", headerNames);
                }

                // Ensure that the record has the same number of columns as the header
                if (record.size() < headerNames.size()) {
                    record = padRecord(record, headerNames.size(), headerNames);
                }
            }

            // Write to new CSV
            String destinationFilePath = destinationDir + File.separator + inputFile.getName();
            try (Writer writer = Files.newBufferedWriter(Paths.get(destinationFilePath), StandardCharsets.UTF_8);
                 CSVPrinter csvPrinter = new CSVPrinter(writer, CSVFormat.DEFAULT.withHeader(headerNames.toArray(String[]::new)))) {

                for (CSVRecord record : records) {
                    csvPrinter.printRecord(record);
                }
                csvPrinter.flush();
            }

            System.out.println("Processed: " + inputFile.getName());
        } catch (IOException | CsvValidationException e) {
            e.printStackTrace();
        }
    }

    private static CSVRecord setRecordValue(CSVRecord record, int index, String value, List<String> headerNames) {
        List<String> values = new ArrayList<>(record.size());
        for (int i = 0; i < record.size(); i++) {
            values.add(i == index ? value : record.get(i));
        }
        for (int i = record.size(); i < headerNames.size(); i++) {
            values.add("");
        }
        Map<String, Integer> headerIndexMap = new HashMap<>();
        for (int i = 0; i < headerNames.size(); i++) {
            headerIndexMap.put(headerNames.get(i), i);
        }
        return new CSVRecord(headerNames, headerIndexMap, Collections.emptyMap(), -1, -1, values);
    }

    private static CSVRecord padRecord(CSVRecord record, int size, List<String> headerNames) {
        List<String> values = new ArrayList<>(size);
        for (int i = 0; i < size; i++) {
            values.add(i < record.size() ? record.get(i) : "");
        }
        Map<String, Integer> headerIndexMap = new HashMap<>();
        for (int i = 0; i < headerNames.size(); i++) {
            headerIndexMap.put(headerNames.get(i), i);
        }
        return new CSVRecord(headerNames, headerIndexMap, Collections.emptyMap(), -1, -1, values);
    }

    private static int getIndexForHeader(String headerName, List<String> header) {
        for (int i = 0; i < header.size(); i++) {
            if (header.get(i).equalsIgnoreCase(headerName)) {
                return i;
            }
        }
        return -1;
    }
}
