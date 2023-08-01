#!/bin/bash

# 定义要构建的服务名称，按照您的实际情况进行调整
SERVICES=("ServiceA" "ServiceB" "ServiceC")

# 获取项目根目录的绝对路径
PROJECT_DIR=$(dirname $(cd "$(dirname "$0")"; pwd))

# 循环遍历每个服务，并执行 mvn clean install
for SERVICE in "${SERVICES[@]}"
do
    SERVICE_DIR="$PROJECT_DIR/$SERVICE"
    
    if [ -d "$SERVICE_DIR" ]; then
        echo "Building $SERVICE..."
        cd "$SERVICE_DIR" # 进入相应的Service目录
        mvn clean install
        cd "$PROJECT_DIR" # 返回到项目根目录
        echo "Built $SERVICE successfully."
    else
        echo "Error: Directory for $SERVICE not found!"
    fi
done

echo "All services are built successfully."


 public static void main(String[] args) {
        String filePath = "path/to/your/excel-file.xlsx";
        ExcelReader reader = ExcelUtil.getReader(filePath);

        // Get the header row to map column names to their indexes
        List<Object> headerRow = reader.readRow(0);

        // Create a map to store column name to index mappings
        Map<String, Integer> columnIndexMap = new HashMap<>();
        for (int i = 0; i < headerRow.size(); i++) {
            String columnName = headerRow.get(i).toString();
            columnIndexMap.put(columnName, i);
        }

        // Read data by column names
        List<List<Object>> dataList = reader.read(1, Integer.MAX_VALUE); // Skip the header row

        for (List<Object> row : dataList) {
            // Access data by column name
            String column1Value = row.get(columnIndexMap.get("Column1")).toString();
            String column2Value = row.get(columnIndexMap.get("Column2")).toString();
            System.out.println("Column1: " + column1Value + ", Column2: " + column2Value);
        }

        reader.close();
    }
