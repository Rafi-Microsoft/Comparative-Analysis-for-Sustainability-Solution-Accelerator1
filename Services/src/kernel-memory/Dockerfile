# Copyright (c) Microsoft. All rights reserved.
# Licensed under the MIT License.

ARG BUILD_IMAGE_TAG="8.0-jammy"
ARG RUN_IMAGE_TAG="8.0-alpine"

#########################################################################
# build and publish
#########################################################################

FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:$BUILD_IMAGE_TAG AS build
ARG BUILD_CONFIGURATION=Release

ARG TARGETARCH
ARG TARGETPLATFORM
ARG BUILDPLATFORM

WORKDIR /src

COPY ["service/Service/Service.csproj", "service/Service/"]
RUN dotnet restore "./service/Service/./Service.csproj"

COPY ["extensions", "extensions"]
COPY ["tools", "tools"]
COPY ["service", "service"]
WORKDIR "/src/service/Service"
RUN dotnet build "./Service.csproj" -c $BUILD_CONFIGURATION -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./Service.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

#########################################################################
# run
#########################################################################

FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/aspnet:$RUN_IMAGE_TAG AS base
# Non-root user that will run the service
ARG USER=km
RUN \
    # Create user
    #Debian: useradd --create-home --user-group $USER --shell /bin/bash && \
    adduser -D -h /app -s /bin/sh $USER && \
    # Allow user to access the build
    chown -R $USER:$USER /app

# Define current user
USER $USER

# Used by .NET and KM to load appsettings.Production.json
ENV ASPNETCORE_ENVIRONMENT Development
#ENV ASPNETCORE_URLS http://+:80


WORKDIR /app
EXPOSE 8080

FROM base AS final

WORKDIR /app

COPY --from=publish --chown=km:km --chmod=0550  /app/publish .

# Define executable
ENTRYPOINT ["dotnet", "Microsoft.KernelMemory.ServiceAssembly.dll"]
