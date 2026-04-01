# --- Stage 1: base ---
# Lightweight ASP.NET runtime-only image used as the final container.
# NOTE: This is NOT the SDK - it cannot build or restore packages.
FROM mcr.microsoft.com/dotnet/aspnet:10.0-azurelinux3.0 AS base
USER app
WORKDIR /app
EXPOSE 8080

# --- Stage 2: build ---
# Full SDK image. The Docker build context must be the 'src' directory (one level
# above RandomQuotes.Web/) so that the COPY paths below resolve correctly.
# Expected context layout:
#   src/
#     RandomQuotes.Web/
#       RandomQuotes.Web.csproj
#       ...
FROM mcr.microsoft.com/dotnet/sdk:10.0-azurelinux3.0 AS build
WORKDIR /src
# Copy only the .csproj first to enable Docker layer caching on restore.
# Source path is relative to the build context (src/); destination mirrors that structure.
COPY ["RandomQuotes.Web/RandomQuotes.Web.csproj", "RandomQuotes.Web/"]
RUN dotnet restore "RandomQuotes.Web/RandomQuotes.Web.csproj"
# Copy the full source tree into /src (build context root → container /src)
COPY . .
# Switch into the project directory to run the build
RUN dotnet build "RandomQuotes.Web/RandomQuotes.Web.csproj" --no-restore -c Release

# --- Stage 3: publish ---
# Continues from the build stage. --no-build skips redundant recompilation.
# Output is written to /app/publish inside this stage's filesystem.
FROM build AS publish
RUN dotnet publish "RandomQuotes.Web/RandomQuotes.Web.csproj" --no-build -c Release -o /app/publish /p:UseAppHost=false

# --- Stage 4: final ---
# Returns to the lightweight base image and copies only the publish output.
# Nothing from the SDK or build stages is included, keeping the image small.
FROM base AS final
WORKDIR /app
# Copy publish output from Stage 3 into the runtime image at /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "RandomQuotes.Web.dll"]
ENV ASPNETCORE_HTTP_PORTS=8080
ENV ASPNETCORE_ENVIRONMENT="Production"
