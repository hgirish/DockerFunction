# See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.
ARG TENANT_ID
#ENV TENANT_ID=$TENANT_ID
# This stage is used when running from VS in fast mode (Default for Debug configuration)
FROM mcr.microsoft.com/azure-functions/dotnet-isolated:4-dotnet-isolated9.0 AS base
WORKDIR /home/site/wwwroot
EXPOSE 8080


# This stage is used to build the service project
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["DockerFunction/DockerFunction.csproj", "DockerFunction/"]
RUN dotnet restore "./DockerFunction/DockerFunction.csproj"
COPY . .
WORKDIR "/src/DockerFunction"
RUN dotnet build "./DockerFunction.csproj" -c $BUILD_CONFIGURATION -o /app/build

# This stage is used to publish the service project to be copied to the final stage
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./DockerFunction.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# This stage is used in production or when running from VS in regular mode (Default when not using the Debug configuration)
FROM base AS final
ARG TENANT_ID
ENV TENANT_ID=$TENANT_ID
WORKDIR /home/site/wwwroot
COPY --from=publish /app/publish .
ENV AzureWebJobsScriptRoot=/home/site/wwwroot \
    AzureFunctionsJobHost__Logging__Console__IsEnabled=true

#RUN powershell.exe -Command $ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; Remove-Item .\AzureCLI.msi
RUN apt-get -y update; apt-get -y install curl

RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash
#RUN  tdnf install ca-certificates
#RUN  tdnf install azure-cli
#FROM mcr.microsoft.com/azure-cli:cbl-mariner2.0
#RUN docker run -it mcr.microsoft.com/azure-cli:2.62.0-cbl-mariner2.0
RUN az login --tenant ${TENANT_ID}
#RUN az account show
RUN az help