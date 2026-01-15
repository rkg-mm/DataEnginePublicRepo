#See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
USER app
WORKDIR /app
EXPOSE 8080

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["AAS.TwinEngine.DataEngine/AAS.TwinEngine.DataEngine.csproj", "AAS.TwinEngine.DataEngine/"]
RUN dotnet restore "AAS.TwinEngine.DataEngine/AAS.TwinEngine.DataEngine.csproj"
COPY AAS.TwinEngine.DataEngine/ AAS.TwinEngine.DataEngine/
WORKDIR "/src/AAS.TwinEngine.DataEngine"

RUN dotnet build "AAS.TwinEngine.DataEngine.csproj" -c "$BUILD_CONFIGURATION" -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "AAS.TwinEngine.DataEngine.csproj" -c "$BUILD_CONFIGURATION" -o /app/publish /p:UseAppHost=false
# Install CycloneDX SBOM Generator Package
RUN dotnet tool install --global CycloneDX --version 5.5.0
# Generate SBOM at sbom/bom.xml (omitting dev/test dependencies as they do not appear in final build)
RUN dotnet CycloneDX "AAS.TwinEngine.DataEngine.csproj" -o "sbom/" --exclude-dev --exclude-test-projects --set-nuget-purl --spec-version 1.6  --disable-package-restore

# Create an "sbomartifact" image which can be use to extract the SBOM after completing the docker build
FROM scratch as sbomartifact
COPY --from=publish sbom/bom.xml /bom.xml

# Create final image
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final

RUN useradd -m appuser \
 && mkdir /app \
 && chown appuser:appuser /app

USER appuser
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "AAS.TwinEngine.DataEngine.dll"]
