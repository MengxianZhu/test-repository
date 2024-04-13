import java.io.*;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

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
        try (BufferedReader reader = new BufferedReader(new FileReader(inputFile))) {
            List<String[]> records = new ArrayList<>();
            String line;
            while ((line = reader.readLine()) != null) {
                records.add(line.split(","));
            }

            // Process header
            String[] header = records.get(0);
            List<Integer> preservedIndexes = new ArrayList<>();
            for (int i = 0; i < header.length; i++) {
                header[i] = header[i].trim().toUpperCase();
                preservedIndexes.add(i);
            }

            // Process data rows
            for (int i = 1; i < records.size(); i++) {
                String[] data = records.get(i);

                // Your existing processing logic
                int formatIndex = findIndex(header, "FORMAT");
                int proIndex = findIndex(header, "PRO");
                int cdIndex = findIndex(header, "CD");
                int isDIndex = findIndex(header, "ISD");
                int sourceIndex = findIndex(header, "SOURCE");

                // Process format
                if (formatIndex != -1 && "-".equals(data[formatIndex])) {
                    data[formatIndex] = "";
                }

                // Process PRO
                if (proIndex != -1) {
                    if ("".equals(data[proIndex])) {
                        data[proIndex] = findValue(data, proIndex, "PRO");
                    }
                } else { // If proIndex is -1, meaning no "PRO" column, check for "PRA" column
                    proIndex = findIndex(header, "PRA");
                    if (proIndex != -1) {
                        if ("".equals(data[proIndex])) {
                            data[proIndex] = findValue(data, proIndex, "PRA");
                        }
                    }
                }

                // Process CD, IsD
                processYNValues(cdIndex, data);
                processYNValues(isDIndex, data);

                // Process Source
                if (sourceIndex != -1 && ("NA".equals(data[sourceIndex]) || "--NA--".equals(data[sourceIndex]))) {
                    data[sourceIndex] = "";
                }

                // Process preserved headers
                for (int index : preservedIndexes) {
                    if (index < data.length) {
                        data[index] = data[index].trim();
                    }
                }
            }

            // Write to new CSV
            String destinationFilePath = destinationDir + File.separator + inputFile.getName();
            try (BufferedWriter writer = new BufferedWriter(new FileWriter(destinationFilePath))) {
                writeCSV(writer, records);
            }
            System.out.println("Processed: " + inputFile.getName());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void writeCSV(BufferedWriter writer, List<String[]> records) throws IOException {
        for (String[] record : records) {
            writer.write(String.join(",", record));
            writer.newLine();
        }
    }

    private static int findIndex(String[] header, String columnName) {
        for (int i = 0; i < header.length; i++) {
            if (columnName.equalsIgnoreCase(header[i])) {
                return i;
            }
        }
        return -1;
    }

    private static void processYNValues(int index, String[] data) {
        if (index != -1 && ("".equals(data[index]) || !"Y".equals(data[index].toUpperCase()))) {
            data[index] = "N";
        }
    }

    private static String findValue(String[] data, int proIndex, String columnName) {
        int index = -1;
        for (int i = 0; i < data.length; i++) {
            if (columnName.equalsIgnoreCase(data[i])) {
                index = i;
                break;
            }
        }
        if (index != -1 && !"Y".equalsIgnoreCase(data[index].trim())) {
            return "N";
        }
        return data[proIndex];
    }
}
