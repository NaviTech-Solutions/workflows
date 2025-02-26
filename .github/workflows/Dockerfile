# 第一階段：構建階段
FROM openjdk:22-jdk-slim-bookworm AS build
WORKDIR /app

# 安裝必要的工具
RUN apt-get update && \
    apt-get install -y curl unzip && \
    rm -rf /var/lib/apt/lists/*

# 下載並安裝 Maven
ENV MAVEN_VERSION=3.9.9
RUN curl -fsSL https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.zip -o /tmp/maven.zip && \
    unzip /tmp/maven.zip -d /opt/ && \
    rm /tmp/maven.zip && \
    ln -s /opt/apache-maven-${MAVEN_VERSION}/bin/mvn /usr/bin/mvn



# 顯示 Java 和 Maven 的版本資訊
RUN java -version && mvn -v

# 複製 settings.xml 到 Maven 配置目錄
# COPY .gitea/workflows/settings.xml /root/.m2/settings.xml
# 下載 GitHub 上的 settings.xml
USER root
RUN mkdir -p /root/.m2 && \
    curl -fsSL https://raw.githubusercontent.com/NaviTech-Solutions/workflows/refs/heads/main/.github/workflows/settings.xml -o /root/.m2/settings.xml


# 安裝依賴
COPY pom.xml .
RUN mvn dependency:go-offline -s /root/.m2/settings.xml

# 複製專案文件並構建
COPY src ./src
RUN mvn clean install -DskipTests -s /root/.m2/settings.xml

# 第二階段：構建最終映像
FROM openjdk:22-jdk-slim-bookworm
ENV PARAMS=""
ENV TZ=PRC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 安裝字體相關依賴
RUN apt-get update && apt-get install -y fontconfig libfreetype6 && rm -rf /var/lib/apt/lists/*

# 拷貝 JAR 文件
COPY --from=build /app/target/*.jar /app.jar

# 啟動應用
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app.jar $PARAMS"]
