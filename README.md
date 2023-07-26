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
