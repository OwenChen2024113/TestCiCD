# 設置 Build Argument，預設值是 "Production"
ARG ENVIRONMENT=Production

# 在此一定要先將Dockerfile與啟動專案的sln放在同一個地方，.net 才要這樣做
# 語言設定
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app

#設定PORT
EXPOSE 8080  
                        
#將ENV ASPNETCORE_URLS接聽的IP從+號改為*號，讓服務接聽所有IP
ENV ASPNETCORE_URLS=http://*:8080

# 根據 Build Argument 設置 ASP.NET Core 環境
ENV ASPNETCORE_ENVIRONMENT=$ENVIRONMENT

# Creates a non-root user with an explicit UID and adds permission to access the /app folder
# For more info, please refer to https://aka.ms/vscode-docker-dotnet-configure-containers
RUN adduser -u 5678 --disabled-password --gecos "" appuser && chown -R appuser /app
USER appuser

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
#建立src資料夾
WORKDIR /src
#["根據專案目錄型態先去抓.csproj檔案","src底下再建立一個HotelMsg資料夾"]
COPY ["Test_CICD/Test_CICD.csproj", "Test_CICD/"]
COPY ["Test_CICD/appsettings.Uat.json", "Test_CICD/"]
#還原docker專案的相依性和工具
RUN dotnet restore "/src/Test_CICD/Test_CICD.csproj"
#從本機端複製所有檔案到docker專案底下
COPY . .
WORKDIR "/src/Test_CICD"
#建置專案和其所有相依性
RUN dotnet build "Test_CICD.csproj" -c Release -o /app/build

FROM build AS publish
#將應用程式和其相依性發佈至資料夾，以部署至主控系統
RUN dotnet publish "Test_CICD.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
#將應用程序及其依賴項發佈到文件夾以部署到託管系統
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Test_CICD.dll"]
